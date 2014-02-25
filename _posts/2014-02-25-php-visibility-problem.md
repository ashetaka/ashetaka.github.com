---
layout: post
title: "子类继承函数覆盖父类同名函数问题"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

先看一下 php.net 上的一个例子

{% highlight php startinline linenos  %}
<?php 
class Bar 
{
    public function test() {
        $this->testPrivate();
        $this->testPublic();
    }

    public function testPublic() {
        echo "Bar::testPublic\n";
    }
    
    private function testPrivate() {
        echo "Bar::testPrivate\n";
    }
}

class Foo extends Bar 
{
    public function testPublic() {
        echo "Foo::testPublic\n";
    }
    
    private function testPrivate() {
        echo "Foo::testPrivate\n";
    }
}

$myFoo = new foo();
$myFoo->test(); // Bar::testPrivate 
                // Foo::testPublic
?>
{% endhighlight %} 

可以看到父类 Bar 和子类 Foo各自都定义了 testPublic 方法和 testPrivate 方法，可为什么一个调用的是 Bar 里的方法，而另一个调用的是 Foo 呢？

原因就在于

>Private limits visibility only to the class that defines the item

这意味着子类是看不到父类的私有方法的，反之亦然。结果就是，当父类和子类以不同的方式实现了相同名称的私有方法时，程序结果只取决于你从哪调用它。因为私有属性已限制你看到另一个同名方法，既然不可见，也就不存在方法覆盖一说了。

但如果想让继承函数使用的是子类中的覆盖方法，而你又觉得 public 太宽松的话，使用 protected 就可以了。

下面是一个例子

{% highlight php startinline linenos  %}
<?php 
abstract class base { 
    public function inherited() { 
        $this->overridden(); 
    } 
    protected function overridden() { 
        echo 'base'; 
    } 
} 

class child extends base { 
    protected function overridden() { 
        echo 'child'; 
    } 
} 

$test = new child(); 
$test->inherited(); 
?> 
{% endhighlight %} 

最后
>同一个类的对象即使不是同一个实例也可以互相访问对方的私有与受保护成员。这是由于在这些对象的内部具体实现的细节都是已知的。


