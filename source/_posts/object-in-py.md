---
title: Python 中的对象概述
date: 2016-08-29 21:58:19
tags: Python
---

在 Python 的世界中，一切皆对象。 int / list / dict / ... 都是对象，除此之外，函数、类本身也是对象，那么，这些对象究竟是什么呢？

从结果看，Python 中的对象是 C 语言中结构体在堆上申请的一片内存区域。而在具体实现上，这里先简单描述一下。
![obj][1]

<!--more-->


---

## 万物基于 MIUI： PyObject

在 Python 中，所有对象都共有一些特性，这些特性定义在 `PyObject` 中。`PyObject` 定义在 `Include/object.h` 中：

```C
#define PyObject_HEAD                   \
    _PyObject_HEAD_EXTRA                \
    Py_ssize_t ob_refcnt;               \
    struct _typeobject *ob_type;

typedef struct _object {
    PyObject_HEAD
} PyObject;
```

简化后即为：
```C
typedef struct _object {
    int ob_refcnt;               
    struct _typeobject *ob_type;
} PyObject;
```

在 PyObject 中，`ob_refcnt` 用以记录对象的引用数（与引用计数的内存回收相关，这里暂且不表），当有新的指针指向某对象时，`ob_refcnt` 的值加 1， 当指向某对象的指针删除时，`ob_refcnt` 的值减 1，当其值为零的时候，则可以将该对象从堆中删除（事实上并不会立即删除，这里暂且不表）。除了 `ob_refcnt` 之外，还有一个 指向 `_typeobject` 指针 `ob_type`。这个结构体用于表示对象类型。跳过 `_typeobject`，可以发现， Python 对象的核心在于一个引用计数和一个类型信息。

`PyObject` 定义的内容会出现在每个对象所占内存的开始部分。

---

## 定长对象与变长对象

在 Python 中，除了 `bool` `float` 这样的定长对象（一旦确定下来需要的内存，便不再有改动），还有另外一种对象：长度可变的对象。这种对象在 Python 的实现中通过 `PyVarObject` 结构体来表示：

```
#define PyObject_VAR_HEAD               \
    PyObject_HEAD                       \
    Py_ssize_t ob_size; /* Number of items in variable part */

typedef struct {
    PyObject_VAR_HEAD
} PyVarObject;
```

事实上，就是在 `PyObject` 的基础上，多了一个 `ob_size` 变量，用以标识对象的长度（**是长度，不是内存占用**）。也就是说，其实 `PyVarObject` 就是 `PyObject` 的一个拓展，于是，**在 Python 中，所有的对象都可以通过 `PyObject *` 指针来引用**，这一点非常重要，它使得很多操作变得统一（这篇博客暂不详述）。

由此，Python 中所有对象在实现的时候，内存无非如下两种情况：

```
   定长对象              变长对象
+-----------+       +-----------+
| ob_refcnt |       | ob_refcnt |
+-----------+       +-----------+
|  ob_type  |       |  ob_type  |
+-----------+       +-----------+
|           |       |  ob_size  |
|           |       +-----------+
|   other   |       |           |
|           |       |   other   |
|           |       |           |
+-----------+       +-----------+
```


---

## 道生一：PyTypeObject

在描述 `PyObject` 的时候，提到了一个 `_typeobject` 结构体。那么，它是干什么的呢？想象一下，一个对象在创建的时候需要多少内存、这个对象的类名是什么等等信息，又是如何记录和区分的呢？

`_typeobject`（也就是`PyTypeObject`）可以被称之为“指定对象类型的类型对象”，其定义如下：

```C
typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name; /* For printing, in format "<module>.<name>" */
    Py_ssize_t tp_basicsize, tp_itemsize; /* For allocation */

    // ...... 省略部分暂时不关心的内容

} PyTypeObject;
```

可以理解为，`PyTypeObject` 对象是 Python 中面向对象理念中“类”这个概念的实现，这里只是简单介绍其定义中的部分内容：
 - ty_name：类型名
 - tp_basicsize, tp_itemsize：创建类型对象时分配的内存大小信息
 - 被省略掉的部分：与该类型关联的操作（函数指针）

这里只是简单描述，上面的内容有些偏颇，暂不必过分深究。

再看一眼 `PyTypeObject` 的定义，可以发现在最开始也有一个 `PyObject_VAR_HEAD`，这意味着它也是一个对象。那么，`PyTypeObject` 既然是指示类型的对象，那么它的类型又是什么呢？答案是 `PyType_Type`：

```
PyTypeObject PyType_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "type",                                     /* tp_name */
    sizeof(PyHeapTypeObject),                   /* tp_basicsize */
    sizeof(PyMemberDef),                        /* tp_itemsize */
    (destructor)type_dealloc,                   /* tp_dealloc */
    // ...... 省略了部分内容
};
```

事实上，它就是 Python 语言中的 `type` 对象就是 `PyType_Type`，它是所有 class 的 class，在 Python 中叫做 metaclass。其实，在实现中它的 `ob_type` 指针又指向了自己本身，既是：

```
   PyType_Type
+-----------+<-------+
| ob_refcnt |        |
+-----------+        |
|  ob_size  +--------+
+-----------+
|           |
|   other   |
|           |
+-----------+
```

---

## 小结

简单概述了 Python 中的对象的最模糊的概念。


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/07/09/987200237.png