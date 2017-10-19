---
title: webpy服务器（Linux+Web+HTML）
date: 2017-02-17 11:38:51
categories: [网页设计, Web服务器]
tags:
---
# 前言：
--------------
刚开始在慕课网上面看到了一个webpy的视频，正好自己有个服务器，想搭个网站玩玩，然后webpy比较简单，所以拿这个动手了，以后可能去用别的库去做网页。

网页也是看了一下午的HTML的标签什么的，CSS还没学，随手写了个简单的，带个 POST 请求。


# Web程序：
--------------------
## URL解析：
---------------
首先在程序开头放上这样一段代码：
```python
urls = ('/','index',
        '/(\d+)', 'read',
        '/(.*)', 'other')
app = web.application(urls, globals())
```
这样用来匹配访问url，要去执行那个类。

匹配方式，分**精确匹配**，**模糊匹配**两种，模糊匹配可以实现带组匹配，即上面加的那一个括号，这样会把括号中匹配的数据以 **name** 变量传递给类的方法。

## 实现响应：
--------------------
每个url匹配后对应一个类，在类里面实现网页中请求的方法就好了，比如 **GET**，**POST** 。

```python
class index:

    def __init__(self):
        self.reader = read()

    def GET(self):
        with open('html/index.html', 'rb') as f:
            html = f.read()
        return html

    def POST(self):
        data = web.data()
        data = urllib.unquote_plus(data)
        with open('temp ', 'wb') as f:
            f.write(data)


class read:

    def GET(self, name):
        print name

```
## 启动Web程序：
---------------
因为是在 Linux 服务器上运行的，直接输入一下代码：
```shell
nohup python test.py 45.76.206.109:80 &
```
记得 ip 后面带上端口号。

# HTML网页：
---------------
## from标签：
--------------
这里的 action 大概是指用 method 里面的方法，访问了** [主域名] + [action]**。
例如下面的，相当于 **POST** 了 **acmfish.top/**，然后根据 URL 匹配，调用了 **index** 类的 **POST** 方法。
```html
<form method="post" action="/">
	<div align="center">
		<label>PID:</label>
		<input type="text" name="PID">
		<input type="submit" value="Submit">
	</div>
</form>
```

# 全部代码：
--------------
## Web程序：
------------------
```python
import web

urls = ('/','index',
        '/(\d+)', 'read',
        '/(.*)', 'other')
app = web.application(urls, globals())

class index:

    def __init__(self):
        self.reader = read()

    def GET(self):
        with open('html/index.html', 'rb') as f:
            html = f.read()
        return html

    def POST(self):
        data = web.data()
        return self.reader.GET(data[4:])

class read:

    def __init__(self):
        self.head = '''<html>
<head>
        <meta charset="utf-8"/>
        <title>Code</title>
</head>
<body>
<pre>'''
        self.tail = '''</pre>
</body>
</html>'''

    def GET(self, name):
        path = 'code/'+name
        try:
            with open(path, 'rb') as f:
                text = f.read()
                return text
        except IOError:
            return 'File not find'

class other:
    def GET(self, name):
        return 'Not find'

if __name__ == "__main__":
    app.run()

```
## HTML：
-----------------
```html
<html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	
	<title>Get Code</title>
	<style>
		a{
		color:#40668c;
		}
		label{
		color:#000000;
		}
		input{
		color:#000000;
		}
	</style>
</head>
<body>
<h1 align="center">Get Code</h1> <hr color="#000000">
<br><br><br>
<h2 align="center">请直接访问<a href="/1000">45.76.206.109/1000</a>以获取PID为1000题目代码。</h2>
<h2 align="center">或</h2>
<h2 align="center">快捷搜索</h2>
<form method="post" action="/">
	<div align="center">
		<label>PID:</label>
		<input type="text" name="PID">
		<input type="submit" value="Submit">
	</div>
</form>


</body></html>

```