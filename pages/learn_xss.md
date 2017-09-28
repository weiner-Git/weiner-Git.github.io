# 开发如何防范XSS -学习笔记整理
博主并不是挖洞的（还在学习中，努力成为一名优质白帽），所以从开发角度，把自己遇到的问题描述出来，主要是防范xss。

#### xss介绍
xss主要是在网页中嵌入恶意代码，主要是js，可以用嵌入的js来获取用户的cookie（包括管理员也属于用户）、改变页面内容、链接等等。就是说利用xss可以打到后台、黑掉页面、钩鱼、挂马、蠕虫等等。

xss主要分为反射型、存储型、DOM型三种类型。

#### 反射型
反射型是比较容易出现的一种，当服务器接收到数据后，没有过滤、存储，直接发送给浏览器，浏览器解析了带有xss的代码，造成xss攻击。主要发生因素是主动点击带有xss代码参数的url。

#### 存储型
存储型又叫持久型，当服务器接收到数据后，存储了带有xss的代码，这样当页面读取数据时候就会造成xss攻击，相比其他两种，存储型比较隐蔽，不需要用户自己手动去触发，危害性相对大一些。

#### DOM型
DOM是指html标签、属性、元素等的每个节点。
DOM型和反射型类似，也是通过url参数传入xss代码，然后通过DOM获取输出页面上，产生xss攻击。

#### xss检测工具
xsser.me、xssf、domxsscanner.com、 xss platform

#### xss利用  
* 会话劫持  
	http是无状态的，通常会使用cookie和session来标识用户、维持会话。劫持状态后就可以使用状态码来响应操作，如直接登录。  
	cookie被本地保存，格式使用“变量=值”，js php等大多数语言都能直接读取cookie，使用js document.cookie可以直接对cookie操作值，利用跨域劫持到的cookie就可以在其他地方登录只使用cookie来验证身份的网站，实现会话劫持。  
	session被保存在服务端，当会话链接开始时候会创建一个sessionID，当关闭浏览器断开链接或服务关闭时候会注销掉。sessionID也可以保存在cookie中，相对提高安全。
	xss platform使xss测试更加便利，跨站漏洞测试能够持久保存状态，通知一些便捷功能
* xss蠕虫
	蠕虫传播非常迅速，xss利用存储型漏洞，可以快速实现传播，如微博，贴吧这种可以利用受害者发送内容传播更加迅速，危害非常大。
	[百度贴吧](http://www.guokr.com/question/464747/)
	[新浪微博](http://bbs.51cto.com/thread-1063524-1-1.html)
	[原理分析](http://www.freebuf.com/articles/web/19408.html)
* getshell  
	利用xss的跨域也可以实现getshell攻击
	[原理介绍](http://www.cnblogs.com/LittleHann/p/4237578.html)
	
#####XSS过滤方法
* 输入输出
	特殊符号转义：php中htmlspecialchars、htmlentities会把& 、 “ 、 ‘ 、 < 、 > 这些特殊符号转义。
* 黑名单
	对禁用的标签过滤，缺点是需要及时补充。
* 白名单
	对html处理只需要自己使用的标签和属性。相对安全，但是为了满足安全条件，阉割了大部分标签功能
* 其它
	* htmlFiler 一个开源的xss过滤脚本
	* [OWASP](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API) 提供api级别的解决方案，支持多种语言。
