---
layout: post
title: "pycurl简介"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

PHP下的curl用的很多了，最近刚好在学python，发现了pycurl这个库，就正好使用了一下。

### 安装

按官网doc的描述一步步下来

{% highlight sh startinline %}
wget https://pypi.python.org/packages/source/p/pycurl/pycurl-7.19.3.1.tar.gz\#md5\=6df8fa7fe8b680d93248da1f8d4fcd12 -P ~/Downloads/
cd ~/Downloads
tar zxvf pycurl-7.19.3.1.tar.gz
cd pycurl-7.19.3.1
python setup.py install
{% endhighlight %} 

结果报错

{% highlight sh startinline %}
clang error: unknown argument: '-mno-fused-madd' (python package installation failure)
{% endhighlight %} 

尝试用配置去掉一个参数后就OK了

{% highlight sh startinline %}
sudo ARCHFLAGS="-arch x86_64" CFLAGS=-Wunused-command-line-argument-hard-error-in-future python setup.py install
{% endhighlight %} 

进入python交互界面，验证下能否使用

{% highlight py startinline %}
>>> import pycurl
>>> pycurl.version
'PycURL/7.19.3.1 libcurl/7.30.0 SecureTransport zlib/1.2.5'
{% endhighlight %}

看来一切顺利

### 请求

这是一个简单的实现返回指定url的请求结果的函数，并且假定返回的是json格式

{% highlight py startinline linenos  %}
def sendRequest(url):
    storage = StringIO()
    c = pycurl.Curl()
    c.setopt(c.URL, url)
    c.setopt(c.WRITEFUNCTION, storage.write)
    c.perform()
    c.close()
    content = storage.getvalue()

    decoded = json.loads(content)

    return decoded

{% endhighlight %} 

### 参考链接

1.Pycurl Error : AttributeError: 'module' object has no attribute 'Curl'

http://stackoverflow.com/questions/12078168/pycurl-error-attributeerror-module-object-has-no-attribute-curl

2.Getting HTML with Pycurl

http://stackoverflow.com/questions/6554386/getting-html-with-pycurl