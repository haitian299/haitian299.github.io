---
layout:     post
title:      "［设计模式］唠唠依赖注入"
subtitle:   ""
date:       2016-05-17
author:     "haitian-coder"
header-img: "img/bg.jpg"
catalog:    true
tags:
    - PHP
    - 设计模式
    - 依赖注入
---


## 啥是依赖注入(Dependency injection)? ##
直接上例子：

 - 这不是依赖注入！

```
//这不是依赖注入！！！
class Bar
{
}

class Foo
{
    protected $bar;

    public function __construct()
    {
        $this->bar = new Bar();
    }

    public function getBar()
    {
        return $this->bar;
    }
}

$foo = new Foo();
```

 - 这就是依赖注入

```
//这就是依赖注入。。。
class Bar
{
}

class Foo
{
    protected $bar;

    public function __construct(Bar $bar)
    {
        $this->bar = $bar;
    }

    public function getBar()
    {
        return $this->bar;
    }
}

$bar = new Bar();
$foo = new Foo($bar);
```
 - 这也是依赖注入

```
//这也是依赖注入。。。
class Bar
{
}

class Foo
{
    protected $bar;

    public function __construct()
    {
    }
    
    public function setBar(Bar $bar)
    {
        $this->bar = $bar;
    }

    public function getBar()
    {
        return $this->bar;
    }
}

$bar = new Bar();
$foo = new Foo();
$foo->setBar($bar);
```
依赖注入就是new好了依赖的对象`注入`进去，而不是在类中显式的new一个依赖的对象。其实，就是这么简单。。。

## 为啥要用依赖注入？ ##

虽然思想简单，但是在降低耦合度方面作用巨大。

## 依赖注入都可以怎么用 ##

下面举个例子说明（just for demonstration）：
比如我们做了个小游戏，里面的男人可以亲自己的妻子。

```
abstract class Human
{
}
class Woman extends Human
{
}

class Man extends Human
{
    protected $wife;

    public function __construct()
    {
        $this->wife = new Woman();
    }

    public function kissWife()
    {
        echo "the man kissed his wife";
    }
}

$man = new Man();
$man->kissWife();
```
玩的人越来也多，新需求随之而来。。。

> 产品经理（`腐腐`）：妻子改成可以是男的吧，好多用户有这个需求，这样玩我们游戏的肯定更多。
> 程序员（`猿猿`）心里：擦，Wife又可以是Man，又可以是Woman，这可咋整。

这个时候，依赖注入就可以闪亮登场了。

```
abstract class Human
{
}

class Woman extends Human
{
}

class Man extends Human
{
    protected $wife;

    public function setWife(Human $human)
    {
        $this->wife = $human;
    }

    public function kissWife()
    {
        echo "the man kissed his wife";
    }
}

$man = new Man();
$man->setWife(new Woman());
$man->kissWife();

$anotherMan = new Man();
$anotherMan->setWife(new Man());
$anotherMan->kissWife();
```
这里我们看到，**依赖注入的可以是继承依赖类的任何类**，所以现在Man的Wife既可以是Woman也可以是Man。

玩的人越来也多，新需求随之而来。。。

> 产品经理（`宅宅`）：把妻子改成伴侣吧，伴侣里面除了Man和Woman再加个Cat，好多用户有这个需求，这样玩我们游戏的肯定更多。
> 程序员（`猿猿`）心里：擦，又是Man又是Woman还有Cat，幸好我会依赖注入。

```
abstract class Human
{
}

interface canBePartner
{
}

class Cat implements canBePartner
{
}

class Woman extends Human implements canBePartner
{
}

class Man extends Human implements canBePartner
{
    protected $partner;

    public function setPartner(canBePartner $partner)
    {
        $this->partner = $partner;
    }

    public function kissPartner()
    {
        echo "the man kissed his partner";
    }
}

$man = new Man();
$man->setPartner(new Woman());
$man->kissPartner();

$man2 = new Man();
$man2->setPartner(new Man());
$man2->kissPartner();

$man3 = new Man();
$man3->setPartner(new Cat());
$man3->kissPartner();
```

这里我们看到，**依赖注入不但可以是继承依赖类的所有子类，也可以是实现依赖接口的所有类。**
所以如果我们在伴侣中再加入一个Dog，只需要让它实现canBePartner接口就可以了：

```
class Dog implements canBePartner
{
}
$man = new Man();
$man->setPartner(new Dog());
```

##  实际应用  ##
依赖注入虽然降低了耦合度，但是也有缺点，就是需要我们自己管理注入的对象。
所以，在实际应用中，我们通常需要实现一个容器去管理和实现依赖对象的注入。
实际上，PHP的常用Web框架中都是这么做的。
