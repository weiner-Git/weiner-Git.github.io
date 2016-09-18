#设计模式-单例模式

**概念**（维基百科）也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息。这种方式简化了在复杂环境下的配置管理。

**解释说明** 
这里拆分解释下概念，单例模式的三个要点：<br />
一：一个类只有一个实例<br />
二：自行创建该类的实例化<br />
三：向全局提供这个实例<br />
从具体来说<br />
一：是单例模式的类只提供私有的构造函数<br />
二：是类定义中含有一个该类的静态私有对象<br />
三：是该类提供了一个静态的公有的函数用于创建或获取它本身的静态私有对象<br />

**应用**<br/>
吃麻辣烫时候给每个人发的餐牌号，这时候有个叫号器，每次只能叫一位，你把开水烫熟的东西拿走后，开始叫下一位。如果这时候一起叫了两个，那就会发生纠纷了。保证一个餐牌号对应一碗麻辣烫。

数据库连接，每次new一个句柄操作都会占用很多资源，利用单例可以避免new操作。

全局的配置文件，利用单例防止多人同时参与修改。

**代码**

```
<?php
/**
* $_instance必须声明为静态的私有变量
* 构造函数和析构函数必须声明为私有,防止外部程序new类从而失去单例模式的意义
* getInstance()方法必须设置为公有的,必须调用此方法以返回实例的一个引用
* ::操作符只能访问静态变量和静态函数
*/
class DB {
 
	//保存类实例的静态成员变量
	private static $_instance;
 
	//private标记的构造方法
	private function __construct(){
		$this->conn = mysql_connect();
	}
 
	//覆盖__clone()方法，禁止克隆
	private function __clone(){}
 
	//单例方法,用于访问实例的公共的静态方法
	public static function getInstance(){
		if(!(self::$_instance instanceof self))
			self::$_instance = new self;

		return self::$_instance;
	}
 
	public function select(){
		echo 'select * from system_user';
	}
 
}
 
//正确方法,用双冒号::操作符访问静态方法获取实例
$db = DB::getInstance();
$db->select();
```