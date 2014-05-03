---
layout: post
title: "PHP40个有用建议-2"
description: ""
category: "译"
tags: [php]
---
{% include JB/setup %}

### 11.不要在程序里用gzip压缩你的输出，让apache去做那件事

考虑使用ob_gzhandler?千万别那样做。那没有意义。php应该做程序自身的事。不要担心如何优化php内部服务器和浏览器之间的数据传输。

在.htaccess文件里用apache的mod_gzip/mod_deflate来压缩内容。

### 12.用php输出js代码时使用json_encode

有时候我们需要从php动态生成一些js代码。

{% highlight php startinline linenos  %}
$images = array(
	'myself.png' , 'friends.png' , 'colleagues.png'
);

$js_code = '';

foreach ($images as $image){
	$js_code .= "'$image' ,";
}

$js_code = 'var images = [' . $js_code . ']; ';

echo $js_code;

// 输出是var images = ['myself.png' ,'friends.png' ,'colleagues.png' ,];
{% endhighlight %}

聪明点的做法是使用json_encode:

{% highlight php startinline linenos  %}
$images = array(
	'myself.png' , 'friends.png' , 'colleagues.png'
);

$js_code = 'var images = ' . json_encode($images);

echo $js_code;

// 输出是var images = ['myself.png' ,'friends.png' ,'colleagues.png' ,];
{% endhighlight %} 

这样是不是简洁多了？

### 13.写入文件前检查目录是否可写

在写入或是保存任何文件前，确保你有检查目录是否可写，如果不可写就提示错误信息。这会为你节约许多调试时间。当你在linux下工作时，必须要处理权限问题，而且当目录不可写，或是文件不可读，诸如此类的情况时，会有很多权限问题。

确保你的程序足够智能，能够在最短的时间里反馈最重要的信息。

{% highlight php startinline linenos  %}
$contents = "All the content";
$file_path = "/var/www/project/content.txt";

file_put_contents($file_path , $contents);
{% endhighlight %} 

完全正确地代码。但是有一些间接问题。file_put_contents可能会因为一些原因失败：

- 父目录不存在
- 目录存在，但是不可写
- 文件被锁定了不让写入

因此在写入文件前最好弄清一切问题。

{% highlight php startinline linenos  %}
$contents = "All the content";
$dir = '/var/www/project';
$file_path = $dir . "/content.txt";

if (is_writable($dir)){
	file_put_contents($file_path , $contents);
}else{
	die("Directory $dir is not writable, or does not exist. Please check");
}
{% endhighlight %} 

这样你可以得到关于哪里导致文件写入失败以及为什么的详细信息。

### 14.为自己程序生成的文件修改权限

当你工作在linux环境下时，权限处理可能会浪费你很多时间。因此无论何时，当你的php程序生成了一些文件，做chmod操作来确保他们是可被访问的。否则，比如说，文件由PHP所对应的用户创建，而你又用不同的用户来工作，系统不会允许你访问或打开这个文件，然后你不得不拿到root权限，改变文件的权限等等。

{% highlight php startinline linenos  %}
// 所有者读写权限，别的用户仅有读权限
chmod("/somedir/somefile", 0644);

// 所有者所有权限，别的用户读权限和执行权限
chmod("/somedir/somefile", 0755);
{% endhighlight %} 

### 15.不要用检查提交按钮的值来检查form的提交

{% highlight php startinline linenos  %}
if ($_POST['submit'] == 'Save'){
	//Save the things
}
{% endhighlight %} 

上面的代码大体上时正确地，除非你的程序是多语言的。这样‘Save’可能会是别的不同的东西。那么你如何比较呢。别依赖提交按钮的值，而是用下面的方法：

{% highlight php startinline linenos  %}
if ($_SERVER['REQUEST_METHOD'] == 'POST' and isset($_POST['submit'])){
	//Save the things
}
{% endhighlight %} 

现在你可以摆脱提交按钮的值了。

### 16.函数中值固定时请使用静态变量

{% highlight php startinline linenos  %}
// Delay for some time
function delay(){
	$sync_delay = get_option('sync_delay');

	echo "<br />Delaying for $sync_delay second...";
	sleep($sync_delay);
	echo "Done <br />";
}
{% endhighlight %} 

用静态变量来代替上面的做法：

{% highlight php startinline linenos  %}
// Delay for some time
function delay(){
	static $sync_delay = null;

	if ($sync_delay == null){
		$sync_delay = get_option('sync_delay');
	}

	echo "<br />Delaying for $sync_delay second...";
	sleep($sync_delay);
	echo "Done <br />";
}
{% endhighlight %} 

### 17.不要直接使用$_SESSION

一个简单的例子：

{% highlight php startinline linenos  %}
$_SESSION['username'] = $username;
$username = $_SESSION['username'];
{% endhighlight %} 

但是这有一个问题。如果你在同一个域名下运行多个程序，session变量可能会发生冲突。两个不同的程序可能会在session变量里设置同一个键。比如说，一个前台入口，一个后端管理程序，在同一域名下。

因此用封装函数来指定键吧：

{% highlight php startinline linenos  %}
define('APP_ID' , 'abc_corp_ecommerce');

//Function to get a session variable
function session_get($key){
	$k = APP_ID . '.' . $key;

	if (isset($_SESSION[$k])){
		return $_SESSION[$k];
	}

	return false;
}

//Function to set the session variable
function session_set($key , $value){
	$k = APP_ID . '.' . $key;
	$_SESSION[$k] = $value;

	return true;
}
{% endhighlight %} 

### 18.把实用辅助函数封装到类里

你在一个文件中有许多这样的实用函数：

{% highlight php startinline linenos  %}
function utility_a(){
	//
}
function utility_b(){
	//
}
function utility_c(){
	//
}
{% endhighlight %} 

你能在整个程序中自由地使用这些函数。你可能会想要把他们封装到类里作为静态函数：

{% highlight php startinline linenos  %}
class Utility{
	public static function utility_a(){
		//
	}
	public static function utility_b(){
		//
	}
	public static function utility_c(){
		//
	}	
}

//and call them as

$a = Utility::utility_a();
$b = Utility::utility_b();
{% endhighlight %} 

这么做的一个明显好处是，如果php内置函数中有同名的情况，那么并不会产生命名冲突。

另一个角度来说，虽然优势不大，但是你可以在同一个程序中无任何冲突地维护同一个类的不同版本。这主要是封装，仅此而已。

### 19.一些蠢建议

- 用echo代替print
- 使用str_replace代替preg_replace，除非你真的需要它
- 不要用短标签
- 对简单地字符串使用单引号而不是双引号
- 永远记得在用header做重定向以后执行exit
- 永远不要把函数调用放在for循环的控制语句中
- isset比strlen快
- 正确地一贯地格式化你的代码
- 不要省略循环语句或是if-else语句的括号

不要写这样的代码：

{% highlight php startinline linenos  %}
if ($a == true)  $a_count++;
{% endhighlight %} 

这完全是浪费

应该这样子

{% highlight php startinline linenos  %}
if ($a == true){
	$a_count++;
}
{% endhighlight %} 

不要通过吃掉语法来使代码更短。宁愿让你的逻辑更短。

- 使用合适的有代码高亮的文本编辑器。代码高亮有助于少犯错。

### 20.用arrat_map更快地处理数组

假定现在你想要trim数组中的所有元素。新手会这样做：

{% highlight php startinline linenos  %}
foreach ($arr as $c => $v){
	$arr[$c] = trim($v);
}
{% endhighlight %} 

但是用array_map可以更简洁：

{% highlight php startinline linenos  %}
$arr = array_map('trim' , $arr);
{% endhighlight %} 

这段代码可以将trim应用到数组$arr的所有元素。另一个相似的函数是array_walk。查看这些函数的文档来了解更多内容。

### 21.用php过滤器去验证数据

你应该有过用正则表达式去验证邮箱或是ip地址之类的数据。是的大家都曾经那样做过。现在我们来尝试点不同的东西，叫做过滤器。

php过滤器为验证或检查值是否为合法的东西提供了简单的方法。

### 22.强制类型检查

{% highlight php startinline linenos  %}
$amount = intval( $_GET['amount']);
$rate = (int) $_GET['rate'];
{% endhighlight %} 

这是个好习惯

### 23.用set_error_handler()来写入php错误到文件中

set_error_handler()可以被用来指定一个错误处理函数。把一些重要的错误写到文件里作为日志是个不错的想法。

### 24.小心处理大数组

大数组或是大字符串，如果变量里的内容很大，处理时需要十分小心。常见的错误是产生一个拷贝然后耗尽了内存并得到一个"Memory size exceeded"的致命错误：

{% highlight php startinline linenos  %}
$db_records_in_array_format; //这个大数组存放着某张表中20列1000行的数组，每一列至少100bytes，所以总共 1000*20*100=2MB

$cc = $db_records_in_array_format; // 2MB或者更多

some_function($cc); // 又来个2MB?
{% endhighlight %} 

上面的情况在导入csv文件或是输出表内容到csv文件时非常常见。

上面的做法会经常由于内存限制导致脚本crash。对于小变量，这不是个问题，但处理大数组时必须要避免这种情况。

考虑通过引用来传递他们，或者把他们存到一个类的成员变量里：

{% highlight php startinline linenos  %}
$a = get_large_array();
pass_to_function(&$a);
{% endhighlight %} 

这么做可以让函数接收到同一个值(而不是它的复制)。查看[文档](http://php.net/manual/en/language.references.pass.php)

{% highlight php startinline linenos  %}
class A{
	function first(){
		$this->a = get_large_array();
		$this->pass_to_function();
	}

	function pass_to_function(){
		//process $this->a
	}
}
{% endhighlight %} 

尽可能早地unset这些大变量，以便内存被及时释放，脚本的剩余部分能顺利执行。

这是个简单的例子，来证明一些情况下引用分配的变量如何节省内存

{% highlight php startinline linenos  %}
<?php

ini_set('display_errors' , true);
error_reportint(E_ALL);

$a = array();

for ($i = 0; $i < 100000 ; $i++){
	$a[$i] = 'A' . $i;
}

echo 'Memory usage in MB : ' . memory_get_usage() / 1000000 . '<br />';

$b = $a;
$b[0] = 'B';

echo 'Memory usage in MB after 1st copy : ' . memory_get_usage() / 1000000 . '<br />';

$c = $a;
$c[0] = 'B';

echo 'Memory usage in MB after 2st copy : ' . memory_get_usage() / 1000000 . '<br />';

$d = & $a;
$d[0] = 'B';

echo 'Memory usage in MB after 3st copy (reference) : '. memory_get_usage() / 1000000 . '<br />';
{% endhighlight %} 

在php5.4机器上运行的结果为：

{% highlight sh startinline linenos  %}
	Memory usage in MB : 18.08208
	Memory usage in MB after 1st copy : 27.930944
	Memory usage in MB after 2st copy : 37.779808
	Memory usage in MB after 3st copy (reference) : 37.779864
{% endhighlight %}

由此可以看到第三次通过引用实现的复制节省了内存。否则纯复制情形下内存会被用的越来越多知道耗尽。

### 25.整个脚本只用单个数据库连接

确保整个脚本中你只用了单个数据库的连接。在最开始打开连接，一直用到最后，并且关闭。不要像这样在函数内打开连接：

{% highlight php startinline linenos  %}
function add_to_cart(){
	$db = new Database();
	$db->query("INSERT INTO cart...");
}

function empty_cart(){
	$db = new Database();
	$db->query("DELETE FROM cart ...");
}
{% endhighlight %} 

拥有多个连接是种不好的做法，而且由于每个连接花费时间去创建并占用更多内存，他们还减慢了执行速度。

对像数据库连接这样的特定场景使用单例模式。

原文链接：
[40+ Useful Php tips for beginners – Part 2](http://www.binarytides.com/40-techniques-to-enhance-your-php-code-part-2/)

