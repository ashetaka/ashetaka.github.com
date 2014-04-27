---
layout: post
title: "通过代理来实现hosts切换"
description: ""
category: ""
tags: [python,pycurl]
---
{% include JB/setup %}

现在的WEB应用中，API正扮演越来越重要的角色。不论是开发人员还是测试人员，都会或多或少的面临以下情况中的一种或更多：

1. 访问另一套环境，如从开发环境，要切换到测试环境
2. 对比线上和开发环境API结果
3. 测试结束后，去掉测试环境的配置

以上几种情形，可以说web项目相关人员每天都有接触。最常用的用法也就是通过修改hosts文件来改变本机的访问去向，通过hosts，而不是DNS来决定要访问的IP。

假设当前有一个测试某API的简单脚本，该API可能部署在不同环境上，同时线上也有一份。这时候如果要切换环境执行测试脚本，除了修改hosts，还有另一种思路：就是通过代理来切换。


直接上代码

{% highlight py startinline linenos  %}
from StringIO import StringIO
import pycurl


def sendRequest(url):
    storage = StringIO()
    c = pycurl.Curl()
    c.setopt(c.URL, url)
    c.setopt(c.WRITEFUNCTION, storage.write)
    c.setopt(c.PROXY, "http://127.0.0.1:8087")
    c.perform()
    c.close()
    content = storage.getvalue()

    return content

url = "http://www.youtube.com"
response = sendRequest(url)
print response
{% endhighlight %} 

这里用了墙外的youtube来做演示。如果不用setpot来配置CURLOPT_PROXY(该选项的介绍[点击查看](http://curl.haxx.se/libcurl/c/curl_easy_setopt.html))，那就是直接访问相应的url，这种情况的流程一般就是：hosts或dns来解析域名，获得ip后访问。配上CURLOPT_PROXY以后呢？其过程则是直接将请求发至proxy设置的地址来访问。因此，在这里，我把proxy配到本机8087端口(8087是个眼熟的端口吧。没错正是goagent的默认端口)上，这样这段代码就能访问到youtube了。

也许有人会说，hosts设置和代理是两码事吧。没错，的确是两回事，但是对于你的curl脚本来说，通常只会访问到API相关的url，也就是你设置的CURLOPT_URL，给脚本配置CURLOPT_PROXY以后，脚本的curl对象就会通过代理去访问url，相当于是修改了域名指向的ip。所以在这里，其实是殊途同归。


