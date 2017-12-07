---
layout: post
title: 人人网挖坟(1)-使用Fiddler抓包分析网页结构并获取好友列表
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
在2017年末研究人人网的数据好像有点太落伍了，不过对于学习练手也还ok。总结起来有以下原因：

1. 微博难爬...
   看了一些关于微博模拟登录的帖子，整个过程非常复杂。况且我也没有很多微博好友，样本数量太少。所以留作之后的项目吧   
   稍微尝试了下对人人网抓包，发现了一些线索，似乎结构比较简单，反爬措施也没多少，所以先拿人人网开刀实验。
2. 好友数量多
   不仅多，而且实名，相互熟悉。所以后期做数据处理的时候可以方便的看出结论是否合理。（例如通过状态和评论推测好友关系图）
3. 一代人的集体回忆
   如果你懂说明你老了...

   ![xiaonei](/img/2017-12-03/xiaonei_logo.gif){: .center-image}

# 模拟登陆/使用Cookie
对于人人网来说，要获取数据，首先要合法登陆。当然可以先抓包分析普通登陆的整个过程，然后使用python发送http request模拟这个过程进行登录，不过登录部分不是重点，所以我们直接使用合法登陆产生的cookie对人人网进行访问。
用自己的账号登陆人人网，并用Fiddler对整个过程抓包（也可以使用Chrome自带的Developer Tool)，可以得到一个cookie。有了cookie我们就可以在一段时间内免登录访问人人的页面了。

[![find_cookie](/img/2017-12-03/find_cookie.png)](/img/2017-12-03/find_cookie.png)

如上图，登录人人的时候浏览器对www.renren.com/home发出了http请求，右边栏里的request header显示了使用的cookie。右键Cookie选择copy header，然后粘贴到一个文本文件里保存就可以了。注意copy的时候Fiddler自己添加了`“Cookie: ”`这样的内容。这并不是cookie的一部分，删掉这个内容，确保保存的cookie是`key1=value1;key2=value2;...`这种形式就好。

使用这个cookie也非常简单，只需要在发送请求的时候添加cookie就可以了：

```python
import requests

myCookie={}
with open('.\\data\\cookie.txt','r') as f:
    for line in f.read().split(';'): #split dict
        name,value=line.strip().split('=',1)
        myCookie[name]=value

response = requests.get(url='http://www.renren.com/home', cookies=myCookie)
```

# 获取自己的好友列表
要抓取自己好友的各种信息，首先要获得自己的好友列表。我们先去【个人主页】->【我的好友】查看自己的好友列表。
使用Fiddler对这个页面进行抓包，获得以下内容
[![friendList](/img/2017-12-03/friendList.png)](/img/2017-12-03/friendList.png)

可以看到浏览器对friend.renren.com/managefriends发出了请求，之后跳转到friend.renren.com/groupsdata。`groupsdata`其实一个javascript代码片段，我们把这个数据保存为`friendList.txt`

```javascript
var user={star: true, vip :false};
var friends_manage_groups = {
  //"code" : 0,
  //"msg" : "操作成功",
  "data" : {"groups" :[{"gname":"组名1","gnum":37},
                  .....],
          "friends": [{"fid":"朋友数字ID",
                    "timepos":722,
                    "fgroups":["分组名"],
                    "comf":53,"compos":266,
                    "large_url":"大头像地址",
                    "tiny_url":"小头像地址",
                    "fname":"显示的名字",
                    "info":"家乡",
                    "pos":1},
                    ......]
      }
};
```

不难发现，自己的所有好友都在一个javascript对象里。如果想使用python自动获取这个列表，可以使用requests库对http://friend.renren.com/groupsdata发一个GET请求，然后处理返回的js代码片，之后获得这个列表。鉴于这个页面一次就返回了所有好友，而不是分批返回，那我就干脆直接copy-paste好友列表到一个文本文件之后再用。

因为是js对象，自然想到使用json解析。先对这个`friendList.txt`进行一些预处理：删掉
var user={star: true, vip :false};   
var friends_manage_groups =    
和中间的两行注释,以及文件最末尾的分号，让整个文件只剩下{ "data": ...}，这就使它变成了一个无名的js对象，或者说是一个json文件。

为了测试处理过后的好友列表确实能用json解析器解析，我们把所有内容copy到一个在线的json解析器里看看：
[![online_parse](/img/2017-12-03/online_parse.png)](/img/2017-12-03/online_parse.png)

确实可以解析！这样我们就可以在python里使用这个完整的好友列表了！

# 进一步挖掘
从上面的json结构可以得到一些有用的信息：好友的`fid`(friend identifier?)，显示名字`fname`(friend name?)，头像图片地址`large_url/tiny_url`等。

`fid`是一个人人网用户的唯一标识符。`fid`的重要之处在于在我们之后需要获取相册信息，相册内容，状态内容甚至评论的时候，都需要`fid`来构造相应的url。后续数据处理的时候`fid`也可以作为标注用户关系的标识符。

相比之下，`fname`就不是特别重要了，它只是给我们一个可以显示的用户名称，比如“张三”，“李四-cool”等。

`large_url/tiny_url`两个url分别对应用户头像的大小分辨率版本。url的形式是http://hdn.xnimg.cn/photos/hdnXXX/yyyymmdd/YYYY/h_large_somestring.jpg 我尝试了一下这些url在没有登录的状态下居然也可以正常访问。这就意味着只要有了好友头像的url，任何人无需登录就可以访问到头像数据。这样做看似暴露用户隐私，然而实际上是为了网页的功能考虑：当未登录用户在网上搜到一个profile想要查看时，人人网展示的页面包含非常有限的内容（其他详细内容需要注册登录之后才能查看），头像就是其中一项。如果之前已经按照上文正确的提取了`friendList.txt`，可以使用以下脚本提取所有好友的头像并保存到head文件夹：

```python
import json
import requests
from contextlib import closing

with open(".\\data\\friendList.txt", 'r', encoding='utf-8') as f:
    friendList = f.read()

friendListJson = json.loads(friendList)
friends = friendListJson['data']['friends']

fCount = 0
for item in friends:
    fName = item['fname']
    target = item['large_url']

    print('Getting head of:', fName)
    print(target)

    with closing(requests.get(url=target, stream=True)) as r:
        with open('./head/%s.jpg' % fName, 'ab+') as f:
            for chunk in r.iter_content(chunk_size=1024):
                if chunk:
                    f.write(chunk)
                    f.flush()

    fCount+=1   

    # time.sleep(round(random.random()+0.5))

print('In total:',fCount)
```

效果差不多就是这样（博主有接近1000个好友）。。。
![head](/img/2017-12-03/head.png)

`friendList`里的`info`数据存储了好友的家乡，然而人人网的实现非常混乱，我在`info`里面不只看到省份名称，还有学校名称和空值。所以下面的代码过滤了所有是省份的非空值然后排序：

```python
import json
import numpy as np
import matplotlib.pyplot as plt
import matplotlib

matplotlib.rcParams['font.sans-serif']='Microsoft YaHei'

with open(".\\data\\friendList.txt", 'r', encoding='utf-8') as f:
    friendList = f.read()

friendListJson = json.loads(friendList)
friends = friendListJson['data']['friends']

home = {}
for item in friends:
    #get home info, and filter out all other infomation
    homeProv = item['info']
    if(0<len(homeProv.strip())<=3):
        if homeProv in home:
            home[homeProv]+=1
        else:
            home[homeProv]=1

prov=[]
cnt=[]
for key in home.keys():
    prov.append(key)
    cnt.append(home[key])

p = np.array(prov)
c = np.array(cnt)

ind = np.argsort(c)
ind=ind[::-1]

# pie plot with top 10 home towns
plt.figure(figsize=(5,5))
plt.pie(c[ind[0:10]],labels=p[ind[0:10]], autopct=lambda p: '{:.0f}'.format(p * sum(c[ind[0:10]]) / 100))
plt.show()
```

结果是我的前十名的好友分布饼图：

![dist](/img/2017-12-03/pie.png){: .center-image}

# 剧透
这篇到此为止，通过好友列表能得到的信息并不多，但是它却是通往更多信息的大门。因为我的好友又有他们自己的好友列表，我的好友的好友又有自己的好友列表，子子孙孙无穷尽也。通过这个列表，单点的一个人和其他人连成一个网络图（顶点，边，权重），我们可以探索一个人或者整个社区的人际网络图，并且挖掘更多的信息。我们之后的几个post在议！


