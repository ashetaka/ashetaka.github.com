---
layout: post
title: "PHP40个有用建议-3"
description: ""
category: "译"
tags: [php]
---
{% include JB/setup %}

### 26.避免直接使用SQL查询语句，而将其抽象化

{% highlight php startinline linenos  %}
$query = "INSERT INTO users(name , email , address , phone) VALUES('$name' , '$email' , '$address' , '$phone')";
$db->query($query); //调用的是mysqli_query()
{% endhighlight %} 

以上是写sql查询语句并与数据库交互做INSERT，UPDATE,DELETE等操作的最简单的实现方式。但是它有一些缺陷，比如：

- 所有的值每次都得手动转义
- 每次手动校验sql语法
- 错误的语句可能在很长一段时间内无法发现(除非每次写完就检查)
- 像那样的长语句难以维护

解决方案：活动记录(ActiveRecord)

它包含编写将sql语句抽象化的简化函数，由此来避免直接编写sql语句。

活动记录实现的插入函数的简单例子如下：

{% highlight php startinline linenos  %}
function insert_record($table_name , $data){
	foreach ($data as $key => $value){
		//mysqli_real_escape_string
		$data[$key] = $db->mres($value);
	}

	$fields = implode(',' , array_keys($data));
	$values = "'" . implode("','" , array_values($data)) . "'";

	//Final query
	$query = "INSERT INTO {$table_name}($field) VALUES($values)";

	return $db->query($query);
}

//data to be inserted in database
$data = array('name' => $name , 'email' => $email  , 'address' => $address , 'phone' => $phone);
//perform the INSERT query
insert_record('users' , $data);
{% endhighlight %} 

上面的例子演示了如何插入数据到数据库而没有实际编写INSERT语句。insert_record函数能很好地处理转义。一个大优点是由于数据是用php数组准备好的，任何语法错误都会被立即捕捉(当然是通过php解释器捕捉的)。

这个函数可以是数据库类的一部分，可以被这样调用，$db->insert_record()。相似的函数可以也用来实现update，select，delete等。这应该是个好的实践。

### 27.将数据库结果缓存到静态文件

通过从数据库获取内容然后展现到页面上的系统，如cms等，是可以被缓存的。这意味着一旦内容生成，就复制一份到文件里。下一次同一个页面再次被请求时，直接从缓存目录获取它，而不再查询数据库。

好处是：

- 减少生成页面时php的处理过程，因此有了更快的执行
- 更少的数据库查询意味着更低的数据库负载

### 28.保存会话到数据库里

基于文件的会话有许多限制之处。用这种会话的程序不能扩展到多服务器的情况，因为文件只被存储在单台服务器上。但是数据库被从不同的服务器访问，因此这个问题就自然而然解决了。另外，在共享主机上，会话文件被存在临时目录里，对被别的账户为可读。这里有安全问题。

存储会话到数据库让别的事情也更容易了：

- 限制同个用户名的并发登录。同样的用户名不能同时从两个不同的地方登录
- 更准确地检查用户的在线状态

### 29.避免使用全局变量

- 使用define()函数和常量
- 用函数来获取值
- 用类，然后通过$this来获取

### 30.在头部标签内使用base url

简单的例子：

{% highlight html startinline linenos  %}
<head>
	<base href="http://www.domain.com/store/">
</head>
<body>
	<img src="happy.jpg"/>
</body>
</html>
{% endhighlight %} 

在html内base标签就好像是所有相对路径的'ROOT'路径一样。当静态内容文件在不同的目录和子目录时这十分有用。

来看个例子。

www.domain.com/store/home.php

www.domain.com/store/products/ipad.php

在home.php里代码可以这样写：

{% highlight html startinline linenos  %}
<a href="home.php">Home</a>
<a href="products/ipad.php">Ipad</a>
{% endhighlight %} 

但是在ipad.php中链接必须是这样的：

{% highlight html startinline linenos  %}
<a href="../home.php">Home</a>
<a href="ipad.php">Ipad</a>
{% endhighlight %} 

这是由于目录不同造成的。对于这段html导航代码的多个版本不得不做更多维护。因此简单地解决方案就是base标签。

{% highlight html startinline linenos  %}
<head>
	<base href="http://www.domain.com/store/">
</head>
<body>
	<a href="home.php">Home</a>
	<a href="products/ipad.php">Ipad</a>
</body>
{% endhighlight %} 

现在这段代码不管是对home目录，还是product目录都能正常工作了。base中的href值是用来组成home.php和product/ipad.php的完整url用的。

### 31.管理错误报告

error_reporting函数是用来设置错误报告的必要级别的。在开发环境上notices和strict级别的信息可以被这样禁用。

{% highlight php startinline linenos  %}
ini_set('display_errors' , 1);
error_reporting(~E_NOTICE & ~E_STRICT);
{% endhighlight %} 

在生产环境里，display应该被禁用。

{% highlight php startinline linenos  %}
ini_set('display_errors' , 0);
error_reporting(~E_WARNING & ~E_NOTICE & ~E_STRICT);
{% endhighlight %} 

值得注意的时error_reporting在生产环境上永远不应该设为0.至少E_FATALS必须被显示。直接用display_errors来关掉错误显示。加入error_reporting被设为0，错误不会被发现而一直隐藏在深处。

在错误显示关闭以后，所有错误应该以日志的形式记录到文件里以便分析。可以用ini_set来实现这一点。

{% highlight php startinline linenos  %}
ini_set('log_errors' , '1');
ini_set('error_log' , '/path/to/errors.txt');

ini_set('display_errors' , 0);
error_reporting(~E_WARNING & ~E_NOTICE & ~E_STRICT);
{% endhighlight %} 

注意一下几点：

1. 路径'/path/to/errors.txt'对web服务器应该是可写的，为便于其记录日志到那里
2. 指定单个的错误文件，否则所有的日志都会放在apache或是web服务器的错误日志里，导致其和别的apache错误混在一起
3. 另外由于是在当前应用中设定的，所以错误日志应该仅包含当前应用的错误(可能在web服务器上还有别的应用在跑)
4. 日志路径也可以是当前应用所在目录内的某个位置，以便不用查找像/var/log这样的目录
5. 不要把error_reporting设为0.否则它再也不会记录任何东西了

另外set_error_handler可以被用来设置一个个性化的用户写入函数来作为错误处理器。那种函数，比如说可以用来记录所有错误到一个文件里。

在开发环境的php.ini里设置'display_errors=On'。

开发环境里正确开启php.ini中的display_errors是很重要的(而不依赖ini_set)。

这是因为任何编译期间发生的致命错误都会允许执行ini_set，因此没有错误会被显示，只会有个空白网页。

相似的情况是当设置是在php.ini中完成时，在含致命错误的脚本中将无法正确关闭显示。

在生产环境的php.ini里设置'display_errors=Off'。

不要用ini_set('display_errors' , 0)；原因很简单，因为当任何编译期间发生的错误会导致它不会被执行，所以错误最后还是会显示。

### 32.熟悉平台架构

整型数据的长度在32位机器和64位机器上时不同的。因此strtotime之类函数会给出不同的结果。

在64位机器上你会看到这样额输出。

{% highlight php startinline linenos  %}
$ php -a
Interactive shell

php > echo strtotime("0000-00-00 00:00:00");
-62170005200
php > echo strtotime('1000-01-30');
-30607739600
php > echo strtotime('2100-01-30');
4104930600
{% endhighlight %} 

但是在32位机上以上所有例子都会返回bool(false)。点击[这里](http://www.binarytides.com/blog/php-strtotime-in-64-bit-environment/)查看更多。

如将一个整型数据左移32位以上会发生什么？这个结果在不同的机器上将会是不同的。

### 33. 不要太依赖set_time_limit

假如你正像下面这样来限制脚本的最大运行时间：

{% highlight php startinline linenos  %}
set_time_limit(30);

//其余代码
{% endhighlight %} 

这可能不总是能起作用。脚本内任何借助系统调用或是系统函数的执行，比如说socket操作或是数据库操作等，都不在set_time_limit的控制下。

因此如果数据库操作花费了许多时间，或者一直处于挂起状态，那么脚本将不会停止。不要为这点惊讶。在处理运行时间上用更好得策略。

### 34. 为执行shell命令创建更方便的函数

system,exec,passthru,shell_exec是4个用来执行系统命令的函数。每一个在表现上都有轻微的不同。但是问题在于当你工作在共享主机环境下，某些函数可能会有选择地被禁用了。许多新手程序员会倾向于先找出可用的那个函数然后使用它。

一种更好的解决方案是：

{% highlight php startinline linenos  %}
/**
	Method to execute a command in the terminal
	Uses:

	1.system
	2.passthru
	3.exec
	4.shell_exec

*/
function terminal($command){
	//system
	if (function_exists('system')){
		ob_start();
		system($command , $return_$var);
		$output = ob_get_contents();
		ob_end_clean();
	}
	//passthru
	else if (function_exists('passthru')){
		ob_start();
		passthru($command , $return_var);
		$output = ob_get_contents();
		ob_end_clean();
	}
	//exec
	else if (function_exists('exec')){
		exec($command , $output , $return_var);
		$output = implode("n" , $output);
	}
	//shell_exec
	else if (function_exists('shell_exec')){
		$output = shell_exec($command);
	}
	else{
		$output = 'Command execution not possible on this system';
		$return_var = 1;
	}

	return array('output' => $output , 'status' => $return_var);
}

terminal('ls');
{% endhighlight %} 

上面的函数会选择可用的函数来执行shell命令，保证你的代码效果始终如一。

### 35. 本地化你的应用

[参考这篇](http://www.binarytides.com/php-tips-localise-application/)。用逗号和小数格式化日期和数字。根据用户所在的时区来显示时间。

### 36. 如果需要的话使用xdebug这样的分析器

分析器是用来生成可展示代码不同地方执行所花时间报告的。当编写大型程序时，其中可能会有很多的库和资源内容来完成一个特定的任务，速度可能是优化的重点方向。

用分析器来检查你的代码在速度方面的表现。具体内容查看xdebug和webgrind的行管内容。

### 37.利用大量外部库

一个应用经常可能需要一些超过基本php能编写的内容。比如生成pdf文件，处理图像，发送邮件，生成图像和文档等。许多库能更快更容易地完成这些事情。

一些流行的库如下：

- mPDF - 生成pdf文件，通过把html漂亮地转化为pdf
- PHPExcel - 读写Excel文件
- PhpMailer - 轻松地发送带附件的html邮件
- pChart - 用php生成图像

### 38. 利用phpbench来做毫秒级优化

如果你实在需要在毫秒级别上实现优化，那么就试试[phpbench](http://www.phpbench.com/)吧。它拥有一些针对不同的语法差异导致的显著差异的基准。

### 39. 使用MVC框架

是时候开始使用一个MVC(Model view controller)框架了，比如codeigniter。MVC不会让你的代码马上变得面向对象化。他们做得第一件事是将php代码和html代码分开。

- 清晰将php代码和html代码分开。有利于团队合作，当设计师和程序员一起工作时候
- 函数以及被组织在类内的泛函性使得维护更加容易
- 内建的库满足各种不同需求，比如邮件、字符串处理、图像处理、文件上传等。
- 大型项目所必须的
- 大量秘诀，技术和技巧已经在框架内应用了

### 40. 查看php官网文档的评论

php官网的文档有针对每个函数、类和他们方法的条目。所有这些独立的页面在底部都有大量用户的评论，包含了社区大量有价值的信息。

他们包含用户反馈，专家建议和有用的代码片段。因此多看看他们吧。

### 41. 到irc频道去提问

irc频道#php是最好的提问php相关问题的地方。尽管有许多博客、论坛，而且每天都在增多，但是当一个特定问题出现时，解决方案可能不是直接现成的。irc就是提问的地方。而且它是完全免费的！！

### 42. 阅读开源代码

阅读别的开源程序通常是提高自身技能的好方法，如果你还没有的话。可以学习的有技术，代码风格，注释风格，文件组织和命名等。

第一个我读过的开源项目是codeigniter框架。它利于开发者使用，查看内部也同样容易。以下是一些别的项目：

1. [Codeigniter](https://github.com/EllisLab/CodeIgniter)
2. [WordPress](https://github.com/WordPress/WordPress)
3. [Joomla CMS](https://github.com/joomla/joomla-cms)

### 43. 在linux下开发

如果你正在windows下开发，你应该试试linux。我最喜欢的是ubuntu。尽管这只是个选择，不过我仍然强烈地感觉对于开发来说linux是更好得环境。

php应用大部分在linux(LAMP)环境发布。因此，在相似的环境下开发有助于更快地生产健壮的应用。

ubuntu下的新立得包管理器很容易安装大部分开发工具。顶多他们需要一点配置来安装和运行。最好的事情是，他们都是免费的。

### 可用资源：

http://www.phptherightway.com/


原文链接：
[40+ Useful Php tips for beginners – Part 3](http://www.binarytides.com/40-techniques-to-enhance-your-php-code-part-3/)
