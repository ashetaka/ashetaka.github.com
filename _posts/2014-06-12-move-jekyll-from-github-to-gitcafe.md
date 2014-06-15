---
layout: post
title: "将 jekyll 从 github-pages转到 gitcafe-pages"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

之前博客一直放在 github 的 page 服务上，访问速度算是一般，毕竟服务器在国外。最近刚好看了唐巧的『将博客从GitHub迁移到GitCafe』一文，他用的是 octopress。虽然我用的 jekyll，不过方法也大同小异。换了以后，访问速度也有了很明显的提升。这点可以从下面的 ping 的对比有很直观的感受,基本就是从300ms 到了5ms 的改变,所以对于国内用户来说，我还是很推荐迁移过来的，当然过程中发现 gitcafe 也存在一个问题，这点会在后面介绍。

{% highlight sh startinline   %}
➜  output  ping -c 5 ashitaka.me
PING ashitaka.me (204.232.175.78): 56 data bytes
64 bytes from 204.232.175.78: icmp_seq=0 ttl=45 time=411.812 ms
64 bytes from 204.232.175.78: icmp_seq=1 ttl=45 time=434.791 ms
64 bytes from 204.232.175.78: icmp_seq=2 ttl=45 time=355.194 ms
64 bytes from 204.232.175.78: icmp_seq=3 ttl=45 time=378.048 ms
64 bytes from 204.232.175.78: icmp_seq=4 ttl=45 time=401.062 ms

--- ashitaka.me ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 355.194/396.181/434.791/27.450 ms

➜  output  ping -c 5 ashitaka.me
PING ashitaka.me (117.79.146.98): 56 data bytes
64 bytes from 117.79.146.98: icmp_seq=0 ttl=51 time=7.900 ms
64 bytes from 117.79.146.98: icmp_seq=1 ttl=51 time=6.520 ms
64 bytes from 117.79.146.98: icmp_seq=2 ttl=51 time=5.144 ms
64 bytes from 117.79.146.98: icmp_seq=3 ttl=51 time=5.393 ms
64 bytes from 117.79.146.98: icmp_seq=4 ttl=51 time=2.577 ms

--- ashitaka.me ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 2.577/5.507/7.900/1.760 ms
{% endhighlight %}

### 迁移仓库

首先是要把代码仓库迁移到 gitcafe 上，为此我们需要注册一个 gitcafe 帐号，之后建立一个仓库。为了使用 gitcafe 的 pages 服务，我们只需要做到以下两点：

+ 将仓库名命名为自己的用户名
+ 给仓库建立一个叫 gitcafe-pages 的分支

建好以后就可以将 github 上的仓库拖进来了

{% highlight sh startinline linenos  %}
git remote add gitcafe git@gitcafe.com:Ashitaka/Ashitaka.git >> /dev/null 2>&1

git push -u gitcafe master:gitcafe-pages

{% endhighlight %}

这段代码用于将代码推送到刚才我们新建的仓库中，以后为了方便在 github 和 gitcafe 同时同步代码，可以将这部分代码放在 shell 脚本里，方便更新。

这个时候，如果页面生成和渲染没问题的话，直接访问 Ashitaka.gitcafe.com 就可以看到博客了。

补充一下，提交的时候，需要你已设好 SSH key,你可以重新生成一个 key，然后到 gitcafe 的 ssh 管理里加入，也可以和我一样和 github 共用一个 ssh key。方法也很简单，进入 gitcafe 的账户设置-SSH 公钥管理,点击添加新的公钥，然后把~/.ssh/id_rsa.pub的内容贴上去。

接着再编辑 ~/.ssh/config 文件, 加入如下内容

{% highlight sh startinline linenos  %}
Host github.com
 HostName github.com
 User ashetaka
 IdentityFile ~/.ssh/id_rsa

Host gitcafe.com
 HostName gitcafe.com
 User Ashitaka
 IdentityFile ~/.ssh/id_rsa
{% endhighlight %}

 这样 SSH key 就搞定了

### 修改域名

接下来就该把域名指向到 gitcafe 的 page 服务地址了。进入上面创面的项目，然后进入项目管理-自定义域名，将自己的域名加进去。

![]({{ site.RES_PATH }}/img/gitcafe-domain.png)

之后就是修改域名的 A 记录，由于我使用的是 dnspod 来解析域名，所以打开 dnspod 的域名管理，然后修改其中的 A 记录地址到117.79.146.98

![]({{ site.RES_PATH }}/img/dnspod-a.png)

之后稍等几分钟，修改就能生效了，你就能感受到文章开始贴出的那种 ping 值的变化

### 一个小问题

我用的是 jekyll 来生成静态博客，jekyll 会在_site 目录中生成博客的最终内容。这其中就有一部分以文章的 category 为文件名的目录。这些目录存储的就是博客里的各篇文章。为了更清晰一点说明，我们先来看一下一篇博文的 URL

{% highlight sh startinline   %}
http://ashitaka.me/learning/2014/01/14/make-php-more-secure/
{% endhighlight %}

在这个 URL 中，learning 就是 category 名，之后是年份，月份和日期，最后定位到文章。我原先用的 category 名是中文的，这就导致_site 目录下有几个中文名命名的目录，结果这种中文目录在 gitcafe 的解析过程中就出了问题，报了以下错误：

> 這裏是 Jekyll 生成過程的輸出:
Configuration file: /home/gitcafe/gitcafe_pages_tmp/ashitaka/_config.yml Source: /home/gitcafe/gitcafe_pages_tmp/ashitaka Destination: /home/git/repos/Ashitaka/Ashitaka.git/_site/ Generating... jekyll 2.0.3 | Error: invalid byte sequence in US-ASCII 

当然这个问题目前还有待gitcafe 官方去解决，我这边为了能正常生成，就把 category 都改成了中文的。


### one more thing

转移过来之后，还是比较稳定的，除了上面那个中文目录出错的情况，暂时也没发现别的问题。如果你想改善博客访问速度的话，迁移过来是个不错的选择。