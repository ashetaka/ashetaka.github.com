---
layout: post
title: "python中pow()与math.pow()的不同"
description: ""
category: ""
tags: [python]
---
{% include JB/setup %}

下午在[projecteuler](http://projecteuler.net/)做题的时候用了下math.pow()这个函数，发现实际结果与预期的不一致，计算结果是用科学计数法表示的。想起以前应该不是这样的，又改成内建的pow()试了下，发现这回能返回完整的计算结果。至此，才发现这两个函数还是有一点差别的。

### 小测试

先用pydoc分别砍下两个函数的描述是否一致：

{% highlight sh startinline linenos  %}
pow(...)
    pow(x, y[, z]) -> number

math.pow = pow(...)
    pow(x, y)
{% endhighlight %} 

看起来具体的实现果然有点差别呢，再进交互式界面看一下：

{% highlight sh startinline linenos  %}
>>> import math
>>> pow is math.pow
False
{% endhighlight %} 

至此，我们可以看到两点：

1. 两者并非是同一个函数
2. 非但不是同一个，具体的实现也有差异

### 继续测试

首先我们拿0和-1作为参数来看一下返回的结果

{% highlight sh startinline linenos  %}
>>> pow(0,-1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: 0.0 cannot be raised to a negative power
>>> math.pow(0,-1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: math domain error
{% endhighlight %} 

很显然，0的-1次是没法计算的。python也分别都返回了错误，可是返回的报错信息似乎不太一样。

当然0，-1这只是其中一个样例，还有别的例子，比如浮点数、长整形等，在此不一一举例了。

### 深入math.pow()

从最开始的pydoc里我们已经发现，math.pow()接收的参数比pow()要少一个。但假如我们抛开那个参数的差别来看，就会发现math.pow()处理参数的方法也有一点不同。从[源码](http://hg.python.org/cpython/file/c7163a7f7cd2/Modules/mathmodule.c#l1781)可以看到，math.pow()直接把传入的参数转为double

{% highlight py startinline linenos  %}
static PyObject *
math_pow(PyObject *self, PyObject *args)
{
    PyObject *ox, *oy;
    double r, x, y;
    int odd_y;

    if (! PyArg_UnpackTuple(args, "pow", 2, 2, &ox, &oy))
        return NULL;
    x = PyFloat_AsDouble(ox);
    y = PyFloat_AsDouble(oy);
/*...*/
{% endhighlight %} 

这里截取了该函数最开始的几行代码，可以看到传入后就被传给了double型变量，之后才进行下一步计算。

### 内建的pow()

内建的pow()其实与`**`操作符是一个概念，它直接使用了`**`操作符的实现方式。针对不同的参数类型，`**`用了不同的实现方法，如[float](http://hg.python.org/cpython/file/6a60359556f9/Objects/floatobject.c#l807),[long](http://hg.python.org/cpython/file/c7163a7f7cd2/Objects/longobject.c#l3599),[complex](http://hg.python.org/cpython/file/c7163a7f7cd2/Objects/complexobject.c#l510)

另外，如果有需要的话，可以用`__pow__()`,`__rpow()`或者`__ipow__()`方法来重写实现方式

### 覆盖默认行为

[官网docs](http://docs.python.org/reference/datamodel.html#numeric-types)里有对数值类型的模拟操作讨论。如果你不确定传入参数的类型，你就必须为这个类型提供`__pow__()`等方法，这样你就可以使用这个操作符了。

### 总结

细微的差别需要实际的使用来发现.

Practice is th way to the master.