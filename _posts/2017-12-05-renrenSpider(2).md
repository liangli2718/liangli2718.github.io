---
layout: post
title: 人人网挖坟(2)-获取好友的新鲜事，探索深层好友列表
categories: [blog ]
tags: [python,爬虫]
description:
---

# Intro
在上一[Post](https://liangli2718.github.io/blog/renrenSpider(1).html)中，我们成功获取了自己的全部好友列表`friendList.txt`。在这一po中，我们将探讨如何利用这个列表获取更多的相关信息。

# 不再新鲜的“新鲜事”
所谓“新鲜事”就是用户发出的公共消息。换句话说，就是：新浪微博的微博，twitter的twitter，微信的朋友圈，QQ空间的说说，美拍的美拍....。原来这么多社交网络的核心思想都是一样的：让用户发声，依靠UGC（用户产生的内容）引爆传播。下文我们统称这些东西为`“状态”（status)`

作为一个社交网络的用户，根据我没有任何研究学术背景的理解和体验，在一定程度上，状态表达了用户当前的情感或行为（即便用户不自知）。例如：

>卧槽，一出门就下暴雨，mlgb！ （行为：出行，情感：崩溃，抱怨）

或者：

>国庆节去长城玩了一下，长城没看见，看到的全是人头！（行为：去玩，情感：无奈而后悔）

所以收集用户的状态可以变成一个很好的研究数据库，对于研究NLP的人叫做“语料库”。我的人人网好友基本上已经沉寂多年了，所以这个语料库已经不能反映当前的状态，不过对于“历史研究”还是有一定意义，这也是为什么个系列叫做“人人网挖坟”。

下面我们就来看看如何获取所有好友的所有状态。前提是在使用python的requests库进行网页访问的时候，需要像上一个帖子中一样，使用cookie来避免进行登录。

在人人网中，每个用户都会有“状态”这一个页面：

![statusPage](/img/2017-12-03/statusPage.png){: .center-image}

在每个好友的这个页面下，可以看到他的所有历史状态。这个页面对应的url是http://status.renren.com/status/v7/`fid`,这里正是用到了上一个post中获取的好友fid。根据上一post的套路，这里自然是使用Fiddler（或者Chrome的Dev Tool）对这个页面的访问进行抓包。

可以看到访问这个页面的时候，页面发出了一个请求http://status.renren.com/GetSomeomeDoingList.do?userId=`fid`&curpage=`pageNumber`，并且返回了一个json object。

>add json obj

可惜的是，不像获取好友列表，这个状态页面是分页的。通过curpage这个url参数选择返回哪一页状态。所以对每个好友，我们需要遍历所有的状态页。如何判断已经遍历完毕所有的页呢？一个简单的方法就是看返回的json里的`doingArray`属性是否为空，如果为空，说明之前的页面已经显示完了所有状态。可以使用以下代码进行遍历：

```python
import json
import requests
import random
import time
from contextlib import closing
import re
import sys

def get_valid_filename(s):
    s = s.strip().replace(' ', '_')
    return re.sub(r'(?u)[^-\w.]', '', s)

#read cookies for session
cookies={}
with open('.\\data\\cookie_nov29.txt','r',encoding='utf-8') as f:
    pair = f.read().split(';')
    for item in pair:
        (key,value)=item.split('=')
        cookies[key]=value


# get friendlist
with open(".\\data\\friendList.txt", 'r', encoding='utf-8') as f:
    friendListFile = f.read()

friendListJson = json.loads(friendListFile)
friends = friendListJson['data']['friends']

fCount = 0
for item in friends:
    currPage = 0
    totalStatus = 0
    fid = item['fid']
    fName = item['fname']

    # information of current user
    sys.stdout.write('Getting all status of:%s,%s' % (fName,str(fid)))
    sys.stdout.flush()

    #loop through all pages of status
    while(True):
        sys.stdout.write(' p{}'.format(currPage))
        sys.stdout.flush()

        # URL to status
        statusLink = 'http://status.renren.com/GetSomeomeDoingList.do?userId=' + str(fid) + '&curpage=' + str(currPage)

        with closing(requests.get(url=statusLink, cookies=cookies)) as r:
            # parse json data
            statusJson = json.loads(r.text)

            # see if returned data contains status
            if('doingArray' in statusJson.keys()):
                # see if it's the last page of status
                if(len(statusJson['doingArray'])==0):
                    break
                else:
                    totalStatus +=len(statusJson['doingArray'])
                    # append to file with separator
                    with open('./status/%s.json' % (str(fid)+'-'+get_valid_filename(fName)), 'a+') as f:
                        f.write(r.text)
                        #split json objects for later processing.
                        f.write('########')

            elif('msg' in statusJson.keys()):
                print('msg is:',statusJson['msg'])
                break
            else:
                print('Something is wrong with user',fName)
                break

        #move to next status page
        currPage +=1

    fCount+=1
    print('\n{} has {} status!'.format(fName,str(totalStatus)))
    # sleep randomly to avoid being blocked by server
    time.sleep(round(random.random(),2))

print('In total:',fCount)
```

代码有以下需要注意的地方：

1. 每页会返回一个json object，脚本会把这些所有object存储在同一个文本文件里。这里使用8个`#`作为object之间的分隔符，方便后续文本处理。或者也可以把这写object存储在一个更大一级的object里方便直接做json parse。博主犯懒，直接用用分隔符分隔了。。。
2. 有一些有识之士（比如博主），早已注销了人人账户，这样我们就抓取不到他们的状态啦（不过其实万恶的人人网还是存储着这些数据的，不信你恢复一下账户，所有东西都在）。这时候页面会返回一个unicode的msg：该用户已注销。
3. 返回的json里面，会有一个叫做`count`的属性，显示了这个用户所有状态的数目。燃鹅，博主自己计数了用户的所状态数，发现居然和这个`count`不一样。。。而且有时候多，有时侯少。这导致博主一气之下去找了一个状态数比较少的好友去数了一下他的状态总数，发现还是我是对的，不知道人人网是怎么坑爹实现的。。。

运行了两个小时之后，在目录中生成了900多个，总共300多MB，包括了38万多条状态的文本文件。同时也发现，有100多名远见卓识之士已经注销了他们的账号。

我们来看一下存储的json的内容：

![statusParse](/img/2017-12-03/statusJson.png){: .center-image}

这个json object里面有几个属性，最重要的当然是这个`doingArray`。他存储了用户的status，其中大部分数据的作用根据命名可以很容易的理解。特殊情况是当这个状态是转自别人的时候，会多出`rootDoingId`,`rootContent`，`rootDoingUserName`，`rootDoingUserId`这几个属性来标识这个状态的最初出处和内容。

虽然我们现在有了所有好友的状态数据，但是文本文件并不适合分析。可以考虑把状态全部解析并且存入数据库，无奈目前还对此不熟，所以另做决定读取所有文本文件并存入一个列表变量。这个列表是这样设计的：

```python
record1 = {
    'uID':'fid',
    'name':name,
    'dTime':dt.datetime.strptime(status['dtime'], '%Y-%m-%d %H:%M:%S'),
    'msgID':id,
    'content':content,
    'comment':comment_count,
    'rootMsgID':rootDoingUserId, #optional
    'rootContent':rootContent, #optional
    'rootUID':rootDoingUserName #optional
};

statusList =[
record1, record2, record3, ..., recordN];
```

`statusList`是所有状态的容器列表，这个列表里包含很多个字典变量`record1-recordN`,每个字典变量对应某个用户的一条状态存储了一些必须的和可选的key-value pair。然后使用pickle模块将这个大list保存在磁盘上以备之后使用。因为38万条状态并不算多，所以这个list可以一次加载到内存中进行分析，在磁盘上占用的存储空间也很小，只有120多MB。可以使用下面的列表解析之前获得的所有文本文件并生成这个list：

```python
import json
import os
import pickle
import datetime as dt

fileList = os.listdir('./status')
print('There are in total %d file in the folder' % len(fileList))

statusList = []
numStatus = []

# traverse all .json files
for fileName in fileList:
    with open('./status/' + fileName, 'r') as f:
        rawJsons = f.read().split('########')
        # pop out the last empty string
        rawJsons.pop()
        print('There are %d pages of status in %s' % (len(rawJsons), fileName))

        numStatusPerPerson = 0
        # traverse all pages in a json file
        for page in rawJsons:
            pageJson = json.loads(page)

            doingArray = pageJson['doingArray']

            # traverse all status in a page
            for status in doingArray:
                # increase number of status
                numStatusPerPerson += 1

                # prepare an empty record
                record = {}

                # add field to record
                # compulsory fields
                record['uID'] = str(int(status['userId']))
                record['name'] = status['name']
                record['dTime'] = dt.datetime.strptime(status['dtime'], '%Y-%m-%d %H:%M:%S')
                record['msgID'] = str(int(status['id']))
                record['content'] = status['content']
                record['comment'] = status['comment_count']

                # optional fields
                if ('rootDoingUserId' in status.keys()):
                    record['rootUID'] = str(int(status['rootDoingUserId']))

                if ('rootContent' in status.keys()):
                    record['rootContent'] = status['rootContent']

                if ('rootDoingUserName' in status.keys()):
                    record['rootName'] = status['rootDoingUserName']

                # write record to list
                statusList.append(record)

        if (numStatusPerPerson != int(pageJson['count'])):
            print('Wrong page record! From page info: {}, actual read: {}\n'.format(int(pageJson['count']), numStatusPerPerson))
            # input('Debug...')

    numStatus.append(pageJson['count'])

print('The data base contains in total %d status!' % len(statusList))

with open('statusList.dat', 'wb') as f:
    pickle.dump(statusList, f)
```

有了状态数据之后，我们可以做一些非常简单的分析，更复杂深入的分析留在之后的post里再写。

# 状态总体分析
首先，我们看看谁话最多。。。前十名的状态总数是这样的:

![top10](/img/2017-12-03/top10.png){: .center-image}

最牛逼的一位大神在他的使用生涯中发了8225条状态。当然，可能更精确的的方式应该看发帖频率：

$$StatusRate = TotalNumOfStatus/DaysInBetween$$

>To be continued





