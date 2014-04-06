---
layout: post
title: "PHP中的函数式编程"
description: ""
category: ""
tags: [php]
---
{% include JB/setup %}


提到函数式编程，也许脑海中最先出现的是Haskell、Lisp这些函数式语言，不过其实PHP也支持函数式风格的一些编程实践。本文就对其中的一些概念做简单实现来介绍PHP中的函数式编程。

### 匿名函数

一个简单的匿名函数实现方式如下

{% highlight hs startinline %}
<?php
    $func = function ($a){
        echo $a;
    };

    $func('haha');
?>
{% endhighlight %} 

这个例子定义了一个输出指定字符串的匿名函数。

接下来来看一个类内部定义匿名函数的例子

{% highlight hs startinline %}
<?php
class Someclass{
    public $value = '2014';

    public function foo(){
        echo "foo is called\n";
    }

    public function run(){
        $func = function(){
            return $this->value;
        };
        $res = $func();
        echo $res . "\n";
        $this->foo();
    }
}

$test = new Someclass();
$test->run();
?>
{% endhighlight %} 

可以看到类内定义的匿名函数式可以用$this来访问类的成员变量和方法。这个特性在PHP5.4以后的版本都支持。

### 闭包

先来看一个例子

{% highlight hs startinline %}
<?php
    $bind = 3;
    $closure = function ($arg) use($bind){
        return $arg + $bind;
    };
    echo $closure(4);
?>
{% endhighlight %} 

如上面的例子，我们用关键字use来捆绑变量。PHP中的捆绑默认是前期绑定(early binding)。这意味着匿名函数接受到的值是函数定义时该变量的值。我们也可以用引用来传递变量，并以此来实现后期绑定(late binding)。看看下面的例子:

{% highlight hs startinline %}
<?php
$time = "morning!\n";

$func = function() use(&$time){
    echo "good " . $time;
};

$func();

$time = "afternoon!\n";
$func();
?>
{% endhighlight %} 

这段代码的输出是

{% highlight sh startinline %}
good morning!
good afternoon!
{% endhighlight %} 

可以看到把$time的值修改以后，匿名函数接受到的绑定参数的值也变了。

### 高阶函数

高阶函数是指把函数作为参数的函数。array_map就是一个常见的高阶函数的例子：

{% highlight hs startinline %}
<?php
function func($v){
    if ($v == 6 || $v == 7)
        return "weekend";
    else
        return "workday";
}

$arr = array(1,2,3,4,5,6,7);
print_r(array_map("func", $arr));
?>
{% endhighlight %} 

在这个例子里，我们定义了func：用来判断传入的日子是工作日还是周末。然后把数据和func函数传给array_map，得到以下输出。

{% highlight sh startinline %}
Array
(
    [0] => workday
    [1] => workday
    [2] => workday
    [3] => workday
    [4] => workday
    [5] => weekend
    [6] => weekend
)
{% endhighlight %} 

### 实例化一个匿名函数类

假如我们有一个类，每次实例化的时候给他传一个匿名函数作为参数，这种情况下该怎么实现以及调用呢。答案是借用魔术方法中的`__invoke()`方法。举个例子：

{% highlight hs startinline %}
<?php
class Lambda{
    private $anonymous;

    public function __construct($f){
        $this->anonymous = $f;
    }
    public function __invoke(){
        $x = func_get_args();
        return call_user_func_array($this->anonymous, $x);
    }
}

$a = new Lambda(
    function ($x){
        return $x * $x;
    }
);

echo $a(5) . "\n";
?>
{% endhighlight %} 

由例子可以看到，实例化Lanbda类的时候，把传入的匿名函数付给anonymous；为了实现调用，__invoke方法就发挥作用了，调用ananymous内的函数并将结果返回。

### 总结

虽然PHP不是函数式语言，但只要用函数式编程的方式去思考和实现程序，你的PHP代码一样可以是函数式的。本文的几个例子简单介绍了函数式编程中各个概念的简单实现，虽然对实际项目帮助不大，但是你总能在你的项目中找到合适的场合去应用这些概念，相信这也能为你的代码带来函数式风格的好处。

Think functionally.