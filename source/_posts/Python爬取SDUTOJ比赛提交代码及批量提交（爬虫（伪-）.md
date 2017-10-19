---
title: Python爬取SDUTOJ比赛提交代码及批量提交（爬虫（伪)）
date: 2017-01-03 22:53:22
categories: [爬虫&数据处理]
tags:
---
# 需求：

 - 把自己之前在contest里面的代码提取出来。
 - 实现批量提交contest和problem里面的题目。
 - 


----------
# 过程：
总共大概花了4个小时，一晚上，一个类一个文件的方法写起来真的爽，一晚上没停住手。
自己首先写的是下载器，首先明确需求。

 - 可以模拟登陆。
 - 可以post请求。
 - 可以下载网页。
 - 为了省事，把提交题目的功能也整合里面了。

代码如下，实现起来没啥困难，毕竟已经是轻车熟路了。

```python
# coding:utf8
import  urllib2
import cookielib
import urllib

class Down(object):

    def __init__(self):
        #初始化常用网址
        self.login_url = 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Login/login'
        self.home_url = 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home'
        self.submit_url = 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Solution/submitsolution/pid/1110.html'

        #安装cookie处理
        cj = cookielib.CookieJar()
        cookie_support = urllib2.HTTPCookieProcessor(cj)
        opener = urllib2.build_opener(cookie_support)
        urllib2.install_opener(opener)

        #初始化cookie
        home_temp = urllib2.urlopen(self.home_url)

        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36',
            'Referer': 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Login/login',
            'Host': 'acm.sdut.edu.cn'}

        self.login()

    def post(self, url, post_data = None):
        #下载网页和POST请求
        if post_data is None:
            request = urllib2.Request(url, headers = self.headers)
            response = urllib2.urlopen(request)
            return response.read()
        else:
            post_data = urllib.urlencode(post_data)
            request = urllib2.Request(url, post_data, self.headers)
            response = urllib2.urlopen(request)
            return  response

    def login(self, user = 'Fish', password = '123456'):
        #登陆信息
        post_data = {'user_name': user, 'password':password }
        self.post(self.login_url, post_data)

    def submit(self, pid, code, language = 'g++', cid = None):
        if cid == None:
            post_data = {'pid': pid, 'code': code, 'lang' : language}
            self.post(self.submit_url, post_data)


if __name__ == '__main__':
    test = Down()
    print test.post('http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Solution/submitsolution/pid/1110.html')
```

 然后是网页分析器，这里我由内到外的写的，首先需求是给定一个贴着代码的网页，把代码爬下来。
 下一步需求是给定一个status，把AC的代码记录的链接爬下来发给上一步的爬取代码的函数。
 最后是给定一个比赛的cid，找到ranklist链接并且找到自己的status页面发给上一步实现的处理。

这样做的好处就是每一步都可以单独调试，每一个函数写起来，它需要调用的函数已经写完调试好了。

这里麻烦点的就是status页面的处理，最后返回数据的时候我按（代码，题号）的二元组构成的列表返回的。

```python
# coding:utf8
from bs4 import BeautifulSoup
import html5lib
import Down
import time

class ReadCode(object):

    def __init__(self):
        self.data = []
        self.downer = Down.Down()

    def read_code(self, html):
        soup = BeautifulSoup(html, "html5lib")
        time.sleep(0.01)
        return soup.find('pre').text

    def read_status(self, html):
        soup = BeautifulSoup(html, "html5lib")
        table = soup.find_all('tr')

        tdata = []

        #读取每一个提交信息
        for tr in table:
            try:
                tr = tr.find_all('td')
                if tr[3].text == "Accepted" and (tr[6].text == "g++" or tr[6].text == "gcc"):
                    temp =  'http://acm.sdut.edu.cn/' + tr[6].a['href'], tr[2].a['href'][-9:-5]
                    print temp[1]
                    tdata.append(temp)
            except:
                pass

        #读取代码
        for temp in tdata:
            html = self.downer.post(temp[0])
            ans = self.read_code(html)
            self.data.append((ans,temp[1]))

        #读取下一页
        try:
            url = 'http://acm.sdut.edu.cn/' + soup.find(class_ = 'next')['href']
            html = self.downer.post(url)
            self.read_status(html)
        except:
            pass

    def read_rank(self, cid):
        html = self.downer.post('http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Contest/contestranklist/cid/'+str(cid))
        soup = BeautifulSoup(html, "html5lib")

        #找到个人的status页面并读取
        try:
            print cid
            url = 'http://acm.sdut.edu.cn' + soup.find(text = "jk160801李东庆").parent.parent.find_all('td')[2].a['href']
            html = self.downer.post(url)
            self.read_status(html)
        except:
            pass

if __name__ == '__main__':
    test = ReadCode()
    print test.data
```

最后是一个Main调度一下，顺便实现了读写到文件的功能。
（这里有个玄学的小细节，代码中也备注了）

```python
# coding:utf8
import Down
import ReadCode
import  urllib2
import cookielib
import urllib

#改下输出流的编码？玄学
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )

class Main(object):

    def __init__(self):
        self.data = []
        self.reader = ReadCode.ReadCode()
        self.cid = 0

    def read_contest(self, cid):
        self.cid = cid
        self.reader.read_rank(cid)
        self.data += self.reader.data
        self.reader.data = []

    def output(self):
        for temp in self.data:
            f = open('code/'+temp[1], 'w')
            f.write(temp[0])
            print self.cid, temp[1]
        self.data = []

if __name__ == '__main__':
    oj = Main()
    for cid in range (1828, 1976):
        oj.read_contest(cid)
        oj.output()
```

最后实现批量提交的时候自己又写了一下提交的函数，之前下载器里面的太弱了，当然这里也调用了下载器，比赛提交的代码的时候是暴力了点...
（当然这个提交器是利用了之前爬下来的代码）

```python
# coding:utf8
import Down
import time

class Submit(object):
    def __init__(self):
        self.downer = Down.Down()

    def submit_problem(self, pid ,url = 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Solution/submitsolution/pid/1000.html', cid = 233):
        try:
            f = open('code/'+str(pid), "r")
            code = f.read()
            post_data = {'pid':pid, 'lang':'g++', 'code':code}
            self.downer.post(url, post_data)
            post_data = {'pid': pid, 'lang': 'g++', 'code': code, 'cid':cid}
            self.downer.post(url, post_data)

            print cid,pid
            time.sleep(0.01)
        except:
            pass

    def submit_contest(self, cid):
        url = 'http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Contest/contestsubmit/cid/' + str(cid) + '/pid/'
        for pid in range(1000, 3764):
            try:
                self.submit_problem(pid, url+ str(pid) + '.html', cid = cid)
            except:
                pass

if __name__ == '__main__':
    submiter = Submit()
    for cid in range(1875, 1881):
        submiter.submit_contest(cid)
```
#效果：
![这里写图片描述](http://img.blog.csdn.net/20170103225156166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![这里写图片描述](http://img.blog.csdn.net/20170103225322323?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

大概就是爬下来了200+个自己的代码，然后给小号刷了一下，一键AC（笑