---
title: Python requests库介绍
date: 2016-01-01 17:06:00
tags:
  - Python
  - Crawler
---

最近有几个知乎上的朋友私信问到关于爬虫爬取数据的时候总是出现这样或者那样的问题，这里介绍一个Python HTTP库： requests。Requests是一个基于Apache2协议开源的Python HTTP库，号称是“为人类准备的HTTP库”。

Python中，系统自带的`urllib`和`urllib2`都提供了功能强大的HTTP支持，但是API接口确实太难用了。requests作为更高一层的封装，确实在大部分情况下对得起它的slogan——HTTP for Humans。

### 发送请求

废话少说，先看看requests的简单使用吧：
```Python
In [1]: import requests  
In [2]: resp = requests.get('http://xlzd.me')

In [3]: resp.status_code
Out[3]: 200
```


<!--more-->


发送一个完整的HTTP请求，只需要一句代码即可。发送其它方式的请求同样如此简洁：
```Python
In [1]: r = requests.post("http://xlzd.me/login", data = {"user":"xlzd", "pass": "mypassword"})
In [2]: r = requests.put("http://xlzd.me/post", data = {"title":"article"})
In [3]: r = requests.delete("http://xlzd.me/foo")
In [4]: r = requests.head("http://xlzd.me/bar")
In [5]: r = requests.options("http://xlzd.me/abc")
```


### 解析URL中的参数
在GET请求的时候，经常会有很多查询参数接在URL后面，形如`http://xlzd.me/query?name=xlzd&&lang=python`，在拼接URL的时候常常容易出现拼接错误的情况，对此你可以使用如下代码让这个过程变得简洁明了：
```Python
In [1]: r = requests.get("http://xlzd.me/query", params={"name":"xlzd", "lang": "python"})

In [2]: print r.url
Out[2]: http://xlzd.me/query?name=xlzd&&lang=python
```
如上，字典中的参数会被requests自动解析并且正确连接到URL中。


### 响应内容
如上的代码拿到的是一个HTTP response，那么怎样取出里面的内容呢？
```Python
In [7]: r = requests.get('http://xlzd.me')

In [8]: r.encoding
Out[8]: 'UTF-8'

In [9]: r.headers
Out[9]: {'Content-Encoding': 'gzip', 'Transfer-Encoding': 'chunked', 'Vary': 'Accept-Encoding', 'Server': 'nginx', 'Connection': 'keep-alive', 'Date': 'Fri, 11 Dec 2015 06:42:31 GMT', 'Content-Type': 'text/html; charset=UTF-8', 'X-Pingback': 'http://xlzd.me/action/xmlrpc'}

In [10]: r.cookies
Out[10]: <RequestsCookieJar[]>

In [11]: r.text
Out[11]: u'<!DOCTYPE HTML>\n<html lang="zh-CN">\n\t<hea......
```
requests会自动对响应内容编码，所以就可以通过`r.text`取出响应文本了。对于别等响应内容（文件、图片、...），则可以通过`r.content`取出来。对于json内容，也可以通过`r.json()`来取。


### 自定义Headers
对于写爬虫来讲，模拟浏览器是发请求的时候做的最多的事情了，最常见的模拟浏览器无非就是伪装headers：
```Python
In [23]: url = 'http://xlzd.me'

In [24]: headers = {'User-Agent': 'my custom user agent', 'Cookie': 'haha'}

In [25]: requests.get(url, headers=headers)
```

### 重定向和超时
requests之所以称为“HTTP for human”，因为其封装层次很高，其中一处体现就在：requests会自动处理服务器响应的重定向。我在做搜狗微信公众号抓取的时候，搜狗搜索列表页面的公众号文章地址，其实不是微信的地址而需要请求到搜狗到服务器做重定向，而requests的默认处理则是将整个过程全部搞定，对此可以这样：
```Python
In [1]: r = requests.get('http://xlzd.me', allow_redirects=False)
```
`allow_redirects`参数为`False`则表示不会主动重定向。


另外，有时候对方网站的响应时间太长了，我们希望在指定时间内完事，或者直接停止这个请求，这时候的做法是：
```Python
In [1]: r = requests.get('http://xlzd.me', timeout＝3)
```
`timeout`表示这次请求最长我最长只等待多少**秒**，


### 代理
为requests套上一层代理的做法也非常简单：
```Python
import requests

proxies = {
  "http": "http://192.168.31.1:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://xlzd.me", proxies=proxies)
```


### 会话对象
服务器端通过session来区分不同的用户请求（浏览器会话），requests的会话对象是用来模拟这样的操作的，比如可以跨请求保持某些参数：就像你在访问微博的时候，不需要每次翻页都重新登录一次。

```Python
session = requests.Session()
session.post('http://xlzd.me/login', data={'user': 'xlzd', 'pass': 'mypassword'})
# 登录成功则可以发布文章了
session.put('http://xlzd.me/new', data={'title': 'title of article', 'data': 'content'})
```
另外，`requests.Session`对象也可以直接使用上面的所有方法哟。


### 小结
requests弥补了Python自带的urllib和urllib2的不好使用，通过使用requests，可以更方便、更直观的处理网络请求相关的代码，在网络爬虫的编写过程中更加得心应手。
