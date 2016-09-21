#openresty简单应用


1. nginx多进程多work。协程高效的信号同步，sema、buffer 、lua_share_dict 


**nginx：**为一个主进程多个工作进程的工作模式，每个进程是单线程来处理多个连接，而且每个工作进程采用了非阻塞I/O来处理多个连接，从而减少了线程上下文切换，从而实现了公认的高性能、高并发；它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。

**lua：**是一个简洁、轻量、可扩展的脚本语言，目标是成为一个很容易嵌入其它语言中使用的语言，以此来实现可配置性、可扩展性。

**ngx_lua：**是nginx的一个模块，在nginx中嵌入lua，这样nginx变成一个web容器，用lua来开发web应用。

**openresty：**是一个基于nginx与lua的高性能web平台，其内部集成了大量精良的lua库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态web应用、web服务和动态网关。openresty将nginx，luajit，lua库，第三方模块打包在一起，这样直接安装openresty就可以使用lua进行开发。

**openresty安装：**
参考官网的下载安装教程：[openresty.org](http://openresty.org/cn/download.html)
说明的很详细，这里就不列出来了。

**openresty简单使用**

下面是我重写了项目中的消息接受模块的代码结构<br />
![lua_st](http://ocaya4boy.bkt.clouddn.com/image/lua_st.jpeg?imageView2/0/w/500)<br />
然后并没有上线替换现有模块，而是对现模块进行了一些重构。

1.首选说一下数据接收，该消息模块接收微信服务器发送来的数据，所以数据结构都是xml格式的；这里使用的xmlSimple来解析xml，xmlSimple没有对xml中的CDATA解析，修改xmlSimple.lua添加了一个过滤

```lua
-- 过滤微信xml数据格式
function XmlParser:formatText(text)
    if string.find(text, "<!%[CDATA%[") then
        text = string.sub(text,10,string.len(text))
    end
    if string.find(text, "%]%]>") then
        text = string.sub(text,0,string.len(text)-3)
    end
    return text
end
```
2.参数获取方式

* get :ngx.req.get_uri_args()
* post:ngx.req.get_post_args()  (这里要注意一下，解析body参数之前一定要先读取body：ngx.req.read_body() )
* body:ngx.req.get_body_data()

3.数据处理与存储

微信接受普通消息接口需要5秒内把响应的被动回复的内容输出，所以这里需要把一些要被动回复的内容返给微信，如默认回复、关键词处理、一些和接受到的消息处理匹配的结果。

openresty的输出有两种：

* ngx.say()
* ngx.print()

这两种方法都是异步输出响应体，区别是ngx.say 会对输出响应体多输出一个 \n，ngx.print的输入参数可以是单个或多个字符串参数，也可以是table对象。这里要及时的响应，用ngx.flush()，显式的向客户端刷新响应输出。

```
ngx.print(data)
ngx.flush(true)
```

在数据处理这块根据业务，对需要及时响应的消息输出消息体，其它消息加入队列入库。（在每次群发后因为大量mysql写库和php线程量大问题都会引发服务器报警，所以拆分消息处理和入队减少写库压力和减少php单个线程处理时间）

3.1 数据格式<br />
3.1.1 JSON <br />
微信响应的消息大多数都为json格式，cjson是一个很好用的解析器,openresty自带直接require就可以使用，window用户可以选择dkjson<br />

```
local cjson = require "cjson"
cjson.encode(args)
cjson.decode(args)
```
3.1.2 XML<br />
上面介绍了xmlSimple来解析，这里介绍一下用lua的xml来创建xml<br />[Lua Xml](http://viremo.eludi.net/LuaXML/)的文档地址，这里要注意一下lua xml安装后require("LuaXml")后，就可以直接只用xml，并不需要local luaxml = require("LuaXml")，这个时候你使用luaxml会发现并没有加载xml的方法，因为luaxml安装后是全局变量直接使用xml就好了。<br />

```
local xmlData = xml.new("xml")	--创建跟xml
local ToUserName = xml.new("ToUserName") --创建一个元素
table.insert(ToUserName, "<![CDATA[lua_table.open_id]]")	--为元素赋值
xmlData:append(ToUserName,1)	--把元素添加到xml中
ngx.log(ngx.STDERR, "xmlData: ", xmlData)	-- 这个时候可以在log中查看一下创建的结果
```
注：append(name,1)等于append(name)[1]

3.1.3 数组<br />
lua中数组下标是以1开始计算的

```
local data = {1, 2}
```
openresty中有个稀疏数组问题 

```
local cjson = require("cjson")

local data = {1, 2}
data[1000] = 99

ngx.say(cjson.encode(data))
``` 
运行测试这时候日志中就会报错，但如果把1000修改为5就会成功，实际是因为cjson想保护你的内存资源。<br />
table.getn(t) 等价于 #t 但计算的是数组元素，不包括hash键值。而且数组是以第一个nil元素来判断数组结束。# 只计算array的元素个数，它实际上调用了对象的metatable的\_\_len函数。对于有\_\_len方法的函数返回函数返回值，不然就返回数组成员数目。

3.1.4 table<br />
table类型实现了一种抽象的“关联数组”。“关联数组”是一种具有特殊索引方式的数组，索引通常是字符串（string）或者number类型，但也可以是除 nil 以外的任意类型的值。table 库是由一些辅助函数构成的，这些函数将 table 作为数组来操作。所以table.getn(n)都适用。</br >
实际上table和数组可以当一种结构计算，因为lua只支持一种数据结构就是table，这里分开是为的介绍一下是为了能更好的理解table。

当然还有其他类型，boolena，number，string，function，nil（ nil 用于表示“无效值”，一个变量在第一次赋值前的默认值是 nil，将 nil 赋予给一个全局变量就等同于删除它。）。

3.2 数据存储

数据存储这里使用的是redis和mysql结合，openresty有[lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)、[lua-resty-redis](https://github.com/openresty/lua-resty-redis)这两个组件，功能上都能满足开发需求，不满足的地方可以扩展一下。

```
local redis = require "redis_iresty"  -- 这里使用openresty最佳实践中redis的二次封装文件，所有的连接创建、销毁连接、连接池部分，都被完美隐藏了，我们只需要业务就可以了。给作者点个赞！！！
local ok, err = red:rpush("resque:queue:wechat", cjson.encode(args))
if not ok then
    ngx.say("failed to set: ", err)
    return
end
```

这里解释一下处理上的逻辑，用lua处理后及时响应体后要把所有信息都入库，直接写库会对mysql很大压力，所以把消息都push到队列中，用php的[php-resque](https://github.com/chrisboulton/php-resque)来处理队列和数据入库，来减少消息接受模块压力。

上面功能基本上可以满足一个web开发，下面通过一段代码，来讲一下简单的说一下lua中如何调用 C 函数和lua元表、元方法，和其它一些openresty中比较有用的东西还没使用到的，算是加深一下学习。（推荐lua程序设计这本书，这本书是一开始博主的小领导推荐的，看元表元方法这章就可以，公司有这本书，一直没看，接触了openresty才看了一遍，理解不是很深，这里简单的说一点点）。

**FFI**

cat一下ttime.lua文件，下面是代码：

```
local ffi = require("ffi") 

local _M = {}
local mt = { __index = _M }

function _M.new(self)
    ffi.cdef[[
        struct timeval {
                long int tv_sec;
                long int tv_usec;
        };
        int gettimeofday(struct timeval *tv, void *tz);
    ]];
    local tm = ffi.new("struct timeval");
    ffi.C.gettimeofday(tm, nil)
    local sec = tonumber(tm.tv_sec)
    local usec = tonumber(tm.tv_usec);
    return setmetatable({
        sec = sec,
        usec = usec
    }, mt)
end

return _M
```

openresty中默认使用的LuaJit，LuaJit中提供了FFI，通过FFI的方式加载其他C接口动态库，这样我们就可以有很多有意思的玩法。
示例：

```
local ffi = require("ffi")  --加载FFI库
ffi.cdef[[
int printf(const char* fmt, ...);
]] --为函数增加一个函数声明。这个包含在`中括号`对之间的部分，是标准C语法。
ffi.C.printf("Hello %s!", "world") -- 调用命名的C函数——非常简单
```

在重写模块时候需要把在lua中按照php-resque的存储格式入队，在源码中有这样一段

```
Resque::push($queue, array(
			'class'	=> $class,
			'args'	=> array($args),
			'id'	=> $id,
			'queue_time' => microtime(true),
		));
```
php中microtime()函数返回的是当前时间戳以及微秒数，lua库中时间只到秒所以来用ffi来实现。

**元表元方法**元表对应的英文是metatable，元方法是metamethod；通常，Lua中的每个值都有一套预定义的操作集合，比如数字是可以相加的，字符串是可以连接的，但是对于两个table类型，则不能直接进行“+”操作。这需要我们进行一些操作。在Lua中有一个元表，也就是上面说的metatable，我们可以通过元表来修改一个值得行为，使其在面对一个非预定义的操作时执行一个指定的操作。
元方法\_\_index，当访问table中不存在的字段时有这个元方法就会由该元方法处理，否则返回nil。<br />
在ttime.lua中的 setmetatable()方法来设置和修改table的元素，任何table都可以作为任何值得元表，也可以和其它table共享一个元表，也可以做自己的元表。
ttime.lua里面的mt是集合的元表，现在把_M 赋值给\_\_index元方法，现在的\_\_index为一个table，它也可以是一个function。在表new的时候通过setmetatable()为table把两个table的元素“+”操作。

```
local ttime = require "ttime"
local t     = ttime.new()
local queue_time = os.time() .. t.usec		-- t.usec为微秒数
```

**变量共享**

* 所有worker间共享使用lua_shared_dict（使用的时候要先定义lua_shared_dict my_cache 128m，并且每次操作都是全局锁可以在这里保存微信的access_token）
* 单worker内（进程间）可以使用不加local的全局变量
* 单次请求内正常定义使用就可以了

......

当然openresty的功能不止这些，以上只是在项目中应用的部分，有想了解更多的同学可以去看看源码和openresty最佳实践。

