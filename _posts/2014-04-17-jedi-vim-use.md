---
layout: post
title: "jedi-vim配置"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

[jedi-vim](https://github.com/davidhalter/jedi-vim/) 是一款实用的python自动补全插件。本文记录下安装过程

#### 1.安装jedi依赖

{% highlight sh startinline linenos  %}
pip install jedi
{% endhighlight %} 

#### 2.安装pathogen.vim

{% highlight sh startinline linenos  %}
mkdir -p ~/.vim/autoload ~/.vim/bundle
curl -Sso ~/.vim/autoload/pathogen.vim https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim
{% endhighlight %}

#### 3.安装jedi-vim

{% highlight sh startinline linenos  %}
cd ~/.vim/bundle
git clone https://github.com/davidhalter/jedi-vim
cd jedi-vim
git submodule update --init 
{% endhighlight %}

#### 4.vimrc里配置pathogen

在vimrc里添加`execute pathogen#infect()`

实际效果

![]({{ site.RES_PATH }}/img/jedi-vim.png)