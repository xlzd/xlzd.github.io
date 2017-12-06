---
title: 如何将自己的程序发布到 PyPI
date: 2017-02-05 23:12:24
tags: Python
---

## 这段是废话

P.S. 这是一篇非常基础的文章，如果你有相关基础，请不必浪费时间阅读。写这篇文章的初衷是收到知友私信问到了怎么讲自己写的程序发布到 PyPI，与其回复一个人的私信，不如写出来供所有初学的人参考参考。
PyPI 的全称是「Python Package Index」，官方介绍如是说：

> The Python Package Index is a repository of software for the Python programming language. There are currently 102159 packages here. 

托管到 PyPI 的仓库，可以方便地通过 easy_install 或 pip 来安装和更新。比如，你直接「 pip install tornado 」就可以方便地安装 tornado 了。

概念性的东西，就一笔带过吧。这篇博客中，我将以发布一个名为「jujube_pill」的包到 PyPI 为例，从头到尾讲解如何将自己的程序发布到 PyPI。

<!-- more -->

## 代码结构

这里的示例代码结构非常简单，就一个 setup 文件和一个源码文件，结构如下：

```bash
jujube-pill $ tree
.
├── jujube_pill
│   └── __init__.py
└── setup.py
```

其中 setup.py 如下：

```Python
#!/usr/bin/env python
# coding: utf-8

from setuptools import setup

setup(
    name='jujube_pill',
    version='0.0.1',
    author='xlzd',
    author_email='what@the.f*ck',
    url='https://zhuanlan.zhihu.com/p/26159930',
    description=u'吃枣药丸',
    packages=['jujube_pill'],
    install_requires=[],
    entry_points={
        'console_scripts': [
            'jujube=jujube_pill:jujube',
            'pill=jujube_pill:pill'
        ]
    }
)
```

很多参数都见名之意，所以这里不赘述每个参数的含义。另外有一些参数对于初学者暂时用不上，也暂不表。 install_requires 是这个库所依赖的其它库，当别人使用 pip 等工具安装你的包时，会自动安装你所依赖的包。console_scripts 是这个包所提供的终端命令，比如我希望在安装这个包后可以使用「 jujube 」和「 pill 」两个命令，则按照 setup 文件的写法，当我在终端输入「 jujube 」的时候，将会执行 jujube_pill 包下（__init__ 中）的 jujube 函数。

__init__.py 文件如下：

```Python
#!/usr/bin/env python
# encoding=utf-8


def jujube():
    print u'吃枣'
 

def pill():
    print u'药丸'
```

## 上传代码到 PyPI

在上传之前，可以先通过命令校验 setup 写错了没有：

```bash
$ python setup.py check
running check
$ 
```

如果没有输出任何错误，则说明格式正确。

然后需要在这里注册一个 PyPI 的帐号，注册完成之后，就可以将这个代码库注册到 PyPI 了：

```bash
$ python setup.py register

running register
......

We need to know who you are, so please choose either:
 1. use your existing login,
 2. register as a new user,
 3. have the server generate a new password for you (and email it to you), or
 4. quit
Your selection [default 1]: 
1
Username: xlzd
Password: 
Registering jujube_pill to https://pypi.python.org/pypi
Server response (200): OK
I can store your PyPI login so future submissions will be faster.
(the login will be stored in /Users/xlzd/.pypirc)
Save your login (y/N)?y
```

中间一些步骤的输出被我省略了，其中是第一次上传代码到 PyPI，则需要先登录帐号。如果刚才没有在网页端注册帐号，在这里注册也是 OK 的。填好用户名密码之后，就可以登录了。登录成功后会提示你是否保存登录信息，如果选择了 y，则会在 home 目录下生成一个 .pypirc 文件存储你的 PyPI 帐号登录信息。

接着的操作是打包代码，使用如下命令：

```bash
$ python setup.py <打包格式>
```

打包格式一般使用「 sdist 」或者「 bdist_egg 」，使用前者居多（sdist 支持 pip 安装，bdist_egg 支持 easy_install 安装）。

打好包之后，通过如下命令上传：

```bash
$ python setup.py upload
```

最后去 PyPI 上看下我们刚刚上传的库（ [jujube_pill][1] ）：


其实，刚才的命令也可以合成一条一次执行：
```bash
python setup.py register sdist upload
```

## 试试看我们自己发布的库

```bash
$ pip install jujube_pill
```

安装完成后，就可以愉快地使用这两个命令了：

```bash
$ jujube
吃枣
$ pill
药丸
```

或者在代码中使用我们刚刚上传的库：

```python
In [1]: import jujube_pill

In [2]: jujube_pill.jujube()
吃枣

In [3]: jujube_pill.pill()
药丸
```

妥了。

 [1]: https://pypi.python.org/pypi/jujube_pill
