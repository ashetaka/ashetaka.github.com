---
layout: post
title: "MAC下上手PostgreSQL"
description: ""
category: ""
tags: [PostgreSQL]
---
{% include JB/setup %}


本文介绍MAC下PostgreSQL的安装和简单使用方法。

### 概要

<img src="{{ site.RES_PATH }}/img/225px-Postgresql_elephant.svg.png" class="center"/>


PostgresSQL是完全开放、免费的对象-关系数据库服务器。其发音为post-GRES-que-ell，你也可以参考这个[音频文件](http://www.postgresql.org/files/postgresql.mp3)。

### 安装

直接用brew即可安装

	brew install postgresql
	
接下来初始化PostgreSQL：

	initdb /usr/local/var/postgres/ -E utf8
	
如果想让PostgreSQL开机自启动，可以用以下两条命令：

    ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
    
对于服务的控制，就要用到pg_ctl这个命令，比如如果要启动：

	pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start

### 创建数据库

PostgreSQL在安装后可以直接在shell里输入一些命令来操作数据库。

比如用createdb来创建一个数据库：

	➜  ~  >createdb test -O ashitaka -E UTF8 -e
	CREATE DATABASE test OWNER ashitaka ENCODING 'UTF8';

在这里test是数据库名，ashitaka为owner。

### 连接数据库

要连接PostgreSQL，可以这样：

	➜  ~  >psql -U ashitaka -d test -h 127.0.0.1
	psql (9.3.2)
	Type "help" for help.
	
-U指定用户名，-d指定数据库名，-h指定数据库地址。

需要注意的是，如果当前用户的名字和你指定的用户名是一致的话，就能直接进入数据库，而不用输该账户在数据库里的密码。否则，敲完这行命令以后，会提示输入密码，之后才能登陆.

如果是第一次使用，可以用这个命令来设置密码：

	test=# \password ashitaka
	Enter new password:
	Enter it again:
	
### 交互式界面操作

PostgreSQL提供了一系列交互命令。

要显示所有的数据库，可以用：

	\l
	
切换当前数据库到另一个数据库：

	\c postgres
	
显示当前库下的所有表：

	\d
	
### SQL语句操作

这个用普通的SQL语句即可，没有特别的语法。

	test=# select * from test;
	 id | text
	----+------
	(0 rows)