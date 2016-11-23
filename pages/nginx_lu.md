# Nginx的配置

nginx的配置文件 nginx.conf，conf文件中有配置项，注释（开头使用#注释），块配置组成。下面来介绍每个配置项的使用：

##### 配置项

格式： 配置项名 配置项值一 配置项值二 … ;

配置项名称和值之间使用空格分隔，一个配置项可以有一个或多个值，配置项值中如果有语法符号或变量时需要使用单引号或双引号,配置项结尾使用分号";"。

##### 块配置

格式 ：配置项名 配置项值 {

}

块配置项由配置项名和大括号组成，参数可空，在大括号中可以嵌套块配置和配置项。

配置项值中会有空间（k：KB，m：MB）、时间（ms：毫秒，s：秒，m：分，h：时，d：天，w：周，M：月，y：年）和变量（$符号开头）。

### 常用配置

- 用于调试和定位问题
  - master_process：默认on，是否以一个master进程管理多个worker进程方式运行，关闭off。
  - error_log：日志配置，格式error_log log文件地址 日志级别（error_log logs/error.log error），日志级别（debug、info、notice、warn、error、crit、alert、emerg）从左到右依次增大。使用debug级别是如果没有全部日志输出，需要在configure时添加—with-debug开发功能。
  - access_log：用户访问信息。
  - log_format：日志格式。
  - debug_connection：制定客户端输出debug日志，需要写在events块中，后面跟IP或CIDR地址
- 正常运行配置
  - include：嵌入文件，文件可以是相对conf所在路径或绝对路径，可以使用通配符*嵌入多个文件。
  - pid：master进程ID存放文件路径。
  - user：worker进程原型的用户，用户组。
  - worker_rlimit_nofile limit：一个worker进程可以打开最大文件句柄数。
  - worker_rlimit_sigpending limit：worker进程排队队列大小。
- 性能优化
  - worker_processes：worker进程数，一般情况下配置和CPU内核数相同
  - worker_cpuaffinity cpumask：绑定worker到指定CPU内核。
  - worker_priority：worker进程优先级，优先级高的进程会有更多的系统资源，范围为-20 ~ +19，值越小优先级越高，不建议比内核进程的nice（ps -l 中NI的值就是nice值）还小。
- 事件类
  - accept_mutex：nginx负载均衡锁，默认开启。
  - lock_file：accept锁在不支持原子锁时候使用的文件。
  - worker_connections：每个worker进程可以同时处理的最大连接数。

#### HTTP块

HTTP配置块中包含server块，location块，upstream，if块等，这些都必须在http块的大括号中。下面的配置项没有说明的在http，server，location块中都可以使用。

- 虚拟主机和请求分发

  - listen：监听端口，在server块中使用。
  - server_name：主机名称，可以多个参数，在server块中使用。
  - server_name_in_redirect：使用on打开后，从定向请求时会使用server_name配置的第一个域名替换原请求头中的host头部。
  - location：根据url后的地址来匹配，匹配成功后使用该location块继续处理请求，在server块中使用。

- 文件路径

  - root：资源路径，可以理解为根目录。
  - alias：资源路径，与root不同的地方是 alias的路径是根据location后的参数来解读的。只能在location中使用。
  - rewrite：url重写，按照重写后格式解析。
  - index：首页设置，当没有路由匹配时候 根据index后面的参数一次来访问。
  - error_page：error_page 404   /404.html; 根据HTTP状态码重定向页面，设置了会根据对应状态码后面的路径来从定向，如果想修改状态码可以使用=（404 = 400），当=后面没有具体状态吗的时候就是不指定提示状态码，对于路径也可以从定向到内部location处理（location 后面参数以@开头的）。
  -  try_files：后面跟一个或多个路径，当路径都无效的时候会从定向到最后一个路径参数上。在server、location中使用。

- 网络连接

  - client_header_timeout：http头超时时间，默认单位60秒，超时返回408。
  - client_body_timeout：http包超时时间。
  - send_timeout：发送响应超时时间。客户端超时未接受则关闭连接。
  - keepalive_disable safari：禁用某些浏览器keepalive功能。
  - keepalive_timeout；对于keepalive闲置超一定时间后关闭连接。
  - keepalive_requests：长连接上最大请求数，默认100。

- 客户端请求

  - limit_except method …{ … } ：限制method方法的请求。
  - client_max_body_size 1m：http包体重content-length所示值。
  - limit_rate：对请求限速
  - limit_rate_after 1m：对响应长度超过设置后开始限速
  - resolver address …：DNS解析地址
  - resolver_timeout：DNS解析超时时间。
  - server_tokens：错误请求时头部是否显示nginx版本。

- 文件

  - sendfile：开启后提高发送文件效率，默认关闭。

  - open_file_cache：文件缓存，参数 max：内存中存储元素的最大数，并使用LRU方式淘汰；

    inactive=60：指定时间内没有使用的元素淘汰，off：关闭。

    ​

- upstream 负载均衡

  upstream backend {

  ​        \#ip_hash;

  ​	server 127.0.0.1:8001 weight=1;

  ​        server 127.0.0.1:8002 max_fails=5 fail_timeout=10s;

  ​	server 127.0.0.1:8003 down;

  }

  ip_hash ：会根据客户ip讲客户请求永远的转发到某个固定的上游服务器中。确保一个客户端之请求一个服务器。

  weight：权重，优先访问。与ip_hash不能同时使用。

  max_filas：根据fail_timeout设置的时间，当失败超过设置的值时停止使用该上游服务器，

  fail_timeout：转发失败次数判断的时间阀值。

  down：使用ip_hash时，用来下线上游服务器。

  backup：不适用ip_hash情况下的备份服务器。

  proxy_pass：反向代理，把请求转发到指定内部的服务器上，可以配合upstream使用。
