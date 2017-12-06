---
title: 优雅的 Python 之 Ellipsis
date: 2016-05-30 09:41:55
tags: Python
---

Python 是一门非常具有包容性的语气，体现在一个优秀的工程师可以非常容易优雅高效地完成一件事情，而一个拙略的工程师通过<del>屎</del>一样的代码同样可以做到几乎一样的功能。今天，介绍一下 Python 的 Ellipsis\~\~\~

想象这样一个问题：
 > 如何优雅地生成一个等差数组？比如输入一个序列的第一、第二项以及最后一项，然后返回这个等差数组。


<!--more-->


这里指的优雅并不是实现代码上，而是调用方式优雅。那么，在具体实现之前，我们先看一眼调用的方式吧：

```Python
In [1]: gen = SeqGenerator()

In [2]: gen[1, 2, ..., 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]

In [3]: gens[20, 16, ..., 0]
[20, 16, 12, 8, 4, 0]
```

有意思吧，这样直观的方式调用，简单明了。下面简单聊聊实现原理吧。

其实，在 `object` 对象中，有一个 `__getitem__` 方法。你可以做如下测试：

```Python
In [1]: class Test(object):
   ....:     def __getitem__(self, item):
   ....:         print item

In [2]: t = Test()

In [2]: t[1]
1

In [3]: t['what the fuck']
what the fuck

In [4]: t[:]
slice(None, None, None)

In [5]: t[1: 10]
slice(1, 10, None)

In [6]: t[1:3:5]
slice(1, 3, 5)

In [7]: t[1, 2]
(1, 2)

In [8]: t[1, 2, ..., 5]
(1, 2, Ellipsis, 5)
```

可以看到，第八行的调用方式与上面产生等差数列的方式基本是一样的了，但是返回的内容( `Test` 中事实上并没有返回，只是直接 `print` 了)不同，多了一个 `Ellipsis`。

那么，对应上面的 `SeqGenerator` 的实现就一目了然了：

```Python
class SeqGenerator(object):
    def __getitem__(self, item):
        if not (isinstance(item, tuple) and len(item) == 4 and item[2] is Ellipsis):
            raise RuntimeError('what the fuck')
        return range(item[0], (item[-1]+1, item[-1]-1)[item[0]>item[1]], item[1]-item[0])
```

然后就可以愉快地按照上面的示例一样调用啦。当然，上面的实现比较简单，没有完整地考虑到各种情况，如果你愿意，可以自行解决之\~\~\~
