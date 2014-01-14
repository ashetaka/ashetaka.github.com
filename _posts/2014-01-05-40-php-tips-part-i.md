---
layout: post
title: "PHP40个有用建议-1"
description: ""
category: "译"
tags: [php]
---
{% include JB/setup %}

在这个系列中我们将看到一些有用的小贴士和技术来提升和优化你得php代码。请注意这些php小贴士对初学者是有意义的，而不是哪些已经在用mvc框架的人。

### 1.不要使用相对路径，以定义根路径代替

这样的代码行是很常见的：

{% highlight php startinline linenos  %}
require_once('../../lib/some_class.php');
{% endhighlight %}	

这种做法有很多弊端：

它首先查找php的include_path中定义的目录，然后才从当前目录查找。因此检查了很多目录。

当一个脚本被另一个脚本包含在不同的目录中，它的基础路径变为了那个包含的脚本所在的位置。

另一个问题是，当一个脚本是从cron启动的，它可能找不到父目录作为工作目录。

因此用绝对路径是个好主意：

{% highlight php startinline linenos  %}
define('ROOT','/var/www/project');
require_once(ROOT . '../../lib/some_class.php');

//代码剩余部分
{% endhighlight %}	

现在这是个绝对路径了并且会保持不变。但是我们可以改进更多。/var/www/project这个目录要是变了，难道我们每次都去修改它吗？不。相反，我们通过使用__FILE__这样的魔法常量来使它更方便。仔细看看：

{% highlight php startinline linenos  %}
//假设脚本为 /var/www/project/index.php
//那么__FILE__ 会试那个完整路径

define('ROOT', pathinfo(__FILE__, PATHINFO_DIRNAME));
require_once(ROOT . '../../lib/some_class.php');

//代码剩余部分
{% endhighlight %}	

因此现在即使你把你的工程移到别的目录，比如移到线上服务器，同样的代码也可以不做任何改变就运行良好。

### 2.不要使用require,include,require_once或者include_ince

你的脚本也许会在顶部包含不同的文件，诸如像这样的类库、公用函数和辅助函数等等：

{% highlight php startinline linenos  %}
require_once('lib/Datavase.php');
require_once('lib/Mail.php');

require_once('helpers/utitlity_functions.php');
{% endhighlight %}	

这是十分粗糙的。代码需要更加灵活些。详细写些辅助函数来更容易地包含文件。举个例子：

{% highlight php startinline linenos  %}
function load_class($class_name){
	//类文件的路径
	$path = ROOT . '/lib' . $class_name . '.php';
	require_once( $path );
}

load_class('Database');
load_class('Mail');
{% endhighlight %}	

看到不同之处了吗？你肯定看到了。不需要更多解释了。

你可以改得更好如果你愿意这样的话：

{% highlight php startinline linenos  %}
function load_class($class_name){
	//类文件的路径
	$path = ROOT . '/lib' . $class_name . '.php';

	if (file_exists($path)){
		require_once($path);
	}
}
{% endhighlight %}	

利用这点可以做很多事情：

- 为同一个类文件查找多个目录。
- 改变包含类文件的目录很容易，并且不会在任何地方破坏代码。
- 使用类似的函数来加载包含辅助函数或是html内容等文件。

### 3.在应用中维护调试环境

在开发期间我们会输出数据库查询语句，或是打印出问题的变量，然后等到问题被解决了，我们会把他们注释掉或是删掉。但是长期来说保留一切会有帮助。

在你的开发机上你可以这样做：

{% highlight php startinline linenos  %}
define('ENVIRONMENT' , 'development');

if (! $db->query( $query )){
	if (ENVIRONMENT == 'development'){
		echo "$query failed";
	}
	else{
		echo "Database error. Please contact administrator";
	}
}
{% endhighlight %}	

然后服务器上你可以这么做：

{% highlight php startinline linenos  %}
define('ENVIRONMENT' , 'production');

if (! $db->query( $query )){
	if (ENVIRONMENT == 'development'){
		echo "$query failed";
	}
	else{
		echo "Database error. Please contact administrator";
	}
}
{% endhighlight %}	

### 4.通过会话(session)来传递状态信息

状态信息是指做完一个任务后产生的信息。

{% highlight html+php startinline linenos  %}
<?php
	if ($wrong_username || $wrong_password){
		$msg = 'Invalid username or password';
	}
?>
<html>
<body>

<?php echo $msg;?>

<form>
...
</form>
</body>
</html>
{% endhighlight %}	

这样的代码是很常见的。使用变量来说明状态信息有局限性。他们不能通过重定向被传送(除非你把他们当做GET变量传到下一个脚本，但这样是很愚蠢的)。在大型脚本中，可能会有许多的信息。

最好的方法是使用会话来传递他们(即使在同一个页面)。对于这种情况，需要在每个页面有个session_start。

{% highlight php startinline linenos  %}
function set_flash($msg){
	$_SESSION['message'] = $msg;
}

function get_flash(){
	$msg = $_SESSION['message'];
	unset($_SESSION['message']);
	return $msg;
}
{% endhighlight %}	

脚本如下:

{% highlight html+php startinline linenos  %}
<?php
	if ($wrong_username || $wrong_password){
		set_flash('Invalid username or password');
	}
?>
<html>
<body>

status is : <?php echo get_flash(); ?>
<form>
...
</form>
</body>
</html>
{% endhighlight %}	

### 5.让你的函数更灵活

{% highlight php startinline linenos  %}
function add_to_cart($item_id,$qty){
	$_SESSION['cart'][$item_id] = $qty;
}

add_to_cart( 'IPHONE3' , 2);
{% endhighlight %}	

当要添加一个物品时你可以用上面的函数。当要添加多个物品时，你会创建另一个函数吗？不用。只要让函数灵活到能够接受不同类别的参数就行了。下面仔细看看：

{% highlight php startinline linenos  %}
function add_to_cart($item , $qty){
	if (!is_array($item_id)){
		$_SESSION['cart'][$item_id] = $qty;
	}
	else{
		foreach ($item_is as $i_id=>$qty){
			$_SESSION['cart'][$i_id] = $qty;
		}
	}
}

add_to_cart( 'IPHONE3' ,2);
add_to_cart( array('IPHONE3'=>2,'IPAD'=>5));
{% endhighlight %}	

所以现在同一个函数可以接受不同类别的输入。上面的方法可以在许多地方应用来让你的代码更敏捷。

### 6.省略php闭合标签如果代码在这结束

我好奇为什么这条贴士在如此多的关于php贴士的博文中被忽略了。

{% highlight php startinline linenos  %}
<?php
	echo "Hello";

// Now dont close this tag
{% endhighlight %}	

这将节省你大量的问题。来看个例子：

类文件super_class.php

{% highlight php startinline linenos  %}
<?php
class super_class{
	function super_function(){
		//super code
	}
}
?>
//super extra character after the closing tag
{% endhighlight %}	

index.php

{% highlight php startinline linenos  %}
require_once('super_class.php');

//echo an image or pdf, or set the cookies or session data
{% endhighlight %}	

现在你会得到“Headers already send error”错误。为什么呢？因为“super extra character”被打印出来了，然后所有的头部跟在那后面。现在你开始调试。你可能会浪费很多时间来找出这个位置。

因此把省略闭合标签作为习惯吧：

{% highlight php startinline linenos  %}
<?php
class super_class{
	function super_function(){
		//super code
	}
}

//No closing tag
{% endhighlight %}	

这样好多了

### 7.在同一个地方收集所有输出，并且一次输出到浏览器上

这就是所谓的输出缓冲。假设你已经像这样从不同的函数输出内容：

{% highlight php startinline linenos  %}
function print_header(){
	echo "<div id='header'>Site Log and Login links</div>";
}
function print_footer(){
	echo "<div id='footer'>Site was made by me</div>";
}

print_header();
for ($i = 0l $i < 100; $i++){
	echo "I is : $i <br />";
}
print_footer();
{% endhighlight %}	

代替那样做，首先收集所有的输出在一个位置。你可以在函数中把它存到变量里，也可以用ob_start和ob_end_clean。因此现在看起来是这样的

{% highlight php startinline linenos  %}
function print_header(){
	$o = "<div id='header'>Site Log and Login links</div>";
	return $o;
}

function print_footer(){
	$o = "<div id='footer'>Site was made by me</div>";
	return $o;
}

echo print_header();
for ($i = 0; $i < 100; $i++){
	echo "I is : $i <br />";
}
echo print_footer();
{% endhighlight %}	

因此你应该做输出缓冲的理由是：

- 如果你需要的话你能在传到浏览器之前修改输出内容。考虑下做一些str_replace操作，或者是preg_replace操作，或者是添加一些额外的html到末尾，比如分析器/调试器输出
- 发送输出到浏览器的同时也做php处理是个糟糕的想法。你是否曾见过出现在侧边栏或是屏幕中间的盒子里的Fatal error。你知道为什么会这样吗？因为处理和输出混在一起了。

### 8.输出非html内容时通过header发送正确地MIME type

我们输出一些xml试试。

{% highlight php startinline linenos  %}
$xml = '<?xml version="1.0" encoding="utf-8" standalone="yes"?>';
$xml = "<response>
	<code>0</code>
	</response>";

//Send xml data
echo $xml;
{% endhighlight %}	

完好运行。但是这需要一些改进。

{% highlight php startinline linenos  %}
$xml = '<?xml version="1.0" encoding="utf-8" standalone="yes"?>';
$xml = "<response>
	<code>0</code>
	</response>";

//Send xml data
header("content-type: text/xml");
echo $xml;
{% endhighlight %}	

注意看header那一行。那一行告诉浏览器这是xml内容。所以浏览器能正确处理它。许多js库也依赖头部信息。

对js，css，jps，png是类似的：

Javascript

{% highlight php startinline linenos  %}
header("content-type:application/x-javascript");
echo "var a = 10";
{% endhighlight %}	

CSS

{% highlight php startinline linenos  %}
header("content-type: text/css");
echo "#div id {background:#000; }";
{% endhighlight %}	

### 9.为mysql链接设置正确地编码

曾近遇上过这样一个问题，mysql表里正确地存储着unicode/utf-8编码的数据，phpmyadmin也显示正确，但是当你获取他们并把他们输出到你的页面上，他们并没有正确显示。秘密在于mysql的连接字符编码。

{% highlight php startinline linenos  %}
$host = 'localhost';
$username = 'root';
$password = 'super_secret';

//尝试连接到数据库
$c = mysqli_connect($host , $username , $password);

//检查连接有效性
if (!$c){
	die("Could not connect to the database host: <br>" . mysqli_connect_error());
}

//设置连接的字符集
if (!mysqli_set_charset( $c , 'UTF8' )){
	die('mysqli_set_charset() failed');
}
{% endhighlight %}	

一旦你连接到数据库，设置连接的字符集是个好主意。当你在应用中处理多种语言时这么做是必须得。

否则会发生什么？你会在非英语文本看到许多方块和????????。

### 10.用正确的字符集选项使用htmlentities

在php 5.4之前，默认的字符编码用的是ISO-8859-1，它不能显示À â 这样的字符。

{% highlight php startinline linenos  %}
$value = htmlentities($this->value , ENT_QUOTES , 'UTF-8');
{% endhighlight %}	

php 5.4之后的版本默认编码改为utf-8，这解决了许多问题，但清楚这一点仍然是推荐的如果你的应用是多语言的。

原文链接：
[40+ Useful Php tips for beginners – Part 1](http://www.binarytides.com/35-techniques-to-enhance-your-php-code/)
