我有充分理由相信所有的开发人员都应该学习 Haskell。我并不是说每个人都需要成为超级 Haskell武士，但是他们应该至少了解 Haskell 能提供什么。学习 Haskell 可以开阔你的想法。

主流语言共享了同样的基础：

- 变量
- 循环
- 指针
- 数据结构，对象和类(对大多数语言来说)

Haskell 则十分不同。它用了许多我以前从未听过的概念，其中的许多概念将会帮助你成为更好的程序员。

但是学习 Haskell 是困难的。至少对我来说是这样。这篇文章中我会提供我学习时不明白的内容。

这篇文章当然会很难追随。这是有意为之的。学习 Haskell 没有捷径，会是困难而且充满挑战的。但是我相信这是一件很好的事情。因为它的困难使得 Haskell 如此有趣。

学习 Haskell 的常规方法是读两本书。第一本是"Learn You a Haskell",接着是"Real World Haskell"。我也相信这是正确的方式。但是为了学习 Haskell 的所有东西，你就不得不仔细地阅读它们。

相反，这篇文章是对 Haskell 所有主要方面的一个简介而又密集的概述。我还增加了一些我学习 Haskell 时不懂的内容。

本文包含5个部分：

- 简介：一个短小的例子来展示 Haskell 的友好。
- Haskell 本质：Haskell 语法和一些基本概念。
- 噩梦：

		- 函数式风格；一个循序渐进的例子，从命令式风格到函数式风格
		- 类型；类型以及标准二叉树例子
		- 无限结构；操作无限二叉树
- 地狱：

		- 处理 IO；非常小的一个例子
		- IO技巧解释；背后的我理解不够的细节
		- Monads；不可思议到难以概括的东西
- 附录：
		- 更多关于无限树的东西；一个关于面向数学的无限树讨论
		
		
注意：每当你看到分隔符以及一个以"lhs"结尾的文件名，你可以惦记文件名来下载这个文件。如果你以"文件名.lhs"的形式保存，你可以用一下语句运行它
		runhaskell filename.lhs
		
有一些可能无法使用，但大部分可以正常使用。你将会在下面看到一个链接。

# 1. 简介
## 1.1 安装
here is a pic

[Haskell Platform](http://www.haskell.org/platform)是安装 Haskell的标准方式。

工具：
- ghc:编译器，类似于 gcc 之于 C
- ghci：**交互式 Haskell解释器(REPL)**
- runhaskell:无需编译就能执行一个程序。方便但是相比编译过的程序要慢得多。

## 1.2 别害怕
here is a pic

许多关于 Haskell 的书籍或者文章会以介绍深奥的公式(快读排序，斐波那契等等)来开篇。我会做相反的事情。首先我不会给你任何 Haskell 的超级力量。我会以 Haskell 和其它编程语言的相似点开始。**让我们跳到强制的"Hello World"。**

	main = putStrLn "Hello World!"
		
要运行这段代码，你可以保存到"hello.hs"里并且输入一下命令

	➜  haskell  >runhaskell ./hello.hs
	Hello World!
		
你也可以下载**能读能写的 Haskell 源码**。你可以在简介标题的正上方看到一个链接。下载这个叫00_hello_world.lhs 的文件，并且执行下面的命令：

	➜  haskell  >runhaskell 00_hello_world.lhs
	Hello World!
		
接下来是一个询问你姓名然后回复"hello"加你输入的名字的程序：

	main = do
        print "What is your name?"
        name <- getLine
        print ("Hello " ++ name ++ "!")
        
首先，让我们将这段代码与几种命令式语言中的相似程序比较一下：

	# Python
	print "What is your name?"
	name = raw_input()
	print "Hello %s!" % name	
<br>

	# Ruby
	puts "What is your name?"
	name = gets.chomp
	puts "Hello #{name}!"
<br>
	
	// In C
	#include <stdio.h>
	int main (int argc, char **argv) {
		char name[666]; // <- An Evil Number!
		// What if my name is more than 665 character long?
	   printf("What is your name?\n"); 
	   scanf("%s", name);
	   printf("Hello %s!\n", name);
	   return 0;
	}
	
结构是相同的，但是有一些语法差异。这篇教程的主要部分将会专门解释为什么。

在 Haskell 里有一个 main 函数，并且每个对象都有一种类型。main 的类型是 IO()。这意味着 main会有副作用。

只要记得 Haskell 可以更主流的命令式语言很像。

### Haskell 的基本概念
here is a pic

在继续教程之前你需要知道一些 Haskell的基本属性。

*函数式*
Haskell 是函数式语言。如果你有命令式语言的基础，你将需要学习很多新知识。希望这些新概念能帮助你编程，即便是在命令式语言中。

*智能静态编写*
与你再 C,C++或 JAVA 中的方式不同，这里的输入系统会帮助你。

*纯粹*
一般来说你的函数在外面的环境不用修改该任何东西。这意味着他们不需要修改该变量的值，也不能得到用户的输入，不能在屏幕上输出，不能**发射导弹**。另一方面，并行是很容易实现的。**Haskell 使得哪里发生影响以及哪里的代码是纯净的变得清晰。**推断你的程序也更加容易。大多数 bug 将会在你程序的纯净部分被阻止。

此外，纯净的函数遵循了Haskell 中的一个基本定律：
> 以相同的参数应用一个函数总是会得到相同的值

*惰性*
默认懒惰是一种不常见的语言设计。默认情况下，Haskell 只有在需要的时候才会计算一些东西的值。举例来说，这样的结果就是，它能提供一种简介的方式来操控**无限结构**。
最后，需要提醒的是你应该如何阅读 Haskell 代码。对我来说，这就像看科学论文。一些部分是清晰的，但是当你看到了一个公式，只管集中注意力并且慢慢的看。在学习 Haskell 的过程中，即便你不了解语法细节也不是很重要。如果你遇上了">>=","<$>","<-"或是别的奇怪的符号，只要忽略他们然后跟着代码流就是。

### 1.3.1 函数声明
你可能习惯于这样声明函数：
在 C 里：
		
	int f(int x,int y) {
		return x*x+y*y;
	}
在 JAvaScript 里：

	function f(x,y) {
		return x*x+y*y;
	}
在 Python 里：

	def f(x,y):
		return x*x+y*y
在 Ruby 里：

	def f(x,y)
		x*x+y*y
	end
在 Scheme 里

	(define (f x y)
		(+ (* x x) (* y y)))
最后，Haskell 的方式是：

	f x y = x*x + y*y
相当简洁。没有括号，没有 def。

别忘了，Haskell 大量地使用函数和**类型**。因此定义他们是很容易的。语法是特别为这些对象深思熟虑的。
### 1.3.2 一个**类型**例子
虽然这不是强制性的，但函数的类型信息通常是明确的。之所以不是强制性的，是因为编译器足够聪明来为你指定类型。这是一个好主意，因为它表明了意图和**理解**。

让我们来小试一下。
	
	-- We declare the type using ::
	f :: Int -> Int -> Int
	f x y = x*x + y*y

	main = print (f 2 3)
<br>
	
	➜  haskell  >runhaskell very_basic.hs
	13	
现在尝试下面的程序

	➜  haskell  >more very_basic1.hs
	f :: Int -> Int -> Int
	f x y = x*x + y*y

	main = print (f 2.3 4.2)
你应该会看到以下错误

	very_basic1.hs:4:17:
    	No instance for (Fractional Int) arising from the literal `2.3'
    	Possible fix: add an instance declaration for (Fractional Int)
    	In the first argument of `f', namely `2.3'
    	In the first argument of `print', namely `(f 2.3 4.2)'
    	In the expression: print (f 2.3 4.2)
上面的问题在于：4.2不是 int 型。
解决方案是现在不要为 f 申明类型，让 Haskell 为我们推断最可能的类型。

	f x y = x*x + y*y

	main = print (f 2.3 4.2)
它成功了！幸运的是，我们不需要为一个新函数声明每一种单一类型。比如说，在 C 里，你需要定义一个针对 int 型的函数，以及 floart 性，long 型，double 型等等。

但是，我们应该声明哪个类型。为了知道 Haskell 给我们发现的类型，只需要启动 ghci：

	➜  haskell  >ghci
	GHCi, version 7.6.3: http://www.haskell.orgghc/  :? for help
	Loading package ghc-prim ... linking ... done.
	Loading package integer-gmp ... linking ... done.
	Loading package base ... linking ... done.
	Prelude> let f x y = x*x + y*y
	Prelude> :type f
	f :: Num a => a -> a -> a
	
呃？这是什么奇怪的类型？
首先让我们把注意力放到右边的部分`a -> a ->a`。为了理解这一点，先看看下面渐进例子的表格：

<table>
<tr>
<td>The written type</td>
<td>Its meaning</td>
</tr>
</table>


在类型`a -> a ->a`中，字母 a 是个类型变量。它的意思是 f 是一个拥有两个参数且参数与返回值都有相同类型的函数。类型变量 a 可以采用许多不同的类型值。比如说 Int，或者是 Integer，或者是 Float...

因此与 C 中强制指定类型，并且需要为函数声明类型(如 int、long、float、double 等等)不同，我们只需要像动态类型语言一样声明函数即可。

有时候这被叫作参数动态化。**这也能被叫作鱼与熊掌可以兼得。**

一般来说，`a` 可以是任何类型，比如 `string` 或者 `Int`，但也可以是更复杂的类型，像 `Tree` 这样的，或是别的函数等等。但是这里我们的类型有个前缀`Num a =>`

Num是个**类型类**。类型类可以理解成一些类型的集合。Num 只包含一些行为像数字的类型。更准确地说，Num 是个包含一些类型的类，这些类型会执行一些特定的函数，尤其是`(+)`和`(*)`。

类型类是很强大的语言构想。我们可以用它做一些不可想象的强大的东西。关于这一点，后面的文章会介绍的更多。

最后，`Num a => a-> a ->a`的意思是：

让 `a` 是属于 Num 类型类的一种类型。这是一个由类型 `a` 到类型`a -> a`的函数。

是的，这十分奇怪。事实上，Haskell 里没有函数会真的拥有两个参数。相反，所有函数都只有一个参数。**但是我们注意到采用两个参数和采用一个参数然后返回一个将第二个参数作为参数的函数是等价的**。

更准确的说，`f 3 4`等价于`(f 3) 4`。注意`f 3`是一个函数：
	
	f :: Num a => a -> a -> a
	
	g :: Num a => a -> a
	g = f 3
	
	g y ⇔ 3*3 + y*y
	
函数存在另一个符号。符号 lambda 允许我们不指定名称就创建函数。这种函数叫作匿名函数。我们也可以这样写：

	g = \y -> 3*3 +y*y
	
使用`\`是因为它看起来像`λ`并且在 ASCII 码中。

如果你不习惯函数式编程，**你应该开始头大了**。现在是时候做一个真正的应用了。

但是在那之前，我们应该先确保类型系统如我们希望的那样运作：

	f :: Num a => a -> a -> a
	f x y =x*x +y*y
	
	main = print (f 3 2.4)
	
这可以运行，因为`3`对于零数，不管是 Float 还是 Integer 都是合法的表示方法。由于`2.4`是个分数，`3`会被转化成分数。

如果我们强制这个函数在不同类型下运行，结果会失败：

	f :: Num a => a -> a -> a
	f x y =x*x +y*y
	
	x :: Int
	x = 3
	y :: Float
	y = 2.4
	main = print (f x y) -- 不会正常运行，因为 x 和 y 的类型不一致
	
编译器会报错。这两个参数必须是相同类型。

如果你认为这是个坏想法，并且编译器应该为你把一种类型转化成另一种，你真的应该看看这个极好(并且有趣)的视频:[WAT](https://www.destroyallsoftware.com/talks/wat)

# 2.Haskell 本质
here is a pic

我建议你应该浏览这个部分。把它作为一个参考。Haskell有许多特色。许多信息在这都漏掉了。如果有的符号让你奇怪就回到这来。

我用符号`⇔`来表示两个表达式是等价的。这是个元符号，`⇔`在 Haskell 中并不存在。我也会用`⇒`来表示表达式的返回值是什么。

## 2.1 符号
**算术**

	3 + 2 * 6 / 3 ⇔ 3 + ((2*6)/3)
	
**逻辑**

	True || False ⇒ True
	True && False ⇒ False
	True == False ⇒ False
	True /= False ⇒ True 符号(/=)是不等于的意思
	
**幂**

	x^n 	n 是整数
	x**y y 可以是任何数字(比如说 Float)
	
`Integer`除非超出你机器的容量，否则没限制：

	4^103
	102844034832575377634685573909834406561420991602098741459288064
	
是的。就连有理数也一样。但是你得先导入`Data.Ratio`模块：

	Prelude> :m Data.Ratio
	Prelude Data.Ratio> (11%15)*(5%3)
	11 % 9	
	
**列表**

	[]                      ⇔ 空的列表
	[1,2,3]                 ⇔ 整数列表
	["foo","bar","baz"]     ⇔ 字符串列表
	1:[2,3]                 ⇔ [1,2,3], (:) 用于追加一个函数
	1:2:[]                  ⇔ [1,2]
	[1,2] ++ [3,4]          ⇔ [1,2,3,4], (++) 用于连接
	[1,2,3] ++ ["foo"]      ⇔ ERROR 字符串 ≠ 整型
	[1..4]                  ⇔ [1,2,3,4]
	[1,3..10]               ⇔ [1,3,5,7,9]
	[2,3,5,7,11..100]       ⇔ ERROR! 我没那么聪明
	[10,9..1]               ⇔ [10,9,8,7,6,5,4,3,2,1]

**字符串**

Haskell 里字符串是一个所有元素都是`Char`型的列表。

	'a' :: Char
	"a" :: [Char]
	""  ⇔ []
	"ab" ⇔ ['a','b'] ⇔  'a':"b" ⇔ 'a':['b'] ⇔ 'a':'b':[]
	"abc" ⇔ "ab"++"c"
	
	备注：真实代码中你不应该使用字符列表来表示文本。大多数时候你应该用`Data.Text`代替。如果你想表示一个 ASCII 字符流，你应该用`Data.ByteString`。
	
**元祖**

一对数据的类型是`(a,b)`。元祖中的元素可以是不同的类型。

	-- 以下元祖都是合法的
	(2,"foo")
	(3,'a',[2,3])
	((2,"a"),"c",3)

	fst (x,y)       ⇒  x
	snd (x,y)       ⇒  y

	fst (x,y,z)     ⇒  ERROR: fst :: (a,b) -> a
	snd (x,y,z)     ⇒  ERROR: snd :: (a,b) -> b

**括号处理**
为了移除括号，你可以用两个函数：`($)`和`(.)`。

	-- 一般情况:
	f g h x         ⇔  (((f g) h) x)

	-- $会将从$开始一直到表达式结尾的部分加上括号
	f g $ h x       ⇔  f g (h x) ⇔ (f g) (h x)
	f $ g h x       ⇔  f (g h x) ⇔ f ((g h) x)
	f $ g $ h x     ⇔  f (g (h x))

	-- (.) 复合函数
	(f . g) x       ⇔  f (g x)
	(f . g . h) x   ⇔  f (g (h x))

## 2.2 一些对函数有用的符号
只有一件事要提醒：

	x :: Int            ⇔ x 是 Int 型
	x :: a              ⇔ x 可以是任何类型
	x :: Num a => a     ⇔ x 可以是任何属于 Num 类的类型 
	f :: a -> b         ⇔ f 是**由 a 到 b 的函数**
	f :: a -> b -> c    ⇔ f 是由 a 到 (b→c)的函数
	f :: (a -> b) -> c  ⇔ f 是由 (a→b) 到 c的函数
	
请记得在声明前定义一个函数的类型不是强制的。Haskell能为你推断大部分类型。但这样做是个好做法。

*中缀表示法*

	square :: Num a => a -> a
	square x = x^2
	
注意`^`用的是中缀表示法。对于每一个中缀运算符都有相关的前缀表示法。你只需要把它放在括号里。

	square' x = (^) x 2
	square'' x = (^2) x
我们可以在两边都去掉`x`!这就是**η压缩**。

	square''' = (^2)
注意我们在名字用用`'`来声明函数。请看下面：

	square ⇔ square' ⇔ square'' ⇔ square'''
	
*测试*
绝对值函数的一种实现方式如下。

	absolute :: (Ord a, Num a) => a -> a
	absolute x = if x >= 0 then x else -x
	
请注意，Haskell 符号`if..then..else..`和 C 运算符`¤?¤:¤`很像。千万别忘了`else`。

另一个等价的版本：

	absolute' x
	    | x >= 0 = x
	    | otherwise = -x

	符号警示：缩进在 Haskell 中十分重要。和 Python 一样，会缩进会毁了你的代码。
	
# 3. 噩梦
现在开始增加难度。
##3.1 函数式风格
here is a pic

在这一部分，我会给出一个简单例子来阐述 Haskell 提供的让人印象深刻的重构能力。我们会选择一个问题并且以标准的命名式方法来解决它。然后我会改进代码。最终结果会更简介、更容易去修改。

让我们解决下面这个问题：

	给定一个整数列表，然会列表中偶数数字的和。
	例如：`[1,2,3,4,5] ⇒ 2 + 4 ⇒ 6`
	
为了说明函数式方法和命令式方法的不同，我会以提供命名式解决方法开始(使用 JavaScript)：

	function evenSum(list) {
		var result = 0;
		
		for (var i=0; i< list.length ; i++) {
			if (list[i] % 2 ==0) {
	            result += list[i];
	     }
	   }
	   return result;
	}
在 Haskell 中，恰恰相反，我们不需要变量或是 for 循环。一种不用循环得到相同结果的方案是使用递归。

	备注：递归一般被认为是命令式语言中很慢的方式。但这一般在函数式编程中不会是问题。大多数实践中 Haskell 能高效地处理递归函数。
	
接下来是`C`语言版的递归函数。请注意为了简单起见，我假定这个整型列表以第一个`0`结尾。

	int evenSum(int *list) {
	    return accumSum(0,list);
	}

	int accumSum(int n, int *list) {
	    int x;
	    int *xs;
	    if (*list == 0) { // if the list is empty
	        return n;
	    } else {
	        x = list[0]; // let x be the first element of the list
	        xs = list+1; // let xs be the list without x
				if ( 0 == (x%2) ) { // if x is even
		            return accumSum(n+x, xs);
	        } else {
	            return accumSum(n, xs);
	        }
	    }
	}
	
把这段代码记在心里。我们会把它翻译到 Haskell 中去。然后首先我需要介绍三个简单而实用的函数，我们一会儿会用到：

	even :: Integral a => a -> Bool
	head :: [a] -> a
	tail :: [a] -> [a]
	
`even`会盘度啊一个函数是否是偶数。

	even :: Integral a => a -> Bool
	even 3 ⇒ False
	even 2 ⇒ True
	
`head`会返回列表的第一个元素：

	head :: [a] -> a
	head [1,2,3] ⇒ 1
	head [] ⇒ ERROR
	
`tail`会返回列表的所有元素，除了第一个：

	tailf :: [a] -> [a]
	tail [1,2,3] ⇒ [2,3]
	tail [3] ⇒ []
	tail [] ⇒ ERROR
请注意对任何非空列表`l`，都有`l ⇒ (head l):(tail l)`

第一种 Haskell 解决方法。函数`evenSum`会返回列表中所有偶数数字的和：

	-- Version 1
	evenSum :: [Integer] -> Integer

	evenSum l = accumSum 0 l

	accumSum n l = if l == []
								then n
								else let x = head l 
											 xs = tail l 
										in if even x
												then accumSum (n+x) xs
												else accumSum n xs
问题1：parse error '='		
要测试一个函数可以使用`ghci`：

	Prelude> :load 11_Functions.lhs
	[1 of 1] Compiling Main             	( 11_Functions.lhs, interpreted )
	Ok, modules loaded: Main.
	*Main> evenSum [1..5]
	6
	
下面是执行过程示例 **注2**：

	*Main> evenSum [1..5]
	accumSum 0 [1,2,3,4,5]
	1 is odd
	accumSum 0 [2,3,4,5]
	2 is even
	accumSum (0+2) [3,4,5]
	3 is odd
	accumSum (0+2) [4,5]
	4 is even
	accumSum (0+2+4) [5]
	5 is odd
	accumSum (0+2+4) []
	l == []
	0+2+4
	0+6
	6
从命令式语言的角度一切看起来都是正确的。实际上，许多地方可以改进。首先，我们归纳类型。

	evenSum :: Integral a => [a] -> a
	
然后，我们使用利用`where`或`letl`的子函数。这种方式下`accumSum`函数不会污染我们模块的命名空间。

	-- Version 2
	evenSum :: Integral a => [a] -> a

	evenSum l = accumSum 0 l
	    where accumSum n l = 
            if l == []
                then n
                else let x = head l 
                         xs = tail l 
                     in if even x
                            then accumSum (n+x) xs
                            else accumSum n xs	
接下来我们将使用模式匹配。

	-- Version 3
	evenSum l = accumSum 0 l
	    where 
	        accumSum n [] = n
	        accumSum n (x:xs) = 
	             if even x
	                then accumSum (n+x) xs
	                else accumSum n xs
模式匹配是什么。使用值来代替一般的变量名称 **注3**.

与`foo l = if l == [] then <x> else <y>`不同，你只要简单地声明：

	foo [] =<x>
	foo l =<y>

但模式匹配可以做的还有更多。它也能检查一个复杂值的内部数据。我们可以将

	foo l = let x = head l
	            xs = tail l
	        in if even x
	            then foo (n+x) xs
	            else foo n xs
替换为

	foo (x:xs) = if even x
							then foo (n+x) xs
							else foo n xs
这是很有用的特色。这使得代码精炼而又易于阅读。

Haskell 里你可以通过**η压缩**来简化函数定义。比如说，要简化：

	f x = (some expression) x
你可以简化的写成

	f = some expression
我们用这种方法来去掉`l`：

	-- Version 4
	evenSum :: Integral a => [a] -> a

	evenSum = accumSum 0
	    where 
	        accumSum n [] = n
	        accumSum n (x:xs) = 
	             if even x
	                then accumSum (n+x) xs
	                else accumSum n xs
### 3.1.1 高阶函数
here is a pic

为了更好的编程我们应该使用高阶函数。这些怪物是啥？高阶函数是用函数作为参数的函数。

下面是一些例子：

	filter :: (a -> Bool) -> [a] -> [a]
	map :: (a -> b) -> [a] -> [b]
	foldl :: (a -> b -> a) -> a -> [b] -> a
	
我们来一步步展开。

	-- Version 5
	evenSum l = mysum 0 (filter even l)
		where
			mysum n [] = n
			mysum n (x:xs) = mysum (n+x) xs
其中

	filter even [1..10] ⇔ [2,4,6,8,10]
	
`filter`函数使用了类型`a -> Bool`的函数和类型`[a]`的列表。它返回只包含函数返回`true`的元素的列表。

下一步是用另一种技术来做到和循环一样的事情。我们用`foldl`函数来遍历一个列表以积攒一个值。`foldl`函数捕捉了一般的编程模式：

	myfunc list = foo initialValue list
		foo accumulated [] =accumulated
		foo tmpValue ( x:xs) = foo (bar tmpValue x) xs
可以被代替为：

	myfunc list = fold1 bar initialValue list
	
如果你实在想知道魔术是怎么发生的，以下是`foldl`的定义：

	foldl f z [] = z
	foldl f z (x:xs) = foldl f (f z x) xs
<br>

	foldl f z [x1,...xn]
	⇔  f (... (f (f z x1) x2) ...) xn
	
但 Haskell 是懒惰的，它不会计算`(f z x)`，只是简单的把它推到栈里。这就是为什么我们通常使用`foldl'`而不是`foldl`；`foldl'`是`foldl`的严谨版本。如果你不明白懒惰和严谨的意思，别担心，只要跟随代码就好像`foldl`和 `foldl'`是一样的。

现在我们新版本的`evenSum`就变成：

	-- Version 6
	-- foldl' isn't accessible by default
	-- we need to import it from the module Data.List
	import Data.List
	evenSum l = foldl' mysum 0 (filter even l)
		where mysum acc value = acc + value
我们可以直接使用 lambda 表示法来简化这段代码。这种方式下我们不用再创建临时变量名`mysum`。

	-- Version 7
	-- Generally it is considered a good practice
	-- to import only the necessary function(s)
	import Data.List (foldl')
	evenSum l = foldl' (\x y -> x+y) 0 (filter even l)
	
当然我们注意到

	(\x y -> x+y) ⇔ (+)
	
最终版本就是这样

	-- Version 8
	import Data.List (foldl')
	evenSum :: Integral a => [a] -> a
	evenSum l = foldl' (+) 0 (filter even l)
	
`foldl'`不是理解起来最简单的函数。如果你不习惯它，你应该多学一点了。

为了帮助你理解接下来要发送的，我们一步步来看看值的变化：

	  evenSum [1,2,3,4]
	⇒ foldl' (+) 0 (filter even [1,2,3,4])
	⇒ foldl' (+) 0 [2,4]
	⇒ foldl' (+) (0+2) [4] 
	⇒ foldl' (+) 2 [4]
	⇒ foldl' (+) (2+4) []
	⇒ foldl' (+) 6 []
	⇒ 6
	
另一个有用的高阶函数是`(.)`。`(.)`函数符合**数学理解**。

	(f . g . h) x ⇔  f ( g (h x))
	
我们可以用这个运算符的优势来η压缩我们的函数：

	-- Version 9
	import Data.List (foldl')
	evenSum :: Integral a => [a] -> a
	evenSum = (foldl' (+) 0) . (filter even)
而且，我们可以重命名某些部分来使得它更整洁：

	-- Version 10 
	import Data.List (foldl')
	sum' :: (Num a) => [a] -> a
	sum' = foldl' (+) 0
	evenSum :: Integral a => [a] -> a
	evenSum = sum' . (filter even)
	
现在是时候讨论我们代码改变的方向，因为我们介绍了更多关于函数式风格的内容。我们从使用高阶函数中得到了什么？

首先，你可能会认为主要的区别是简洁。但实际上，这还需要更多的思考。比如说，假如我们想轻微地修改我们的函数，来得到列表中所有偶数平凡的和。

	[1,2,3,4] ▷ [1,4,9,16] ▷ [4,16] ▷ 20
	
更新后的版本10是解毒简单的：

	squareEvenSum = sum' . (filter even) . (map (^2))
	squareEvenSum' = evenSum . (map (^2))
	squareEvenSum'' = sum' . (map (^2)) . (filter even)
	
我们只需要增加另一个"转化函数" **注4**。

	map (^2) [1,2,3,4] ⇔ [1,4,9,16]
	
`map`函数简单地对列表中所有元素应用函数。

我们不需要修改该函数定义内的任何东西。这使得代码更加模块化。但是额外的你需要更加数学化地考虑你的函数。需要的话，你也可以可交换地使用你的函数。那就是说，你可以利用你的新函数去组合，映射，打开和过滤。

修改版本1就留作读者的练习吧☺。

如果你认为我们已经到了最后，那么你错了。比如说，有一种不仅仅把这个函数用在列表上还能用在别的任何递归类型上的方法。如果你现在就想知道，我加你你看一下这篇很有趣的文章：[ Functional Programming with Bananas, Lenses, Envelopes and Barbed Wire by Meijer, Fokkinga and Paterson](http://eprints.eemcs.utwente.nl/7281/0%201/db-utwente-40501F46.pdf)

这个例子应该已经向你展示了纯函数式编程是多棒了。不幸的是，使用纯函数式编程并不适用于所有应用。或者至少这样的一种语言还没被创造。

Haskell 的强大能力之一是创建 DSLs(特定领域语言)，使得它改变编程范式很容易。

实际上，Haskell 在你想要写命名式风格编程的时候也是很棒的，当我第一次学 Haskell 的时候，我很难理解这一点。要解释函数式方法的优越性需要大量的努力。当你开始用命令式风格来使用 Haskell，就很难理解什么时候以及怎么去使用它。

但是在谈论 Haskell 的超级能力前，我们必须谈一谈Haskell 的另一个基本方面：*类型*。

## 3.2类型
here is a pic

>摘要

>- `type Name = AnotherType` 类型别名定义，编译器看待`Name`和`AnotherType`是一样的
>- `data Name = NameConstructor AnotherType` 是有不同的
>- `data`可以构造能**重复使用**的结构
>- `deriving`是个神奇的功能，可以为你创建函数

Haskell 里类型是健壮而且静态的。

为什么这很重要？它能助你极大地避免错误。Haskell 里大部分 bug 都是在程序编译的时候发现的。这主要是因为编译中的类型推断。比如说，类型推断能很容易地检测出错误的变量在什么错误的位置上。

### 3.2.1 类型推断

**对于快速的执行静态类型是基本的**。但是大多数静态类型语言不善于归纳概念。Haskell 的可取之处在于它能推断类型。

下面这个简单的例子是 Haskell 中的`square`函数：

	square x = x * x
	
这个函数可以使用任何数字类型的参数。你可以为`square`提供 `Int`、`Integer`、`Float`、`Fractional`甚至 `complex`。让我们用例子来说明：

	Prelude> let square x = x * x
	Prelude> square 2
	4
	Prelude> square 2.1
	4.41
	Prelude> -- load the Data.Complex module
	Prelude> :m Data.Complex
	Prelude Data.Complex> square (2 :+ 1)
	3.0 :+ 4.0
`x :+ y`是复数(x+ib)的意思。

现在与 C 中必需代码的数量比较一下：

	int     int_square(int x) { return x*x; }

	float   float_square(float x) {return x*x; }

	complex complex_square (complex z) {
	    complex tmp;
	    tmp.real = z.real * z.real - z.img * z.img;
	    tmp.img = 2 * z.img * z.real;
	}

	complex x,y;
	y = complex_square(x);

每一种类型都需要你去写一个新函数。解决这个问题的唯一办法是用一些元编程技巧，比如说使用预处理机。C++中有更好的方法，以下是 C++的模板：

	#include <iostream>
	#include <complex>
	using namespace std;

	template<typename T>
	T square(T x)
	{
	    return x*x;
	}

	int main() {
		// int
		int sqr_of_five = square(5);
		cout << sqr_of_five << endl;
		// double
		cout << (double)square(5.3) << endl;
		// complex
		cout << square( complex<double>(5,3) )
		<< endl;
		return 0;
	}

就这一点而言，C++比 C 要做的好得多。但是对于更加复杂的函数，语法将会更加复杂：比如这篇[文章](http://bartoszmilewski.com/2009/10/21/what-does-haskell-have-to-do-with-c/)的内容。

C++中你不需要声明一个函数是能支持多种类型的参数。Haskell 里则恰好相反。函数会跟一般情况下没什么两样。

类型推断为 Haskell 提供了动态类型语言提供的自由感。但是与动态类型语言不同的是，大多数错误都可以在运行前被发现。一般来说，在 Haskell 中：

	只要它能编译，它就能做你想要它做的。
	
### 3.2.2 类型构造
你可以构造自己的类型。首先你可以使用别名或者类型同义词。

	type Name   = String
	type Color  = String

	showInfos :: Name ->  Color -> String
	showInfos name color =  "Name: " ++ name
                        ++ ", Color: " ++ color
	name :: Name
	name = "Robin"
	color :: Color
	color = "Blue"
	main = putStrLn $ showInfos name color
	
但它的保护不足。试着交换`showInfos`两个参数的顺序，然后运行程序：

	putStrLn $ showInfos color name
	
它会编译然后运行。实际上你可以在任何位置替换使用 Name、Color、String。编译器会无差别对待它们。

另一种创建自己类型的方法是使用关键字`data`。

	data Name   = NameConstr String
	data Color  = ColorConstr String

	showInfos :: Name ->  Color -> String
	showInfos (NameConstr name) (ColorConstr color) =
      "Name: " ++ name ++ ", Color: " ++ color

	name  = NameConstr "Robin"
	color = ColorConstr "Blue"
	main = putStrLn $ showInfos name color
	
现在假如交换`showInfos`的参数，编译器会报错！因此这不会是你再产生的一个潜在错误，而且唯一的代价是更详细些。

另外也可以注意到构造器其实是函数：
	
	NameConstr  :: String -> Name
	ColorConstr :: String -> Color
`data`的语法主要如下：

	data TypeName =   ConstructorName  [types]
                | ConstructorName2 [types]
                | ...
一般的做法是为 DataTypeName 和 DataTypeConstructor 使用同一个名字。

如：

	data Complext = Num a => Complex a a
你也可以用**record syntax**：

	data DataTypeName = DataConstructor {
                      field1 :: [type of field1]
                    , field2 :: [type of field2]
                    ...
                    , fieldn :: [type of fieldn] }
**这样就给你产生了许多存储器**。此外，你也能在设置值的时候用另一种顺序。

如：

	data Complex = Num a => Complex { real :: a, img :: a}
	c = Complex 1.0 2.0
	z = Complex { real = 3, img = 4 }
	real c ⇒ 1.0
	img z ⇒ 4
	
### 3.2.3 递归类型
你已经遇到过一种递归类型：列表。你可以再次创建列表，但这次用更详细的语法：

	data List a = Empty | Cons a (List a)
	
如果你真的想让语法更简单，你可以为构造器使用中缀名。

	infixr 5 :::
	data List a = Nil | a ::: (List a)
	
`infixr`后的数字给出了优先级。

如果想新的数据结构支持输出(`Show`)、读取(`Read`)、是否相等(`Eq`)和比较函数(`Ord`)，你可以让 Haskell 为你派生合适的函数。

	infixr 5 :::
	data List a = Nil | a ::: (List a) 
	              deriving (Show,Read,Eq,Ord)
当你把`deriving (Show)`加到数据声明里的时候，Haskell 为你创建了一个`show`函数。我们很快就能看到如何使用自己的`show`函数。

	convertList [] = Nil
	convertList (x:xs) = x ::: convertList xs
	
<br>

	main = do
			print (0 ::: 1 ::: Nil)
	     print (convertList [0,1])
	     
这段代码会输出：

	0 ::: (1 ::: Nil)
	0 ::: (1 ::: Nil)
	
### 3.2.4 树
here is a pic

我们马上会看到另一个标准例子：二叉树。

	import Data.List

	data BinTree a = Empty
                 | Node a (BinTree a) (BinTree a)
                              deriving (Show)
然后创建一个把列表转成有序二叉树的函数。

	treeFromList :: (Ord a) => [a] -> BinTree a
	treeFromList [] = Empty
	treeFromList (x:xs) = Node x (treeFromList (filter (<x) xs))
                             (treeFromList (filter (>x) xs))
可以看到这个函数是多么的简洁。以文字来解释就是：

- 空列表会转换成空树
- 列表(x:xs)会转成这样的一棵树：
- 根是 x
- 左子树由列表 xs 中严格小于 x 的成员组成
- 右子树由列表 xs 中样儿大于 x 的成员组成

<br>	

	main = print $ treeFromList [7,2,4,8]
你应该得到如下的输出：

	Node 7 (Node 2 Empty (Node 4 Empty Empty)) (Node 8 Empty Empty)
这种树的表示法信息足够但不够友好。


为了好玩，我们为树编写一个更好的展示。我以普通方法简单地实现了一个很棒的函数来展示树。你完全可以跳过这个部分如果你觉得内容太难学习。

我们有一些地方需要改动。首先将`BinTree`类型声明中的`deriving (Show)`给去掉。让 BinTree 成为(`Eq`和`Ord`)的实例，这样我们就能测试相等性以及比较树，这将会是十分有用的。

	data BinTree a = Empty
                 | Node a (BinTree a) (BinTree a)
                  deriving (Eq,Ord)
                  
去掉`deriving (Show)`以后，Haskell 不会再为我们创建一个`show`方法。我们来创建自己版本的`show`。为了实现这一点，我们必须声明新造的类型 BinTree 是类型类`Show`的实例。一般语法是：

	instance Show (BinTree a) where
	   show t = ... -- You declare your function here
	   
这里是如何展示二叉树的我的版本。别担心这明显的复杂性。我做了许多改进来显示更多奇怪的对象。

	-- declare BinTree a to be an instance of Show
	instance (Show a) => Show (BinTree a) where
	  -- will start by a '<' before the root
	  -- and put a : a begining of line
	  show t = "< " ++ replace '\n' "\n: " (treeshow "" t)
	    where
	    -- treeshow pref Tree
	    --   shows a tree and starts each line with pref
	    -- We don't display the Empty tree
	    treeshow pref Empty = ""
	    -- Leaf
	    treeshow pref (Node x Empty Empty) =
	                  (pshow pref x)
	
	    -- Right branch is empty
	    treeshow pref (Node x left Empty) =
	                  (pshow pref x) ++ "\n" ++
	                  (showSon pref "`--" "   " left)
	
	    -- Left branch is empty
	    treeshow pref (Node x Empty right) =
	                  (pshow pref x) ++ "\n" ++
	                  (showSon pref "`--" "   " right)
	
	    -- Tree with left and right children non empty
	    treeshow pref (Node x left right) =
	                  (pshow pref x) ++ "\n" ++
	                  (showSon pref "|--" "|  " left) ++ "\n" ++
	                  (showSon pref "`--" "   " right)
	
	    -- shows a tree using some prefixes to make it nice
	    showSon pref before next t =
	                  pref ++ before ++ treeshow (pref ++ next) t
	
	    -- pshow replaces "\n" by "\n"++pref
	    pshow pref x = replace '\n' ("\n"++pref) (show x)
	
	    -- replaces one char by another string
	    replace c new string =
	      concatMap (change c new) string
	      where
	          change c new x
	              | x == c = new
	              | otherwise = x:[] -- "x"


`treeFromList`方法保持不变。


	treeFromList :: (Ord a) => [a] -> BinTree a
	treeFromList [] = Empty
	treeFromList (x:xs) = Node x (treeFromList (filter (<x) xs))
                             (treeFromList (filter (>x) xs))

现在我们开始：

	main = do
		putStrLn "Int binary tree:"
		print $ treeFromList [7,2,4,8,1,3,6,21,12,23]
<br>

	Int binary tree:
	< 7
	: |--2
	: |  |--1
	: |  `--4
	: |     |--3
	: |     `--6
	: `--8
	:    `--21
	:       |--12
	:       `--23
这样好多了！根节点用`<`开头的一行表示。接下来的每一行都用`:`开头。但是我们也可用另一种类型。

	putStrLn "\nString binary tree:"
	print $ treeFromList ["foo","bar","baz","gor","yog"]
	
<br>

	String binary tree:
	< "foo"
	: |--"bar"
	: |  `--"baz"
	: `--"gor"
	:    `--"yog"
	
由于我们可以测试相等性以及给树排序，我们可以**用由树组成的树**！

	putStrLn "\nBinary tree of Char binary trees:"
	print ( treeFromList
           (map treeFromList ["baz","zara","bar"]))
           
<br>

	Binary tree of Char binary trees:
	< < 'b'
	: : |--'a'
	: : `--'z'
	: |--< 'b'
	: |  : |--'a'
	: |  : `--'r'
	: `--< 'z'
	:    : `--'a'
	:    :    `--'r'
	
这就是我为何选择`:`作为树输出每行的前缀(除了根节点)。

here is a pic

	putStrLn "\nTree of Binary trees of Char binary trees:"
	print $ (treeFromList . map (treeFromList . map treeFromList))
             [ ["YO","DAWG"]
             , ["I","HEARD"]
             , ["I","HEARD"]
             , ["YOU","LIKE","TREES"] ]
             
等同于

	print ( treeFromList (
          map treeFromList
             [ map treeFromList ["YO","DAWG"]
             , map treeFromList ["I","HEARD"]
             , map treeFromList ["I","HEARD"]
             , map treeFromList ["YOU","LIKE","TREES"] ]))
             
会输出：

	Binary tree of Binary trees of Char binary trees:
	< < < 'Y'
	: : : `--'O'
	: : `--< 'D'
	: :    : |--'A'
	: :    : `--'W'
	: :    :    `--'G'
	: |--< < 'I'
	: |  : `--< 'H'
	: |  :    : |--'E'
	: |  :    : |  `--'A'
	: |  :    : |     `--'D'
	: |  :    : `--'R'
	: `--< < 'Y'
	:    : : `--'O'
	:    : :    `--'U'
	:    : `--< 'L'
	:    :    : `--'I'
	:    :    :    |--'E'
	:    :    :    `--'K'
	:    :    `--< 'T'
	:    :       : `--'R'
	:    :       :    |--'E'
	:    :       :    `--'S'new

请注意重复的树是如何不被插入的；只有一颗对应`"I","HEAD"`的树。我们实现这一点(几乎)没做什么，因为我们已经什么 Tree 是`Eq`的一个实例。

看看这个结构是多了不起：我们构造了不止包含整数、字符串、字符类型的树，也还有别的树。我们甚至能构造一颗包含树的树。

## 3.3 无限结构

here is a pic

经常有一种说法是 Haskell 是懒惰的。

实际上，如果稍微考究一点，你应该说 [Haskell 是非严格的](http://www.haskell.org/haskellwiki/Lazy_vs._non-strict)。懒惰只是非严格语言的一个常见实现。

那么"非严格"是什么意思呢？Haskell wiki 上说：

	**约简(求值的数学术语说法)由外开始**。
	
	因此假如要计算`(a+(b*c))`，你首先简化`+`，然后简化内部的`(b+c)`
	
比如说 Haskell 里你可以：

	-- numbers = [1,2,..]
	numbers :: [Integer]
	numbers = 0:map (1+) numbers

	take' n [] = []
	take' 0 l = []
	take' n (x:xs) = x:take' (n-1) xs

	main = print $ take' 10 numbers

它会停止。

如何做到的？

与不停地完全计算`numbers`不同，它只在需要的时候给元素求值。

而且要注意的是 Haskell 里有无限列表的表示法

	[1..]   ⇔ [1,2,3,4...]
	[1,3..] ⇔ [1,3,5,7,9,11...]
	
大多数函数都能正常运行。此外，内在函数`take`和我们的`take'`是等价的。

假设我们不介意有一个有序二叉树。这里是个无限二叉树：

	nullTree = Node 0 nullTree nullTree
	
这是个每个节点都等于0的完全二叉树。现在我将证明你能用下面的函数来操作这个对象：

	-- take all element of a BinTree 
	-- up to some depth
	treeTakeDepth _ Empty = Empty
	treeTakeDepth 0 _     = Empty
	treeTakeDepth n (Node x left right) = let
	          nl = treeTakeDepth (n-1) left
	          nr = treeTakeDepth (n-1) right
	          in
						Node x nl nr
让我们看看这个程序会发生什么：

	main = print $ treeTakeDepth 4 nullTree
	
代码顺利编译、运行、停止并且给出了如下的结果：

	<  0
	: |-- 0
	: |  |-- 0
	: |  |  |-- 0
	: |  |  `-- 0
	: |  `-- 0
	: |     |-- 0
	: |     `-- 0
	: `-- 0
	:    |-- 0
	:    |  |-- 0
	:    |  `-- 0
	:    `-- 0
	:       |-- 0
	:       `-- 0
	
让你神经更加热情一点，我们实现一个稍微有趣些的树：

	iTree = Node 0 (dec iTree) (inc iTree)
	        where
					dec (Node x l r) = Node (x-1) (dec l) (dec r) 
	           inc (Node x l r) = Node (x+1) (inc l) (inc r) 	
另一种创建这棵树的方法使用高阶函数。这个函数应该类似于`map`，但要能在`BinTree`上运行而不是列表。下面是一个这样的函数：

	-- apply a function to each node of Tree
	treeMap :: (a -> b) -> BinTree a -> BinTree b
	treeMap f Empty = Empty
	treeMap f (Node x left right) = Node (f x) 
                                     (treeMap f left) 
                                     (treeMap f right)

*注释*：这里我不会再讨论更多内容了。如果你对于 map 泛化到别的数据类型有兴趣，就搜索仿函数和`fmap`。

现在的定义是：

	infTreeTwo :: BinTree Int
	infTreeTwo = Node 0 (treeMap (\x -> x-1) infTreeTwo) 
                    (treeMap (\x -> x+1) infTreeTwo) 
                    
用这来看结果

	main = print $ treeTakeDepth 4 infTreeTwo
	
<br>


	<  0
	: |-- -1
	: |  |-- -2
	: |  |  |-- -3
	: |  |  `-- -1
	: |  `-- 0
	: |     |-- -1
	: |     `-- 1
	: `-- 1
	:    |-- 0
	:    |  |-- -1
	:    |  `-- 1
	:    `-- 2
	:       |-- 1
	:       `-- 3	
	
# 4.地狱

恭喜你看到了这里！现在开始真正底层的东西。

如果你和我一样，你应该掌握了函数式风格。你也对于默认情况下采用懒惰的优势也有了更多的理解。但是你不是真正知道从哪开始来做一个真正的程序。特别地：

- 怎么处理**效应**？
- 处理 IO 的时候为什么有个奇怪的像命令式的符号？

做好准备吧，答案会是很复杂的。但他们都很值得。

## 4.1 处理 IO

here is a pic

摘要

一个典型的处理 IO 的函数和命令式程序有很多地方很像：

	f :: IO a
	f = do
	  x <- action1
	  action2 x
	  y <- action3
	  action4 x y

- 给对象复制用`<-`
- 每一行的类型是`IO *`；在这个例子中
- `action1 :: IO b`
- `action2 x :: IO ()`
- `action3 :: IO c`
- `action4 x y :: IO a`
- `x :: b`,`y :: c`
- 很少有对象会使用类型`IO a`，这有助于你选择。特别是你不能直接在这使用纯函数。举例来说，要使用纯函数你可以用`action (purefunction x)`。


在这个部分，我会解释如何使用 IO，而不是它的工作原理。你将看到Haskell 是如何将代码纯正的部分和不纯的部分区分开来的。

不要停下来，因为你正试着理解语法的细节。答案会在下一部分揭晓。

要达成什么？

	让用户输入一串数字，并打印出数字的和。
<br>

	toList :: String -> [Integer]
	toList input = read ("[" ++ input ++ "]")
	
	main = do
		putStrLn "Enter a list of numbers (separated by comma):"
		input <- getLine
		print $ sum (toList input)
		
理解这个程序的行为是很简单的。我们来详细得分析一下类型。

	putStrLn :: String -> IO ()
	getLine :: IO String
	print :: Show a => a -> IO ()
	
更有趣的是，我们注意到`do`代码块下的每个表达式都有`IO a`。

	main = do
		putStrLn "Enter ... " :: IO ()
		getLine :: IO String
		print Something :: IO ()
		
我们也应注意下符号`<-`的影响。

	do
		x <- something
		
若`something :: IO a`，则`x :: a`。

使用`IO`的另一个重点是：do 代码块中的所有行必须是以下两种格式中的一种：

	action1 :: IO a
				-- in this case,generally a = ()
				
或

	value <- action2 -- where
							  -- bar z t :: IO b
							  -- value ::b	
							  
这两种形式的代码行会对应两种不同的执行顺序。这句话的意思会在下章的末尾更加清晰。

现在我们来看看这个程序会如何运行。比如说，假如用户输入了奇怪的东西时会发生什么？让我们来试试：

    % runghc 02_progressive_io_example.lhs
    Enter a list of numbers (separated by comma):
    foo
    Prelude.read: no parse
    
哎呀！出现了一个可恶的报错信息和崩溃！我们首先做的改进是简单地提示一条更友好的信息。

为了做到这点，我们必须检测出错的地方。这里有种方法：使用类型`Maybe`。这是一种 Haskell 里常见类型。

	import Data.Maybe
	
这是什么？`Maybe`是一种能获取一个参数的类型。它的定义是：

	data Maybe a = Nothing | Just a

这对创建或计算一个值时判断是否有错误是很好的方法。函数`maybeRead`就是个好例子。这个是个类似于`read` **注5** 的函数，但如果出了错它的返回值会是`Nothing`。如果值是正确的，它会返回`Just <the value>`。不用花太多经理在理解这个函数上。我用了个比`read`低级些的函数；`reads`。

	maybeRead :: Read a => String -> Maybe a
	maybeRead s = case reads s of
								[(x,"")]	-> Just x
								-				-> Nothing
								
现在为了更可读一些，我们定义一个这样的函数：如果字符串格式错误，它会返回`Nothing`。否则，比如说对于"1,2,3"，它会返回 `Just [1,2,3]`。

	getListFromString :: String -> Maybe [Integer]
	getListFromString str = maybeRead $ "[" ++ str ++ "]"	
我们在主函数中简单得测试下值。

	main :: IO ()
	main = do
		putStrLn "Enter a list of numbers (separated by comma):"
		input <- getLine
		let maybeList = getListFromString input in
				case maybeList of
						Just l  -> print (sum l)
						Nothing -> error "Bad format. Good Bye."
						
对于错误的情况，我们现实了一个漂亮的错误信息。

注意到 main 的 do 代码块中每个表达式的类型保持了`IO a`这样的形式。唯一奇怪的构造是`error`。我马上就要说`error msg`使用了它需要的类型(这里是 `IO ()`)。

很重要的一点是目前所有已经定义函数的类型。只有一个函数在包含了`IO`类型：`main`。这意味着 main 函数是不纯净的。但是 main 使用了纯净的`getListFromString`。因此只要看一下被定义的类型就能知道哪些函数是纯净的，哪些不是。

为什么纯净性这么重要。在众多优势中，这三点已足以说明：

- 理解纯净的代码比脏代码更容易
- 纯净性避免了那些来自边界效应的难以重现的 bug
- 你可以毫无风险地以任何顺序并行计算纯净的函数

这就是为什么你应该尽可能多地把代码放在纯函数里。

咱们接下来要做个循环，不停地提示用户直到她输入了合法的内容。

我们保留第一部分：

	import Data.Maybe

	maybeRead :: Read a => String -> Maybe a
	maybeRead s = case reads s of
                  		[(x,"")]    -> Just x
                  		_           -> Nothing
	getListFromString :: String -> Maybe [Integer]
	getListFromString str = maybeRead $ "[" ++ str ++ "]"

现在再来创建一个函数，让用户输入一串数字，直到输入对了为止。

	askUser :: IO [Integer]
	askUser = do
		putStrLn "Enter a list of numbers (separated by comma):"
		input <- getLine
		let maybeList = getListFromString input in
			case maybeList of
				Just l  -> return l
				Nothing -> askUser
				
该函数的类型是`IO [Integer]`。这样的类型意味着我们通过一些 IO 行为重新获取了类型`[Integer]`的值。有的人也许会在打招呼的时候这么解释：

	这是`IO`内的一个`[Integer]`
	
如果你想理解背后的细节，你需要阅读下一章。但是说真的，如果你只想使用 IO，只要多一点时间，并且记得考虑下类型。

最后我们主函数就简单多了：

	main :: IO ()
	main = do
		list <- askUser
		print $ sum list
		
我们完成了对`IO`的介绍。虽然有些快，但下面是一些要记住的重点：

- 在`do`代码块里，每个表达式必须是`IO a`型的。然后就有了可用表达式数量的限制。如，`getLine`、`print`、`putStrLn`等等...
- 尽可能多的具体化纯函数
- 类型`IO a`的意思是：一个返回`a`型元素的 IO 动作。`IO`代表动作；高级点说，`IO a`是一个函数的类型。如果你好奇就继续看下一章。

只要稍微联系下，你就能使用`IO`了。

	练习：
	- 写一个程序来求它所有参数的和。提示：使用函数`getArgs`。
	
## 4.2 IO 技巧解释

here is a pic

	本章摘要
	
	要区分纯净的部分和不纯的部分，`main`被定义为修改世界状态的函数
	
	main :: World -> World
	
	一个函数只有在它有这种类型的时候才确保有边界效应。但是看一下典型的 main 函数：
	
	main w0 =
    let (v1,w1) = action1 w0 in
    let (v2,w2) = action2 v1 w1 in
    let (v3,w3) = action3 v2 w2 in
    action4 v3 w3
    
	我们有许多要传入下一步的临时变量(`w1`,`w2`,`w3`)。
	
	我们创建一个函数`bind`或者`(>>=)`。有了`bind`我们不再需要临时变量了。
	main =
		action1 >>= action2 >>= action3 >>= action4
		
	附：Haskell为我们提供了语法糖：
	main = do
		v1 <- action1
		v2 <- action2 v1
		v3 <- action3 v2
		action4 v3
		
为什么我们要用这种奇怪的语法？这种`IO`类型实际上是什么？它看起来更像魔术。

现在让我们忘了程序中所有纯净的部分，然后把注意力集中在不纯净的部分上：

	askUser :: IO [Integer]
	askUser = do
		putStrLn "Enter a list of numbers (separated by commas):"
		input <- getLine
		let maybeList = getListFromString input in
			case maybeList of
          Just l  -> return l
          Nothing -> askUser

	main :: IO ()
	main = do
		list <- askUser
		print $ sum list
		
第一印象是这看起来是命令式的。Haskell 足够强大以至它可以让不纯净的代码看起来是命令式的。比如说，如果你想你能在 Haskell 里创建`while`。事实上，对于处理`IO`来说，命令式风格一般来说更合适。

但是你得注意表示方法有点不寻常。这就是详细的原因。

在非纯语言中，world 的状态可以被视为一个隐藏的全局变量。这个隐藏变量能被所有函数访问。比如说，你可以在任何函数中读写文件。world 所处的可能状态的区别在于该文件是否存在。

Haskell 里这种状态不是隐藏的。相反，很明确的是主函数是个可能改变 world 状态的函数。所以它的类型是像这样的：

	main :: World -> World
		
不是所有函数都能访问这个变量。那些能访问这个变量的函数是不纯的。world 变量没有提供访问的函数是纯的 **注6**。

Haskell 把 world 的状态当做`main`的输入变量。但是 main 的真正类型接近于这一个 **注7**：

	main :: World -> ((),World)
	
`()`型是单元类型。这里不会有介绍。

现在我们记着这点来重写我们的 main 函数：

	main w0 =
		let (list,w1) = askUser w0 in
		let (x,w2) = print (sum list,w1) in
		x
		
首先我们可以注意到的是所有有副作用的函数都是以下这种类型：

	World -> (a,World)
	
其中`a`是结果的类型。举例来说，`getChar`函数应该是`World -> (Char,World)`型的。

另一点要注意的是确定求值顺序的技巧。Haskell 中要求`f a b`的值，你有许多选择：

- 先求`a`，再求`b`，然后求`f a b`
- 先求`b`，再求`a`，然后求`f a b`
- 同时求`a`和`b`然后求`f a b`

这是真的因为我们正关注语言中纯的部分。

现在再看看主函数，很明显你必须在第二行前先求第一行的值，**因为要求第二行前你必须拿到第一行结果给出的参数**。

这个把戏效果很好。编译器会在每一步提供一个指向新的真实 world id 的指针。深入之下，`print`会这样执行：

- 打印东西到屏幕上
- 修改 world 的 id
- 计算`((),new world id)`

现在来看看主函数的风格，这显然是不优美的。我们来试着对 askUser 函数做相同的事情：

	askUSer :: World -> ([Integer],World)
	
原来的：

	askUser :: IO [Integer]
	askUSer = do 
		putStrLn "Enter a list of numbers:"
		input <- getLine
		let maybeList = getListFromString input in
			case maybeList of
					Just l -> return l
					Nothing ->askUser
					
修改后：

	askUSer w0 = 
		let (_,w1) = putStrLn "Enter a list of numbers:" in
		let (input,w2) = getLine w1 in
		let (l,w3) = case getListFromString input of
							Just l -> (l,w2)
							Nothing -> askUser w2
		in
				(l,w3)
				
虽然是想死，但却不优美。看到那些临时变量名`w?`了吧。

这给我们的教训是：纯函数语言中的天真的 IO 执行是不优美的。

幸运的是，有种更好的处理这个问题的方法。我们来看一种模式。每一行是这种形式的：

	let (y,w') = action x w in
	
即使一些行中不需要第一个参数`x`。输出类型是一对，`(answer,newWorldValue)`。每个函数`f`都有类似于下面这样的类型：

	f :: World -> (a,World)
	
不仅仅是这点，我们也能注意到我们总是跟随同样的用法模式：

	let (y,w1) = action1 w0 in
	let (z,w2) = action2 w2 in
	let (t,w3) = action3 w2 in
	...
	
每个动作都能采用0到 n 个参数。特别地，每个动作可以把上一行的结果作为参数。

比如说，我们可以这样：

	let (_,w1) = action1 x w0 in
	let (z,w2) = action2 w1 in
	let (_,w3) = action3 x z w2 in
	..
	
当然还有 `actionN w :: (World) -> (a,World)`。

	重要：只有两种要考虑的模式：
	
	let (x,w1) = action1 w0 in
	let (y,w2) = action2 x w1 in
	
	和
	
	let (_,w1) = action1 w0 in
	let (y,w2) = action2 w1 in
	
现在来玩个神奇的把戏。我们会让临时 world 符号"消失"。我们会`捆绑`这两行。我们来定义`bind`函数。它的类型起初是很吓人的：

	bind :: (World -> (a,World))
				-> (a -> (World -> (b,World)))
				-> (World -> (b,World))
				
但是请记住`(World -> (a,World))`是一种 IO 动作的类型。现在为了清晰我们重命名它：

	type IO a = World -> (a,World)
	
一些函数例子：

	getLine :: IO String
	print :: Show a => a -> IO ()
	
`getLine`是个 IO 动作，它将 world 作为参数然后返回一对`(String,World)`。可以总结为：`getLine`是`IO String`型，我们也将它作为一个 IO 动作，它会返回一个字符串"里面内嵌一个 IO"。

`print`函数也很有趣。它采用一个能被显示的参数。实际上它用了两个参数。第一个是要输出的值，另一个是 world 的状态。然后它返回一对`((),World)`。这意味着它改变了 world 的状态，但是不会产生更多的数据。

这种类型能帮助我们简化`bind`的类型：

	bind :: IO a
				-> (a -> IO b)
				-> IO b
				
这是说`bind`用两个 IO 动作作为参数然后返回了另一个 IO 动作。

现在记住几个重要的模型。第一个是：

	let (x,w1) = action1 w0 in
	let (y,w2) = action2 x w1 in
	(y,w2)
	
看看这几个类型：

	action1 :: IO a
	action2 :: a -> IO b
	(y,w2) :: IO b
	
是不是很眼熟？

	(bind action1 action2) w0 =
		let (x,w1) = action1 w0
				(y,w2) = action2 x w1
		in 	(y,w2)
		
这种想法是为了隐藏函数的 World 函数。我们来试试：作为例子设想假如我们想要模拟：

	let (line1,w1) = getLine w0 in
	let ((),w2) = print line1 in
	((),w2)
	
现在，使用捆绑函数：

	(res,w2) = (bind getLine (\l -> print l)) w0
	
由于 print 是`(World -> ((),World))`型的，我们可以得到`res = ()`（null 型）。如果你没看到神奇之处在哪，我们这次来试试三行的。

	let (line1,w1) = getLine w0 in
	let (line2,w2) = getLine w1 in
	let ((),w3) = print (line1 ++ line2) in
	((),w3)
	
等价于：

	(res,w3) = (bind getLine (\line1 ->
						(bind getLine (\line2 ->
							print (line1 ++ line2))))) w0

你没注意到什么吗？不，任何地方都没有临时的 world 变量了。这就是 MAGIC。

我们可以用更好的表示法。使用`(>>=)`来代替`bind`。`(>>=)`是个和`(+)`一样的中缀函数；提醒下 `3 + 4 ⇔ (+) 3 4`

	(res,w3) = (getLine >>=
						(\line1 -> getLine >>=
						(\line2 -> print (line1 ++ line2)))) w0
						
嗬嗬嗬！大家圣诞快乐！Haskell 为我们提供了语法糖：

	do 
		x <- action1
		y <- action2
		z <- action3
		...
		
被替换为：

	aciton1 >>= (\x ->
	aciton2 >>= (\y ->
	action3 >>= (\z ->
	...
	)))
	
请注意你可以在`action2`中使用`x`，在`action3`中使用`x`和`y`。

但是没有使用`<-`的行怎么办呢？这很容易，只要用另一个函数`blindBind`：

	blindBind :: IO a -> IO b -> IO b
	blindBind action1 action2 w0 =
			bind action (\_ -> action2) w0
			
为了清楚起见，我没有简化定义。当然我们可以用更好的表示法，我们将使用操作符`(>>)`。

所以

	do
		action1
		action2
		action3
		
被转化为：

	action1 >>
	aciton2 >>
	action3
	
而且，另一个函数也很有用。

	putInIO :: a -> IO a
	putInIO x = IO(\w -> (x,w))
	
这是把纯值放进"IO 上下文"中的一般方法。`putInIO`的通用名称是`return`。在你学 Haskell 时这是个糟糕的名称。`return`于以前习惯的用发十分不同。

为了完成代码，我们来翻译下这个例子：

	askUSer :: IO[Integer]
	askUSer = do
		putStrLn "Enter a list of numbers (separated by commas):"
		input <- getLine
		let maybeList = getListFromString input in
			case maybeList of
					Just l -> return l
					Nothing -> askUSer
					
	main :: IO ()
	main = do
		list <- askUser
		print $ sum list
		
它被翻译为：

	import Data.Maybe
	
	maybeRead :: Read a => String -> Maybe a
	maybeRead s = case reads s of
								[(x,"")]	-> Just x
								-				-> Nothing
	getListFromString :: String -> Maybe [Integer]
	getListFromString str = maybeRead $ "[" ++ str ++"]"
	askUser :: IO [Integer]
	askUser = 
			putStrLn "Enter a list of numbers (sep. by commas):" >>
			getLine >>= \input ->
			let maybeList = getListFromString input in
				case maybeList of 
					Just l -> return l
					Nothing -> askUser
					
	main :: IO ()
	main = askUser >>=
		\list -> print $ sum list
		
	你可以编译这段代码验证下它能否运行。
	
设想要是没有`(>>)`和`(>>=)`它会成什么样。

## 4.3 Monads

here is a pic

现在可以揭开谜底了：`IO`是个 monad。成为 monad 意味着你能用`do`表示法来访问一些语法糖。但是主要来说，你得到了一种编码模式，它能简化你的代码流。

> 重要备注:
> 
> - monad不是必须有关副作用！实际上有许多纯 monad
> - monad 更加有关的是排序

Haskell 中，`Moad`是一种类型类。要成为这种类型类的实例，你必须提供函数`(>>=)`和`return`。函数`(>>)`由`(>>=)`衍生而来。下面的代码介绍了类型类`Monad`是如何(基本地)定义的：

	class Monad m  where
	  (>>=) :: m a -> (a -> m b) -> m b
	  return :: a -> m a
	
	  (>>) :: m a -> m b -> m b
	  f >> g = f >>= \_ -> g
	
	  -- You should generally safely ignore this function
	  -- which I believe exists for historical reasons
	  fail :: String -> m a
	  fail = error

> 备注：

> - 关键字`class`不是你的朋友。一个 Haskell 类不是面向对象编程中的那种类。它与 Java 的接口有许多相似点。一个更好的词是`typeclass`，因为那代表一系列类型。对于属于类中的一个类型，类的所有函数必须支持这种类型
> - 这个特别的类型类例子中，类型`m`m 必须是采用一个参数的类型。比如说，`IO a`，`Maybe a`，`[a]`等等都是。
> - 要成为有用的 monad，你的函数必须遵循一些规则。如果你的构造不遵循这些规则，会发生奇怪的事情：
> 
> ~ return a >>= k ==k a m >>= return == m m >>= (-> k x >>= h ) == ( m >>= k ) >>= h ~

### 4.3.1 Maybe 是种 monad
有许多不同的类型是`Monad`的实例。其中最容易描述中的一种是`Maybe`。如果你有一串`Maybe`值，你可以使用 monads 来操控它们。它会特别有用当你要移除像`if..then..else..`这样的深构造时。

设想有一个复杂的银行操作。你只有在跟随一些列操作后还保持余额大于0的情况下才有资格获得大约700€ 。

	deposit  value account = account + value
	withdraw value account = account - value
	
	eligible :: (Num a,Ord a) => a -> Bool
	eligible account =
	  let account1 = deposit 100 account in
	    if (account1 < 0)
	    then False
	    else
	      let account2 = withdraw 200 account1 in
	      if (account2 < 0)
	      then False
	      else
	        let account3 = deposit 100 account2 in
	        if (account3 < 0)
	        then False
	        else
	          let account4 = withdraw 300 account3 in
	          if (account4 < 0)
	          then False
	          else
	            let account5 = deposit 1000 account4 in
	            if (account5 < 0)
	            then False
	            else
	              True
	
	main = do
	  print $ eligible 300 -- True
	  print $ eligible 299 -- False

现在，我们用 Maybe 来改善它，并且实际上这是个 Monad

	deposit :: (Num a) => a -> a -> Maybe a
	deposit value account = Just (account + value)
	
	withdraw :: (Num a,Ord a) => a -> a -> Maybe a
	withdraw value account = if (account < value) 
	                         then Nothing 
	                         else Just (account - value)
	
	eligible :: (Num a, Ord a) => a -> Maybe Bool
	eligible account = do
	  account1 <- deposit 100 account 
	  account2 <- withdraw 200 account1 
	  account3 <- deposit 100 account2 
	  account4 <- withdraw 300 account3 
	  account5 <- deposit 1000 account4
	  Just True
	
	main = do
	  print $ eligible 300 -- Just True
	  print $ eligible 299 -- Nothing


还不错，不过我们甚至可以做的更好：

	deposit :: (Num a) => a -> a -> Maybe a
	deposit value account = Just (account + value)
	
	withdraw :: (Num a,Ord a) => a -> a -> Maybe a
	withdraw value account = if (account < value) 
	                         then Nothing 
	                         else Just (account - value)
	
	eligible :: (Num a, Ord a) => a -> Maybe Bool
	eligible account =
	  deposit 100 account >>=
	  withdraw 200 >>=
	  deposit 100  >>=
	  withdraw 300 >>=
	  deposit 1000 >>
	  return True
	
	main = do
	  print $ eligible 300 -- Just True
	  print $ eligible 299 -- Nothing

我们已经证明过 Monasd 是一种让带吗更简洁的好方法。注意下代码组织的想法，特别是`Maybe`可以被用在大多数命令式语言中。事实上这是我们自然而然创建的构造。

> 重要备注：

> 序列中值为`Nothing`的第一个元素会停止完整的运算。这意味着你不用执行所有的行。你免费得到了这一点，得益于惰性。

你也可以在心里重做这些例子用`(>>=)`代替`Maybe`：

	instance Monad Maybe where
	    (>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
	    Nothing  >>= _  = Nothing
	    (Just x) >>= f  = f x
	
	    return x = Just x

monad`Maybe`在简单的例子中被证明是十分有用的。我们看到了`IO`monad的实用。但是现在来个更酷的例子，列表。

### 4.3.2 列表 monad
here is a pic

列表 monad 帮助我们模拟非确定性计算。我们开始吧：

	import Control.Monad (guard)
	
	allCases = [1..10]
	
	resolve :: [(Int,Int,Int)]
	resolve = do
	              x <- allCases
	              y <- allCases
	              z <- allCases
	              guard $ 4*x + 2*y < z
	              return (x,y,z)
	
	main = do
	  print resolve

奇迹在此发生：

	[(1,1,7),(1,1,8),(1,1,9),(1,1,10),(1,2,9),(1,2,10)]
	
列表 monad 也有这个语法糖：

	  print $ [ (x,y,z) | x <- allCases,
	                      y <- allCases,
	                      z <- allCases,
	                      4*x + 2*y < z ]

我不会列举所有的 monads，只列表他们中的一部分。使用 monad 简化了春语言中一些概念的处理。特别地，monad 对于以下几项特别有帮助：

- IO，
- 非确定性计算，
- 生成非随机数，
- 保持配置状态，
- 写的状态
- ...

你如果跟我到了这里，那证明你做到了！你知道了 monad **注解8**！

# 5.附录

这一部分和学习 Haskell 不是那么相关。只是顺便讨论下更多的细节。

## 5.1 更多关于无限树的内容

在*无限结构*这一章节中我们看到了简单的构造。不幸的是我们移除了树的两个属性：

1. 没有重复的节点值
2. 有序树

在这一节中，我们会尝试保持第一个性质。至于第二个，我们必须放宽但我们会讨论如何尽可能的保持它。

第一部是创建一个伪随机数列表：

	shuffle = map (\x -> (x*3123) `mod` 4331) [1..]
	
作为提醒，这里是`treeFromList`的定义

	treeFromList :: (Ord a) => [a] -> BinTree a
	treeFromList []    = Empty
	treeFromList (x:xs) = Node x (treeFromList (filter (<x) xs))
                             (treeFromList (filter (>x) xs))
                             
以及`treeTakeDepth`的：

	treeTakeDepth _ Empty = Empty
	treeTakeDepth 0 _     = Empty
	treeTakeDepth n (Node x left right) = let
          	nl = treeTakeDepth (n-1) left
          	nr = treeTakeDepth (n-1) right
          	in
              	Node x nl nr
              	
看一下下面代码的结果：

	main = do
	      putStrLn "take 10 shuffle"
	      print $ take 10 shuffle
	      putStrLn "\ntreeTakeDepth 4 (treeFromList shuffle)"
	      print $ treeTakeDepth 4 (treeFromList shuffle)
<br>

	% runghc 02_Hard_Part/41_Infinites_Structures.lhs
	take 10 shuffle
	[3123,1915,707,3830,2622,1414,206,3329,2121,913]
	treeTakeDepth 4 (treeFromList shuffle)
	
	< 3123
	: |--1915
	: |  |--707
	: |  |  |--206
	: |  |  `--1414
	: |  `--2622
	: |     |--2121
	: |     `--2828
	: `--3830
	:    |--3329
	:    |  |--3240
	:    |  `--3535
	:    `--4036
	:       |--3947
	:       `--4242

耶！她结束了！不过要注意，它只有在你总是有东西可以放进分支的时候才能工作。

比如说
	      
	treeTakeDepth 4 (treeFromList [1..]) 
	
会一直循环。原因很简单，它会一直试着访问`filter (<1) [2..]`的头部。但是`filter`不够聪明以至它不能明白结果是空列表。

尽管如此，这依然是个很酷的例子，表明了不严格程序必须付出的代价。

以下留作读者的练习：

- 证明一个数字`n`的存在使`treeTakeDepth n (treeFromLis shuffle)`会进入无限循环。
- 找到`n`的上界
- 证明不存在`shuffle`使得程序能在任一深度结束

为了解决这些问题，我们得稍微修改下`treeFromList`和`shuffle`函数。

第一个问题是`shuffle`的执行中缺少无限不同的数字。我们只剩除了`4331`个不同的数字。为了解决这点，我们用一个稍微更好些的`shuffle`函数。

	shuffle = map rand [1..]
	          where 
	              rand x = ((p x) `mod` (x+c)) - ((x+c) `div` 2)
	              p x = m*x^2 + n*x + o -- some polynome
	              m = 3123    
	              n = 31
	              o = 7641
	              c = 1237

这个随机函数(但愿)拥有即无上限也无下限这个性质。但是有了更好的随机列表并不足以进入无限循环。

通常我们无法知道`filter (<x) xs`是否是空的。于是要解决这个问题，我会批准二叉树生成时的一些错误。新版的代码能生成自身结点没有以下属性的二叉树：

>左/右子树的任何元素必须严格小于/大于根结点。

记得它将会基本保持为一个有序二叉树。此外，每个结点在生成时都是树中唯一的。

下面是新的`treeFromList`。我们仅仅用`safefilter`代替了`filter`。

	treeFromList :: (Ord a, Show a) => [a] -> BinTree a
	treeFromList []    = Empty
	treeFromList (x:xs) = Node x left right
          	where 
              	left = treeFromList $ safefilter (<x) xs
              	right = treeFromList $ safefilter (>x) xs
				
新的函数`safefilter`基本等价于`filter`，但是在结果是有限列表的情况下不会进入无限循环。如果连续10000步以后还找不到一个判断为真的元素，那么它会被当做搜索的结束。

	safefilter :: (a -> Bool) -> [a] -> [a]
	safefilter f l = safefilter' f l nbTry
  		where
      		nbTry = 10000
      	  	safefilter' _ _ 0 = []
      	  	safefilter' _ [] _ = []
      	  	safefilter' f (x:xs) n = 
                  		if f x 
                     		then x : safefilter' f xs nbTry 
                     	   	else safefilter' f xs (n-1) 
							
现在开心地运行吧：

	main = do
      	putStrLn "take 10 shuffle"
      	print $ take 10 shuffle
      	putStrLn "\ntreeTakeDepth 8 (treeFromList shuffle)"
      	print $ treeTakeDepth 8 (treeFromList $ shuffle)

你应该意识到输出每个值的时间是不同的。这是因为 Haskell 只有在需要的时候才计算每个值。这种情况下，就是当被要求输出到屏幕的时候才计算。

足够了，现在试着把深度从`8`扩大到`100`。它能正常工作而且没有杀了你的内存！流和内存管理是由 Haskell 自然地完成的。

留给读者的练习：

- 即使给`deep`和`nbTry`以大的常数，它看起来还是工作地不错。但是最糟糕的情况下，这会是指数级的。把最糟糕的列表作为参数传给`treeFromList`。
提示：考虑下(`[0,-1,-1,....,-1,1,-1,...,-1,1,...]`)。
- 我起初这样来实现`safefilter`：

		safefilter' f l = if filter f (take 10000 1) == []
									then []
									else filter f l
解释下为什么它不能工作而进入无限循环。
- 假如`shuffle`是上限渐渐增长的真随机列表。如果你了解过这个结构，你会发现在 可能性1 下，这个结构是有限的。用下面的代码(假定我们能直接用`safefilter'`就好像它不在 safefilter 中 where 子句中)来找到`f`的定义，使得在可能性`1`下 treeFromList'随机是无限的。并证明它。免责声明，这只是个猜想。
	
	
		treeFromList' []  n = Empty
		treeFromList' (x:xs) n = Node x left right
    		where
        		left = treeFromList' (safefilter' (<x) xs (f n)
        		right = treeFromList' (safefilter' (>x) xs (f n)
        		f = ??? 

# 6.感谢
感谢[/r/haskell](http://reddit.com/r/haskell)和[/r/programming](http://reddit.com/r/programming)。你们的评论是最受欢迎的。

特别地，我想感谢[Emm](https://github.com/Emm) 一千次，因为他改正了我的英语。谢谢兄弟。

1. 即使大部分近期的语言都试着隐藏他们，但还是存在。↩
2. 我知道我作弊了。但我稍后会讨论非严格性。
3. 这里有篇更完整的类型匹配的解释。
4. 注意到`squareEvenSum''`比另两种方法更高效。所以`(.)`的顺序是很重要的。
5. 这很类似于 JavaScript 的对包含 JSON 字符串的`eval`操作。
6. 对这个规则有一些不安全的异常。但是你不会在真实应用中看到这种用发，除非 maybe 被用来 debug。
7. 对于好奇的人们，真正的类型是`data IO a = IO {unIO :: State# RealWorld -> (# State# RealWorld, a #)}。所有的`#`都以最优化执行，而且我在例子中交换了位置。但是这是基本思想。
8. 当然，你需要稍微练习下才能在使用他们和创建自己的程序时习惯和理解。不过你已经朝这个方向迈了一大步了。
