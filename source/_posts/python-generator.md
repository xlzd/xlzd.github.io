---
title: "Python: generator与yield"
date: 2015-08-24 03:21:44
tags: Python
---

第一步，我们想要生成fibonacci数列前N项，我们可以这样做：

```Python
def fib(size):
    a, b = 0, 1
    while size:
        print a, 
        a, b = b, a + b
        size -= 1
```

执行可以得到输出如下：
```Python
In [1]: fib(10)
0 1 1 2 3 5 8 13 21 34
```
这个函数的问题在于，我们只能调用它输出结果，并没办法拿到返回值，于是通用性不够。现在对这个函数做一点修改如下：


<!--more-->

```Python
def fib(size):
    a, b = 0, 1
    lst = []
    while size:
        lst.append(a)
        a, b = b, a + b
        size -= 1
    return lst
```

现在再次调用：
```Python
In [15]: fib(10)
Out[15]: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```
现在成功拿到fibonacci数列的前N项了，但是还有一个问题就是，当传入参数size过大的时候，待返回的list可能会因为内存不够而撑爆。如果要控制内存为O(1)，一个做法便是使用可迭代（iterable）的对象。Python中的`list|set|str|tuple|dict|file`都是`iterable`的， 要自己实现一个`iterable`对象，方法之一便是定义class的`__iter__()`方法，在`__iter__()`方法中返回一个迭代器(iterator)即可。迭代器(iterator)可以看做一个数据流，重复调用它的`next()`方法可以依次取出这个数据流中的每一个元素，当没有更多元素的时候则抛出`StopIteration`。
下面是Python文档中对iterable和iterator的描述：

> **iterable**
An object capable of returning its members one at a time. Examples of iterables include all sequence types (such as list, str, and tuple) and some non-sequence types like dict and file and objects of any classes you define with an __iter__() or __getitem__() method. Iterables can be used in a for loop and in many other places where a sequence is needed (zip(), map(), ...). When an iterable object is passed as an argument to the built-in function iter(), it returns an iterator for the object. This iterator is good for one pass over the set of values. When using iterables, it is usually not necessary to call iter() or deal with iterator objects yourself. The for statement does that automatically for you, creating a temporary unnamed variable to hold the iterator for the duration of the loop. 

> **iterator**
An object representing a stream of data. Repeated calls to the iterator’s next() method return successive items in the stream. When no more data are available a StopIteration exception is raised instead. At this point, the iterator object is exhausted and any further calls to its next() method just raise StopIteration again. Iterators are required to have an __iter__() method that returns the iterator object itself so every iterator is also iterable and may be used in most places where other iterables are accepted. One notable exception is code which attempts multiple iteration passes. A container object (such as a list) produces a fresh new iterator each time you pass it to the iter() function or use it in a for loop. Attempting this with an iterator will just return the same exhausted iterator object used in the previous iteration pass, making it appear like an empty container.

所以上面的代码又可以改为如下：

```Python
class Fib(object): 
    def __init__(self, size): 
        self.size = size
        self.a, self.b = 0, 1 
    def __iter__(self): 
        return self 
    def next(self): 
        if self.size: 
            now = self.a
            self.a, self.b = self.b, self.a + self.b 
            self.size -= 1
            return now
        raise StopIteration()
```

现在的调用结果如下：
```Python
In [1]: for x in Fib(10):
   ....:     print x,
   ....:     
0 1 1 2 3 5 8 13 21 34
```
至此，我们可以拿到fibonacci的前N项，又不用担心过大的内存开销。但是我们也会发现，上面的代码相比之前的函数冗杂了很多。我们希望兼顾简洁的风格和iterable的优势，就用到了Python的`yield`：

```Python
def fib(size):
    a, b = 0, 1
    while size:
        yield a
        a, b = b, a + b
        size -= 1
```
调用如下：
```Python
In [1]: for x in fib(10):
   ....:     print x,
   ....:     
0 1 1 2 3 5 8 13 21 34
```
通俗的理解就是，`yield`把一个普通的函数变成了一个生成器（generator），对`fib`函数的调用将会返回一个iterable对象，在对其在for循环中迭代时，每次执行到yield的时候，就返回一个值。到下一次循环，代码从`yield`的下一句开始继续执行。

现在我们手动执行一次，就明白了：
```Python
In [1]: f = fib(10)

In [2]: type(f)
Out[2]: generator
 
In [3]: f.next()
Out[3]: 0

In [4]: f.next()
Out[4]: 1

In [5]: f.next()
Out[5]: 1

In [6]: f.next()
Out[6]: 2

In [7]: f.next()
Out[7]: 3

In [8]: f.next()
Out[8]: 5

In [9]: f.next()
Out[9]: 8

In [10]: f.next()
Out[10]: 13

In [11]: f.next()
Out[11]: 21

In [12]: f.next()
Out[12]: 34

In [13]: f.next()
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-26-c3e65e5362fb> in <module>()
----> 1 f.next()

StopIteration:
```

当函数执行结束以后，就会抛出一个StopIteration，for循环会根据StopIteration判断循环结束。

值得注意的是，`fib`是一个`generator function(生成器函数)`，而`fib(10)`则是一个`generator(生成器)`。

----------

另一个概念叫做`generator expression(生成器表达式)`：
我们知道在Python中可以通过列表推导式创建一个List：
```Python
In [1]: [x for x in xrange(10)]
Out[1]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
这样的做法同样可能因为列表过大导致内存占用过多甚至溢出的问题，生成器表达式的作用在于生成一个generator，然后在我们需要的时候（迭代到的时候）再计算生成需要的值：

```Python
In [1]: (x for x in xrange(100))
Out[1]: <generator object <genexpr> at 0x7fe221ad29b0>
```
生成器表达式和列表推导一样简单灵活又强大，这里不过多赘述。

----------
## 总结 ##
关于`generator expression`、`generator function`、`generator`、`iterator`、`iterable`之间的关系，下图阐释的非常清除了（图片摘自一个国外博客）。

![~~~][1]


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/08/24/relationships.png