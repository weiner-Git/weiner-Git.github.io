# test

**php实现的一个在线接口文档注释**

在项目开发中需要写接口文档，每次都要花好多时间写word，所以想用这种读取接口注释的方法，实现简单的在线文档。

zaiyii框架在xia开发；首先确定那些类需要注释：

```php
//	这里定义了一个注释变量，用来维护需要注释的类
private $aliases = array(
  	'foo'=>'application.controllers.*',	//	controller整个目录
  	'bar'=>array(
    	'application.controllers.HomeController',	//	HomeController
  	)
);

/**
 *	根据注释变量返回需要注册的类的集合
 */
protected function filterControllers($alias){
	$controllers	= array();
  	//	在验证目录还是php文件时候没更多的验证是否都是正确能处理的参数，在维护别名变量时候注意一下
    if(substr($alias,-1) == '*'){	
        $path = Yii::getPathOfAlias($alias);	//	这里用的框架方法获取根目录，其它环境可以用绝对路径来替换。
        $menuControllers	= glob($path."/*.php",GLOB_BRACE);	//	用glob来匹配.php结尾的文件。
        foreach($menuControllers AS $val){
          $controllers[]	= pathinfo($val,PATHINFO_FILENAME);	//	用pathinfo来返回filename，去掉路径和结尾的那个不 ***Controller
        }
    }else{
        //	这里用正则对单文件处理，获取filename
        preg_match('/(\w+Controller)/', $alias, $matches);
        $controllers[]		= $matches[1];
    }

  	return $controllers;
}
```

上面已经把我们需要注释的类都放到一个数组中了，下面说一下怎么获取类的注释；

这里用到[ReflectionClass](http://php.net/manual/zh/class.reflectionclass.php)这个类(reflection反射类有method、function很多种类)，用它来解析php类，可以获取到类、方法、参数、属性、注释信息（需要注意的是ReflectionClass解析的是声明的结构，实例化后的属性不会解析，需要先new一下要解析的类，再解析才会获取在函数中声明的属性）。

下面看怎么处理获取到的信息

```php
Yii::import($alias);	//	加载类(alias可以是目录) 可用require替代
$ref = new ReflectionClass($alias);	//	解析加载类
$doc = $this->parseDoc($ref); 

/**
 *	处理类信息
 */
protected function parseDoc($ref) {
        $doc = new stdclass();	//	创建一个记录信息的object
        $doc->class = $ref->getDocComment();	//	首先获取类的注释

        $methods = $ref->getMethods(ReflectionMethod::IS_PUBLIC);	//	获取public声明的方法

  		//	过滤一下为action的方法
        $actions = array_filter($methods, function($method){
            return preg_match('/^action[A-Z](.*)$/', $method->name);
        });

        $doc->actions = array();
  		//	根据action名称和注释 
  		//	这里可以再添加一个字段获取注释中action的主要备注(可以用标示正则获取)	'/start:([\W\w\b]*?\@)/'   
        array_map(function($action) use (&$doc) {
          	$actionDoc = $action->getDocComment();
            $doc->actions[$action->name] = $actionDoc;
            return $action;
        }, $actions);

        return $doc;
    }
```

信息也获取到了下面就是展示了。通过页面参数（aliases的key+controller名称或是参数可以是类的集合数组的下标）来访问。功能不是通用，可以按照这个方法嵌入自己需要的逻辑处理。

![for exp](http://ocaya4boy.bkt.clouddn.com/desc2.jpeg)
