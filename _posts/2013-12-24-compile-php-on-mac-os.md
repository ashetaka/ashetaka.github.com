---
layout: post
title: "Mac OS X上编译安装 PHP"
description: ""
category: ""
tags: [php,osx]
---
{% include JB/setup %}

OS X 更新到10.9以后不知不觉把/etc/下的 php.ini 弄没了，好多原来自己加的扩展也用不了了，于是干脆重新编译个 php。

### 编译

	tar zxvf php-5.5.7.tar.gz
	cd php-5.5.7
	./configure
	make
	make install

### 命令行改为新的 php

装完 php -v 了一下发现还是5.4
看了一下实际调用的是/usr/bin/下的，可是新装的在/usr/local/bin 下，又环境变量里，第一个目录排在第二个之前，所以调用了5.4的 php

	➜  local  >which php
	/usr/bin/php

	➜  local  >sudo rm /usr/bin/php
	Password:
	➜  local  >php
	^C
	➜  local  >php -v
	PHP 5.5.7 (cli) (built: Dec 21 2013 20:42:54)
	Copyright (c) 1997-2013 The PHP Group
	Zend Engine v2.5.0, Copyright (c) 1998-2013 Zend Technologies

删除以后，终于对了。

### 让 apache 调用新的 php
写个 phpinfo()打开浏览器看一下，发现 apache 用的还是5.4的。于是乎才发觉之前编译的时候忘记重新编译 apache 的 php 模块了。

重新编译一下

	./configure --with-apxs2
	make
	make install

重启 apache

	sudo apachectl -k restart
	
再开浏览器看测试页面，终于是5.5.7的了。

#### 参考资料：

http://www.linuxfromscratch.org/blfs/view/svn/general/php.html
