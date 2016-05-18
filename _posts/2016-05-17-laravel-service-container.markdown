---
layout:     post
title:      "［Laravel］唠唠Service Container"
subtitle:   ""
date:       2016-05-17
author:     "haitian-coder"
header-img: "img/bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - PHP
    - Laravel
---



## 什么是Service Container ##

> The Laravel service container is a powerful tool for managing class dependencies and performing dependency injection. 

从Laravel官方文档对于Service Container的解释可以看出，它的作用就是帮助我们管理和进行依赖注入的。

## 为什么要用Service Container ##

在[《唠唠依赖注入》][1]中，我们看到使用依赖注入可以极大的降低代码的耦合度，但是也带来了一个缺点，就是需要自己管理注入的对象。
如果一个组件有很多依赖，我们需要创建多个参数的setter方法​​来传递依赖关系，或者建立一个多个参数的构造函数来传递它们，另外在使用组件前还要每次都创建依赖，这让我们的代码像这样不易维护。
所以为依赖实例提供一个容器（Service Container），就是一个实用而且优雅的方法。
比如下面是laravel的入口文件（已去掉注释）：


```
// public/index.php
<?php

require __DIR__.'/../bootstrap/autoload.php';

$app = require_once __DIR__.'/../bootstrap/app.php';

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();

$kernel->terminate($request, $response);

```

```
// bootstrap/app.php
<?php

$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;

```

首先看`bootstrap/app.php`，其中`$app`是`Illuminate\Foundation\Application`的一个实例，而`Illuminate\Foundation\Application`类继承自Container，所以`$app`实际上就是一个Service Container。
然后下面的三个singleton方法定义了当依赖`Illuminate\Contracts\Http\Kernel`、`Illuminate\Contracts\Console\Kernel`、`Illuminate\Contracts\Debug\ExceptionHandler`这三个接口时，注入哪个类的单例。
然后看`public/index.php`,其中的make方法实际上就是用Service Container来new一个`Illuminate\Contracts\Http\Kernel`实例，跟普通new的区别就是会把他的依赖自动注入进去。

是不是很简洁？

其实不单是Laravel，像Yii2、Phalcon等框架都通过实现容器来管理依赖注入。

## 如何使用Service Container ##

既然是一个容器，无非就是两个事，往里放东西和往外取东西，对应到Service Container就是绑定(Binding)和解析(Resolving)。

### 获得容器

在Laravel的Service Provider中，可以通过`$this->app`获取容器，除此之外，还可以使用`app()`来获取容器。
如果在Laravel外使用Service Container，直接new一个`Illuminate\Container\Container`就可以获得容器了。

以下都用$container代表获取到的容器。

### 绑定

 - 绑定返回接口的实例

```
//使用闭包
$container->bind('BarInterface', function(){
    return new Bar();
});
//或者使用字符串
$container->bind('FooInterface', 'Foo');
```

 - 绑定单例

> singletion 方法绑定一个只会被解析一次的类或接口至容器中，且后面的调用都会从容器中返回相同的实例：

```
$container->singleton('BarInterface', function(){
    return new Bar();
});
```

 - 绑定实例

> 你也可以使用 instance 方法，绑定一个已经存在的对象实例至容器中。后面的调用都会从容器中返回指定的实例：

```
$bar = new Bar();
$bar->setSomething(new Something);

$container->instance('Bar', $bar);
```

 - 情境绑定

> 有时候，你可能有两个类使用到相同接口，但你希望每个类都能注入不同实现。

```
$container->when('Man')
          ->needs('PartnerInterface')
          ->give('Woman');
$container->when('Woman')
          ->needs('PartnerInterface')
          ->give('Man');
```

 - 标记

> 有些时候，可能需要解析某个「分类」下的所有绑定。

```
$container->bind('Father', function () {
    //
});
$container->bind('Mother', function () {
    //
});
$container->bind('Daughter', function () {
    //
});
$container->bind('Son', function () {
    //
});
$container->tag(['Father', 'Mother', 'Daughter', 'Son'], 'familyMembers');

$container->bind('Family', function ($container) {
    return new Family($container->tagged('familyMembers'));
});

```

### 解析

 - make方法

```
$foo = $container->make('Foo');
```

 - 数组方法

```
$bar = $container['Bar'];
```

 - 解析被标记绑定

```
$familyMembers = $container->tagged('familyMembers');

foreach ($familyMembers as $individual) {
    $individual->doSomething();
}
```

### 解析事件

> 每当服务容器解析一个对象时就会触发事件。你可以使用 resolving 方法监听这个事件。

```
$container->resolving(function ($object, $container) {
    // 当容器解析任何类型的对象时会被调用...
});

$container->resolving('Foo', function (Foo $foo, $container) {
    // 当容器解析「Foo」类型的对象时会被调用...
});
```

  [1]: http://haitian299.github.io/2016/05/17/Dependency-injection/