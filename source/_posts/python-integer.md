---
title: Python里的整数
date: 2015-11-04 21:13:00
tags: Python
---

在Python里试验一下下面的代码：
```Python
a = 1
b = 1
print a is b

a = -5
b = -5
print a is b

a = -6 
b = -6 
print a is b
```
输出的结果令人咋舌，竟然会是`True`、`True`、`False`，是什么原因导致这样的差异呢？


<!--more-->

 在Python源码中可以找到Python整数的定义（`Include/intobject.h`）：
```C
typedef struct {
    PyObject_HEAD
    long ob_ival;
} PyIntObject;
```
Python会将一部分常用的整数型对象缓存到内存中，以加快运行效率（空间换时间）。在Python源码中有如下代码（`Objects/intobject.c`）：

```C
struct _intblock {
    struct _intblock *next;
    PyIntObject objects[N_INTOBJECTS];
};
typedef struct _intblock PyIntBlock;
```
上面即为为整数对象分配内存的结构，整数对象即为上面的`PyIntObject`结构体。当在Python代码中为新的整数做赋值操作的时候，实际上就是使用了这个`PyIntBlock`结构体为一批整型变量申请了内存，剩下未使用的空间称为**待用的空闲整数对象**，在空闲整数对象赋值的过程中不需要分配内存空间，只有每个`PyIntObject`都有值之后才会再次申请内存，所以效率上会很高。每个`PyIntBlock`可存储的整数数量为N_INTOBJECTS，其定位为：
```C
#define BLOCK_SIZE      1000    /* 1K less typical malloc overhead */
#define BHEAD_SIZE      8       /* Enough for a 64-bit pointer */
#define N_INTOBJECTS    ((BLOCK_SIZE - BHEAD_SIZE) / sizeof(PyIntObject))
```
对于已经分配的整数对象，通过一个单向链表连接起来，在Python中，这个链表叫做`block_list`， 其定义在`Objects/intobject.c`中：
```C
static PyIntBlock *block_list = NULL;
```
整个过程大致是这样的：
![屏幕快照 2015-11-04 下午8.09.21.png][1]

![屏幕快照 2015-11-04 下午8.12.31.png][2]

Python中使用了一种特殊的结构将一部分整数提前缓存到了内存中，这些整数在`block_list`初始化的时候就分配了空间。由于程序会经常使用到这部分整数，这样的预处理可以加快程序运行时效率。这个结构叫做`small_ints`，定义在`Include/intobject.h`中：
```C
#ifndef NSMALLPOSINTS
#define NSMALLPOSINTS           257
#endif
#ifndef NSMALLNEGINTS
#define NSMALLNEGINTS           5
#endif
static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSINTS];
```
这个结构保存了`-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS`之间也即是`[-5, 257)`之间的整数。如图：

![屏幕快照 2015-11-04 下午8.12.31.png][3]

如上，值为-5的整数位于small_ints中的第一位，就是说-5的偏移量为0，对应的，-4的偏移量为1，-3的偏移量为2.......

那么，当我们在运行`num = -5`的时候，实际上在运行什么？
在Python的实现机制中，上面的代码我们将会调用`PyInt_FromLong`函数，大概的逻辑是这样的：
```Python
def PyInt_FromLong(n):
    if n in [-5, 257):
        return small_ints[n + 5]
    if 没有剩余的PyIntBlock空间：
        申请内存，分配PyIntBlock
    在PyIntBlock中新建整型对象
    return 整型对象
```
这段逻辑对应的C代码如下（Python2.7.10）：
```C
PyObject *
PyInt_FromLong(long ival)
{
    register PyIntObject *v;
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) {
        v = small_ints[ival + NSMALLNEGINTS];
        Py_INCREF(v);
#ifdef COUNT_ALLOCS
        if (ival >= 0)
            quick_int_allocs++;
        else
            quick_neg_int_allocs++;
#endif
        return (PyObject *) v;
    }
#endif
    if (free_list == NULL) {
        if ((free_list = fill_free_list()) == NULL)
            return NULL;
    }
    /* Inline PyObject_New */
    v = free_list;
    free_list = (PyIntObject *)Py_TYPE(v);
    PyObject_INIT(v, &PyInt_Type);
    v->ob_ival = ival;
    return (PyObject *) v;
}
```
执行`a = -5`时，-5在small_ints范围`[-5, 257)`内，于是会返回第`-5 + 5 = 0`个缓存好的对象(的指针)。同理，对于开头的`a = -6`，由于-6没有处于small_ints范围之间，就需要在PyIntBlock中分配一个新的PyIntObject对象，为其赋值为-6，并返回其指针。

于是，开头的Python代码结果为`True`、`True`、`False`，就变得容易理解了。

----------

如上~~~


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/11/04/1263232205.png
  [2]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/11/04/2980139042.png
  [3]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/11/04/1234567890.png
