---
title: 从源码编译 Python
date: 2016-08-22 21:55:54
tags: Python
---

尝试通过源码自己编译 Python，使用的系统是 Ubuntu14.04 LTS。

![python-back.png][1]

<!--more-->

首先去官网下载源码，地址：[源码下载][2]。下载完成之后，解压源码：
```
tar -zxvf Python-2.7.12.tgz
```

可以看到目录结构如下：

```
.
├── aclocal.m4
├── config.guess
├── config.sub
├── configure
├── configure.ac
├── Demo
├── Doc
├── Grammar
├── Include
├── install-sh
├── Lib
├── LICENSE
├── Mac
├── Makefile.pre.in
├── Misc
├── Modules
├── Objects
├── Parser
├── PC
├── PCbuild
├── pyconfig.h.in
├── Python
├── README
├── RISCOS
├── setup.py
└── Tools
```

其中，我们比较关注的几个目录是：
 - Include： 这个目录包括了 Python 的所有头文件。
 - Lib：这里是 Python 标准库，都是用 Python 实现的。
 - Modules：用 C 语言编写的模块，比如 cStringIO / tkinter 等。
 - Objects：Python 内建对象，如 int / list 等。
 - Python：Python 解释器的 Compiler 和执行引擎。
 - Parser：Python 解释器的 Scanner 和 Parser。


我并不只是想尝试简单的通过源码编译安装，那么，在编译之前，我们先对它做一点小小的改动吧。今天先不做太复杂的事情，尝试一下“颠倒黑白”吧。所谓颠倒黑白，就是在输出（只有输出时）bool 型变量时，将 True/False 对调。关于输出 bool 变量的 C 语言实现，在 Objects/boolobject.c 的第 7-14 行，如下：

```C
 static int
bool_print(PyBoolObject *self, FILE *fp, int flags)
{
    Py_BEGIN_ALLOW_THREADS
    fputs(self->ob_ival == 0 ? "False" : "True", fp);
    Py_END_ALLOW_THREADS
    return 0;
} 
```

可以看出，对于输出 True 还是 False 的判断是用三元运算符 `self->ob_ival == 0 ? "False" : "True"`，那么，其实改动就非常容易了：

```
fputs(self->ob_ival != 0 ? "False" : "True", fp);
```

将比较运算符做一点小改动，就“颠倒黑白”啦。然后执行：
```
./configure --prefix=/path/u/what/to/install
make
make install
```

第一条命令的 `--prefix=` 后面是你想要安装的位置，你可以自行调整。等待运行完毕，就安装好啦，进入指定的目录，目录结构如下：

```
.
├── bin
├── include
├── lib
└── share
```

想要运行的话，执行 `bin/python` 即可，你也可以将其加入到 PATH 中，不过还是不建议去搞乱系统那个。好了，用我们自己编译的解释器执行几条语句吧：

```
>>> print True
False

>>> print False
True

>>> print 3 > 5       
True

>>> print 1 == 2
True
```

很明显，已经“颠倒黑白”啦。


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/07/08/561824476.png
  [2]: https://www.python.org/downloads/source/