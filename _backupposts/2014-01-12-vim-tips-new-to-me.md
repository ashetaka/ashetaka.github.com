---
layout: post
title: "vim tips"
description: ""
category: ""
tags: [vim]
---
{% include JB/setup %}

本文介绍一些vim技巧

### 基本

1. :e 编辑

我们知道要编辑另一个文件，可以用下面的命令
{% highlight sh startinline linenos  %}
:e file2
{% endhighlight %} 

但是:e还有一种别的用法就是用于恢复文件到最初的版本,只要加上感叹号即可，文件就会恢复到你最初打开文件时的版本
{% highlight sh startinline linenos  %}
:e!
{% endhighlight %}

2. :sav 另存为

平时都是在shell下用cp file1 file2，然后再打开file2进行编辑保存。实际上更简单的方法是用:sav来另存为。

{% highlight sh startinline linenos  %}
:sav file2
{% endhighlight %} 

3. . 重复上一次操作

这个很简单就是用于重复上一次的操作，可以在'.'前面加上数字，表示将上一次的操作重复指定次数
{% highlight sh startinline linenos  %}
5.
{% endhighlight %} 

### 文件内的移动操作

|命令|功能描述|
|:---|:---|
|e|移动光标到单词末尾|
|b|移动光标到单词开头|
|0|移动光标到行首|
|G|移动光标到文件末尾|
|gg|移动光标到文件首部|
|L|移动光标到屏幕底部|
|20\||移动20列|
|%|移动光标到匹配的括号处|
|[[|跳到函数开头|
|[{|跳刀块开头|

补充

|命令|功能描述|
|:---|:---|
|^|移动光标到行首|
|$|移动光标到行尾|

### 剪切，复制和粘贴

yy是复制当前行，y是复制选中的文本到剪贴板，注意是复制到剪贴板了，所以不能用p来粘贴

|命令|功能描述|
|:---|:---|
|y|复制选中文本到剪贴板|
|y$|复制光标到行尾的内容|
|D|删除光标到行尾的内容|

### 查找


/word是从上到下查找word，?word则是从下到上查找

|命令|功能描述|
|:---|:---|
|*|查找光标所在位置的单词|
|/\cstring |对大小写不敏感的查找|
|/jo[ha]n |查找json或joan|
|/\<the |查找the开头的内容|
|/the\> |查找the结尾的内容|
|/\<....\> |查找长度为4的单词|
|/\<fred\> |严格匹配，fred可以被找到，alfred和frederick都不会被匹配上|
|/\<\d\d\d\d\> |查找4个数字|
|/\D\d\d\d\d\D |查找4个数字|
|/\<\d\{4}\> |同上|
|/fred\\\|joe |查找fred或joe|
|/^\n\{3}| 查找3个空行|


