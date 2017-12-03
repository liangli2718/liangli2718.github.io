---
layout: post
title: 人人网挖坟(1)-使用Fiddler抓包分析网页结构
categories: [blog ]
tags: [python,爬虫]
description:
---

# 起源
这个系列post的起源是因为最近在学习python。首先花了两周时间熟悉了基本的语法（推荐[简明Python教程](https://www.gitbook.com/book/lenkimo/byte-of-python-chinese-edition/details)），作为一个平时主要写C/C++和Matlab的工程师，十分惊叹python的简洁优美却又功能强大。

根据以往的经验，学习一门编程语言的最好的方法是用它来实现一个特定领域的应用。首先尝试的是用python+numpy+matplotlib移植了工作中的一个简单的雷达信号处理的应用，这个简单的移植让我感觉python+numpy+matplotlib几乎可以无缝代替Matlab。

第二个想尝试的应用就是网络爬虫，所以才有了以下的各种尝试。

如前文所述说，在这里实现特定应用的目的是为了在实践中学习python以及它常用的库，如果文中有任何不妥之处，欢迎大家指正。

# 为什么是人人网
1. 微博难爬...
   看了一些关于微博模拟登录的帖子，整个过程非常复杂。况且我也没有很多微博好友，所以留作之后的项目吧  
   稍微尝试了下对人人网抓包，发现了一些线索，所以先拿人人网开刀实验。
2. 好友数量多
   不仅多，而且实名，相互熟悉。所以后期做数据处理的时候可以方便的看出结论是否合理。（例如通过状态和评论推测好友关系图）
3. 一代人的集体回忆
   如果你懂说明你老了...
   ![xiaonei](/img/2017-12-03/xiaonei_logo.gif)

# 模拟登陆/使用Cookie
对于人人网来说，要获取数据，首先要合法登陆。当然可以先抓包分析普通登陆的整个过程，然后使用python发送http request模拟这个过程进行登录，不过登录部分不是重点，所以我们直接使用合法登陆产生的cookie对人人网进行访问。
用自己的账号登陆人人网，并用Fiddler对整个过程抓包（也可以使用Chrome自带的Developer Tool)，可以得到一个cookie。有了cookie我们就可以在一段时间内免登录访问人人的页面了。

[![find_cookie](/img/2017-12-03/find_cookie.png)](/img/2017-12-03/find_cookie.png)

如上图，登录人人的时候浏览器对www.renren.com/home发出了http请求，右边栏里的request header显示了使用的cookie。右键Cookie选择copy header，然后粘贴到一个文本文件里保存就可以了。注意copy的时候Fiddler自己添加了“Cookie: ”这样的内容。这并不是cookie的一部分，删掉这个内容，确保保存的cookie是key1=value1;key2=value2;...这种形式就好。

# 获取自己的好友列表
要抓取自己好友的各种信息，首先要获得自己的好友列表。我们先去【个人主页】->【我的好友】查看自己的好友列表。
使用Fiddler对这个页面进行抓包，获得以下内容
[![friendList](/img/2017-12-03/friendList.png)](/img/2017-12-03/friendList.png)

可以看到浏览器对friend.renren.com/managefriends发出了请求，之后跳转到friend.renren.com/groupsdata。groupsdata其实一个javascript代码片段，我们把这个数据保存为friendList.txt

{% highlight javascript %}
var user={star: true, vip :false};
    var friends_manage_groups = {
    //"code" : 0,
    //"msg" : "操作成功",
    "data" : {
            "groups" :[{"gname":"组名1","gnum":37},
                    .....],

            "friends": [{"fid":朋友数字ID,"timepos":722,"fgroups":[分组名],"comf":53,"compos":266,"large_url":大头像地址,"tiny_url":"小头像地址","fname":"显示的名字","info":"家乡","pos":1}
            ......]
        }
{% endhighlight %}

不难发现，自己的所有好友都在一个javascript对象里。如果像使用python自动获取这个列表，可以使用requests库发一个GET请求，然后使用BeautifulSoup对返回的页面解析，之后获得这个列表。鉴于这个页面一次就返回了所有好友，而不是分批返回，那我就干脆直接copy-paste好友列表到一个文本文件之后再用。

因为是js对象，自然想到使用json解析。先对这个friendList.txt进行一些预处理：删掉
var user={star: true, vip :false};   
var friends_manage_groups =    
和中间的两行注释,以及文件末尾的分号，让整个文件之剩下{ "data": ...}，这就使它变成了一个无名的js对象，或者说是一个json文件。

为了测试处理过后的好友列表确实能用json解析器解析，我们把所有内容copy到一个在线的json解析器里看看：
[![online_parse](/img/2017-12-03/online_parse.png)](/img/2017-12-03/online_parse.png)

确实可以解析！这样我们就可以在python里使用这个完整的好友列表了！





