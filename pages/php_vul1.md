# php发开漏洞防范 -学习笔记整理
学习笔记整理  
[]()
####PHP相关函数
* phpinfo  
&nbsp;&nbsp;phpinfo 用于输出当前环境信息，配置设置，同时也包含GET、POST、COOKIE、SERVER等信息。
* passthru  
&nbsp;&nbsp;执行外部命令，并且能显示原始输出，手册上提示用escapeshellarg()或escapeshellcmd() 转义执行命令  
* exec  
&nbsp;执行外部命令，不显示输出，可将返回内容存入数组。同样可使用escapeshellarg()或escapeshellcmd() 转义
* system  
&nbsp;执行外部命令，显示输出的最后一行。
* pcntl_exec  
&nbsp;在当前进程执行可执行的程序或文件
* chown  
&nbsp;同liunx命令，改变文件所有权
* proc_open  
&nbsp;执行外部命令，双向非阻塞操作，  
* include  
&nbsp;引用文件
* require  
&nbsp;引用文件， 不同于include，require未找到文件会报错，而include是警告
* extract  
&nbsp;将数组中key变成变量，value为变量的值。   
* parse_str  
&nbsp;解析字符串为多个变量。
* unserialize  
&nbsp;对可解序列化的字符串处理为php变量。  
* fread  
&nbsp;对fopen打开发文件，使用fread读取内容
* copy  
&nbsp;文件拷贝
* fputs  
&nbsp;将内容写入文件
* unlink  
&nbsp;删除文件
* popen  
&nbsp;单向的通过新的进程执行外部命令  
* eval  
&nbsp;把字符串作为php代码任意执行，非常危险。
* preg_replace  
&nbsp;正则表达式替换
* mysql\_query mysqli\_query mysqli::query pdo::query
&nbsp;执行sql查询  
* $\_SERVER  
&nbsp;服务器和执行环境信息  
* $\_GET  
&nbsp;HTTP GET 变量
* $\_POST  
&nbsp;HTTP POST 变量  
* $\_COOKIE  
&nbsp;HTTP Cookies
* $\_REQUEST  
&nbsp;http request变量， 默认包含get post cookies
* $\_FILES  
&nbsp;文件上传的变量
* $\_ENV  
&nbsp;环境变量

####常见的风险
*可操作的参数名*

```php
foreach($_GET AS $key=> $val){
	print $key."\n";
}
```  
可以根据使用key的函数来触发漏洞

*变量覆盖*  

```php
//url = ?var=1&a[1]=var1%3d222
$a = 'hi';
$var1 = 'init';
parse_str($a[$_GET['var']]);
print $var1;
```

*eval/preg_replace命令执行*  
特别容易的拿shell  

*字符串截断*  
[iconv](http://www.cnseay.com/3700/)   
[字符串编码](http://www.cnblogs.com/hongfei/p/3893305.html)
[特殊字符null](http://php.net/manual/zh/security.filesystem.nullbytes.php)


待补充ing...
