---
title: 通过微博 API 和 Pushbullet 准实时关注你的心上人
date: 2017-06-01 01:56:56
tags: Python
---

名字取得有点标题党了，就在刚刚，闲的无聊，通过微博接口和 Pushbullet 接口做了一个近实时关注别人的小工具。
具体原理比较简单，就是不断轮询微博接口，发现有新的微博的时候，通过 Pushbullet 的接口推送消息到手机和电脑。

[Pushbullet][pushbullet] 是一个跨平台的消息推送工具，可以很方便将消息在各端间传递，同时也提供了 API 接口供通过程序调用。PYPI 上有一个 [pushbullet.py][pypi-pushbullet] 的库对它的 API 做了封建，可以更简单方便地使用，这里用到的功能比较简单：

```Python
from pushbullet import Pushbullet

pb = Pushbullet(API_KEY)
pb.push_note(title, body)
```

这样可以将消息发送到 API_KEY 对应帐号登录的所有设备，API_KEY 通过登录后如下截图中页面的「Create Access Token」创建。

![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/create-access-token.jpg)

<!-- more -->

而关注别人发送新微博则可以通过微博的 [API 接口][weibo-api]。很久很久以前为了做一个微博报时机器人曾经申请过一个微博应用，这里刚好用上了。如果你没有应用，创建一个就好啦。

![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/weibo.jpg)

[weibo][pypi-weibo] 是一个对微博接口的封装库，大概使用如下：

```Python
from weibo import Client

APP_KEY = '*****'
APP_SECRET = '******'
CALLBACK_URL = 'https://api.weibo.com/oauth2/default.html'
AUTH_URL = 'https://api.weibo.com/oauth2/authorize'
USERID = '******'
PASSWD = '*******'

client = Client(APP_KEY, APP_SECRET, CALLBACK_URL, username=USERID, password=PASSWD)
data = client.get('API_PATH')
```

![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/weibo2.jpg)

由于微博接口限制，无法再直接获取到非授权用户的微博列表，由于我刚好微博并没有关注任何人，所以关注需要关注动态的人后，获取自己的微博列表拿到的刚好就是这个人时间序排列的前几条微博。说到这里刚好还能给微博提一个 bug，大概是由于缓存不同步吧，我明明已经删除了所有的微博（只剩一条）和取关了所有人，个人主页上显示的数字不正确：
![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/weibo2.jpg)

如果你关注了很多人，一个解决方案是在获取自己微博列表的时候通过 [user_timeline 接口][weibo-api2]的 since_id 和 count 字段多获取一点，每次获取都记录下上次的最后一条的位置，下次从最新数据取到上次取过的地方为止，然后 filter 过滤掉其它人的微博。
接下来的事情，就是写一个 crontab，隔段时间读一次微博列表，判断如果有新的微博，就通过 Pushbullet 的接口推消息给自己。这个接口返回的数据还有微博 po 主的个人资料，所以顺便还能监测目标用户微博资料的变更。

这里 crontab 的间隔时间，根据微博接口[访问频次权限][weibo-api3]，设置为每分钟一次是没问题的~

最后的效果是这样的：

![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/result.jpg)
![create-access-token](http://7xkpi6.com1.z0.glb.clouddn.com/blog/2017/06/01/result2.jpg)


有点晚了，写的比较马虎，完整的代码在这里：[weibo monitor - Github Gist][gist]，仅供参考。


[pushbullet]: https://www.pushbullet.com/
[pypi-pushbullet]: https://pypi.python.org/pypi/pushbullet.py
[pypi-weibo]: https://pypi.python.org/pypi/weibo
[weibo-api]: http://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI
[weibo-api2]: http://open.weibo.com/wiki/2/statuses/user_timeline
[weibo-api3]: http://open.weibo.com/wiki/%E6%8E%A5%E5%8F%A3%E8%AE%BF%E9%97%AE%E9%A2%91%E6%AC%A1%E6%9D%83%E9%99%90-
[gist]: https://gist.github.com/xlzd/01b8b8e1909ae0f601c85e142f2bd15b
