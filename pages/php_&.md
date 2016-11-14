# 内核学习相关笔记-变量引用

php是由c语言实现的，但是php变量引用不同于c的指针，php中的变量名称和值不是存储一个地方，通过指针链接；名称通过zend_has_add方法添加到符号表里并赋值。

先说一下存储变量的zval结构体：

> typedef struct _zval_struct zval;
>
> struct _zval_struct {
>
> ​     zvalue_value value;     /*存储变量的值*/
>
> ​     zend_unit refcount__gc;     /*引用计数 默认1*/
>
> ​     zend_uchar type; /*变量具体类型*/
>
> ​     zend_uchar is_ref__gc value; /*是否为引用 默认0*/
>
> }

##### 第一种情况：

```php
<?php
$a = 1;
$b = $a;
unset($a);
```

先创建变量$a，并申请内存来存放1，并发$a赋值给$b,然后释放$a。

php在赋值的时候两个变量指向同一个地址，不需要额外申请内存，当$b也引用这个zval结构时候，refcount\_\_gc 值+1，现在有两个变量在使用这个地址。当释放$a时，删除符号表里面$a信息，再对zval结构中的 refcount\_\_gc 值减一。 当释放只有一个变量使用的zval，减一后引用计数为0，直接清理变量。

##### 第二种情况：

```php
<?php
$a = 1;
$b = $a;
$b += 1;
```

当修改$b值得时候，zval结构中is\_ref\_\_gc值为0，代表没有被php变量引用，$a和$b就不会共享一个zval结构，再查看zval结构的引用计数refcount\_\_gc，值大于1，从原zval结构复制一份新zval结构并修改值，同时原zval结构的引用计数减一。

##### 第三种情况：

```php
$a = 1;
$b = &$a;
$b += 1;
```

当变量$b引用$a时候，$a的zval结构中refcount\_\_gc +1，并且is\_ref\_\_gc值变为1。这是执行到修改$b值时，原zval结构中的is\_ref\_\_gc值为1，有php变量在引用。所以不会复制新zval。直接修改原zval值。

##### 第四种情况：

```php
<?php
$a = 1;
$b = &$a;
$c = $a;
```

或者

```php
<?php
$a = 1;
$b = $a;
$c = &$a;
```

当 共享被php变量引用的zval 或者 变量引用的值为共享的zval值时，zval必须强复制，需要两个zval来实现，防止产生歧义。

也就是is_ref = 1、ref\_\_count>1，共享的zval有被php变量引用，新赋值的变量需要强制复制zval， 或者 is_ref = 0; ref\_\_count>1，共享的zval没有被php变量引用，当有新的变量引用时候强制复制zval。







