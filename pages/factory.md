#工厂模式

**工厂模式：**（维基百科）定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。

工厂模式 是一种类，它具有为您创建对象的某些方法。您可以使用工厂类创建对象，而不直接使用 new。这样，如果您想要更改所创建的对象类型，只需更改该工厂即可。使用该工厂的所有代码会自动更改。

下面是php示例的代码：

```
<?php 
//	定义button接口类
interface ButtonFactory(){
	public function createbutton();
}

//	创建大按钮的类
class BagButton{
	public function action(){
		echo "创建大按钮";
	}
}

//	创建小按钮的类
class SmallButton{
	public function action(){
		echo "创建小按钮";
	}
}

//	执行创建的类实现button接口类
class CreatButton() implements ButtonFactory{
	public $type;	//	根据type类型决定创建那个button
	public function createbutton(){
		if($this->type == 'bag')
			return new BagButton();
		else if($this->type == 'small')
			return new SmallButton();
		else
			return '创建失败';
	}
}

//	创建按钮
$button = new CreateButton();
$button->type = 'bag'; //	创建大button
$button->createbutton();

```

工厂方法如果在较小项目中使用有些实际意义不大的感觉，但是了解一下工厂模式，在项目中应用可以简单的创建一个类，不用考虑内部细节，降低了耦合。

这里要注意一些：implements是实现接口，因为接口中的方法只是定义不实现，所以实现接口时候要实现接口的方法，必须重写才能使用。
