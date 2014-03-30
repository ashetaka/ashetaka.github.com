---
layout: post
title: "php从大文件读取A行到B行"
description: ""
category: ""
tags: [php]
---
{% include JB/setup %}

### 背景

因为评测的数据文件基本都很大，而实际评测的时候有时候只需要抽样。在没有特殊抽样要求的情况下，有时候想把大文件的某行到另一行的数据给读出来并做相应地分析操作，这时候若是用常规的fopen+fgets效率很低。在网上找解决方案的时候发现了SplFileObject这个利器。

ps：linux下sed是更简便的实现方法 不过本文讨论php下的实现思路

### 简介

SplFileObject类为文件提供了一个面向对象接口。他定义了一系列接口，甚至还提供了一些内置类，对文件编程有极大简化作用。

对于所有的内置类，可以这样来查看：

{% highlight php startinline   %}
<?php
	foreach (spl_classes() as $key=>$value){
		echo $key . '->' . $value . '\n';
	}
?>	
{% endhighlight %} 

### 实现

回到我们的问题上来，要实现读取文件X的A行到B行的数据，需要用到SplFileObject::fseek。

对于这个方法，php.net上的介绍是这样的

public int SplFileObject::fseek ( int $offset [, int $whence = SEEK_SET ] )
Seek to a position in the file measured in bytes from the beginning of the file, obtained by adding offset to the position specified by whence.

表面上看好像和原生的fseek方法没有太大差别。可是实际上SplFileObject::fseek中的offset并不是字节数，而是行数，这点与原生的fseek方法是有很大差别的。不知道官网这里为什么写成了bytes。

看到这里，思路就很清晰了。用SplFileObject::fseek直接定位到A行，然后开始读，直到B行，然后直接结束。

### 具体代码

先看常规的fgets方法的实现
{% highlight php startinline  %}
function getSpecLinesGeneral($file, $start = 1, $end = 10){
        $content = array();
        $line_cnt = $end - $start;
        $fp = fopen($file, "r");
        if (!$fp){
                return 'error:' . $file . ' can not be opened';
        }
        for ($i = 1; $i < $start; $i++){
                fgets($fp);
        }
        for (; $i <=$end; $i++){
                $content[] = fgets($fp);
        }
        fclose($fp);

        return $content;
}
{% endhighlight %} 


再看看用SplFileObject实现的方法
{% highlight php startinline  %}
function getSpecLines($file, $start = 0, $end = 10){
    $content = array();
    $line_cnt = $end - $start;
    $fp = new SplFileObject($file, "r");
    $fp->seek($start - 1); 
    for ($i = 0;$i <= $line_cnt; $i++){
        $content[] = $fp->current();
        $fp->next();

    }   

    return array_filter($content);
}
{% endhighlight %} 

可以看到常规方法用了两次for循环：第一次先扫描掉前($start-1)行，第二次扫描的才是我们想要的结果。

用了SplFileObject的方法2则先用内置的seek直接定位到$start行，然后扫描指定行。`$fp->seek($start - 1);`此处减1是因为offset是从0开始计数的 。

### 测试对比

针对上述两种方法，做了两次测试，第一次只读取两行，第二次读取100000行。

对于每次测试，又按起始位置不同做了分组测试，结果如下

|起始位置和结束位置|SplFileObject方法|fgets方法|
|---|---|---|
|1行到2行|0.2ms|0.1ms|
|10行到11行|0.1ms|0.1ms|
|100行到101行|0.1ms|0.3ms|
|1000行到1001行|0.5ms|2.8ms|
|10000行到10001行|3.7ms|27.9ms|
|100000行到100001行|35ms|280.5ms|
|1000000行到1000001行|286.6ms|2029.9ms|
|2000000行到2000001行|429.2ms|4165.8ms|
|3000000行到3000001行|643.1ms|6027.3ms|
|4000000行到4000001行|848.8ms|8168.4ms|

<br>

|起始位置和结束位置|SplFileObject方法|fgets方法|
|---|---|---|
|1行到100001行|660ms|308ms|
|10行到100010行|435.2ms|244.2ms|
|100行到100100行|435ms|248.8ms|
|1000行到101000行|445.2ms|252.9ms|
|10000行到110000行|452.1ms|274.5ms|
|100000行到200000行|475.5ms|454.8ms|
|1000000行到1100000行|703.7ms|2365.7ms|
|2000000行到2100000行|928.6ms|4320.4ms|
|3000000行到3100000行|1132.5ms|6288.8ms|
|4000000行到4100000行|1380.3ms|8461.9ms|

<br>

### 最后

实际使用的情况来看，如果是要读取的内容在文件的前面部分，用SplFileObject的优势并不明显，甚至可能还不如常规的fgets方式。只有在要读取的内容是文件的后面部分时，此种方法才有优势，而且越是start行越靠后，优势越明显。