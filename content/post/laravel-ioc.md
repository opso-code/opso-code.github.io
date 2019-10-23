---
layout: post
title: Laravel依赖注入和控制反转
date: 2017-04-20
author: opso
tags:
- php
- laravel
---

**IOC**( inversion of controller )叫做控制反转模式，也可以称为(dependency injection ) 依赖注入模式。这也是**Laravel**框架的核心“容器IOC”概念。

<!--more-->

这里通过 bind 方式绑定了一些依赖关系， 然后通过make 方法 获取到到我们想要的实例. 在make中有牵扯到了闭包函数，反射等概念。

举例：

```php
<?php
//支付类接口
interface Pay {
    public function pay();
}

//支付宝支付
class Alipay implements Pay {
    public function __construct() {
    }

    public function pay() {
        echo 'pay bill by alipay';
    }
}

//微信支付
class Wechatpay implements Pay {
    public function __construct() {
    }

    public function pay() {
        echo 'pay bill by wechatpay';
    }
}

//银联支付
class Unionpay implements Pay {
    public function __construct() {
    }

    public function pay() {
        echo 'pay bill by unionpay';
    }
}

//付款
class PayBill {
    private $payMethod;

    public function __construct(Pay $payMethod) {
        $this->payMethod = $payMethod;
    }

    public function payMyBill() {
        $this->payMethod->pay();
    }
}

//生成依赖
$payMethod = new Alipay();
//注入依赖
$pb = new PayBill($payMethod);
$pb->payMyBill();
```

把class Alipay 的实例通过constructor注入的方式去实例化一个 class PayBill. 在这里我们的注入是手动注入, 不是自动的. 而Laravel 框架实现则是自动注入。

**反射**(reflection)

「 反射它指在PHP运行状态中，扩展分析PHP程序，导出或提取出关于类、方法、属性、参数等的详细信息，包括注释。这种动态获取的信息以及动态调用对象的方法的功能称为反射API。反射是操纵面向对象范型中元模型的API，其功能十分强大，可帮助我们构建复杂，可扩展的应用。其用途如：自动加载插件，自动生成文档，甚至可用来扩充PHP语言」

下面通过一个例子来进一步了解IOC容器

```php
<?php

//容器类装实例或提供实例的回调函数
class Container {
    //用于装提供实例的回调函数，真正的容器还会装实例等其他内容
    //从而实现单例等高级功能
    protected $bindings = [];

    //绑定接口和生成相应实例的回调函数
    public function bind($abstract, $concrete = null, $shared = false) {
        //如果提供的参数不是回调函数，则产生默认的回调函数
        if (!$concrete instanceof Closure) {
            $concrete = $this->getClosure($abstract, $concrete);
        }

        $this->bindings[$abstract] = compact('concrete', 'shared');
    }

    //默认生成实例的回调函数
    protected function getClosure($abstract, $concrete) {
        return function ($c) use ($abstract, $concrete) {
            $method = ($abstract == $concrete) ? 'build' : 'make';
            return $c->$method($concrete);
        };
    }

    public function make($abstract) {
        $concrete = $this->getConcrete($abstract);
        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete);
        } else {
            $object = $this->make($concrete);
        }

        return $object;
    }

    protected function isBuildable($concrete, $abstract) {
        return $concrete === $abstract || $concrete instanceof Closure;
    }

    //获取绑定的回调函数
    protected function getConcrete($abstract) {
        if (!isset($this->bindings[$abstract])) {
            return $abstract;
        }
        return $this->bindings[$abstract]['concrete'];
    }

    //实例化对象
    public function build($concrete) {
        if ($concrete instanceof Closure) {
            return $concrete($this);
        }
        $reflector = new ReflectionClass($concrete);
        if (!$reflector->isInstantiable()) {
            echo $message = "Target [$concrete] is not instantiable";
        }
        $constructor = $reflector->getConstructor();
        if (is_null($constructor)) {
            return new $concrete;
        }
        $dependencies = $constructor->getParameters();
        $instances = $this->getDependencies($dependencies);
        return $reflector->newInstanceArgs($instances);
    }

    //解决通过反射机制实例化对象时的依赖
    protected function getDependencies($parameters) {
        $dependencies = [];
        foreach ($parameters as $parameter) {
            $dependency = $parameter->getClass();
            if (is_null($dependency)) {
                $dependencies[] = NULL;
            } else {
                $dependencies[] = $this->resolveClass($parameter);
            }
        }
        return (array)$dependencies;
    }

    protected function resolveClass(ReflectionParameter $parameter) {
        return $this->make($parameter->getClass()->name);
    }
}

// 上面的代码就生成了一个容器,下面是如何使用容器
$app = new Container();
$app->bind("Pay", "Alipay");//Pay 为接口， Alipay 是 class Alipay
$app->bind("tryToPayMyBill", "PayBill"); //tryToPayMyBill可以当做是Class PayBill 的服务别名
//通过字符解析，或得到了Class PayBill 的实例
$paybill = $app->make("tryToPayMyBill");
//因为之前已经把Pay 接口绑定为了 Alipay，所以调用pay 方法的话会显示 'pay bill by alipay '
$paybill->payMyBill();
$app->bind("Pay", "Alipay");
$app->bind("tryToPayMyBill", "PayBill");
$paybill = $app->make("tryToPayMyBill");
$paybill->payMyBill();
```