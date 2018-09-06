---
title: Redis dict rehash
date: 2018-09-04 20:06:29
tags: 
 - Redis
 - data-structure
---

字典（`dict`）是 Redis 实现中非常常用的数据结构，比如用来作为 set 和 hash 的底层实现之一，dict 也是 Redis 数据库中 redisDb 用来存储所有数据的基本格式。

## dict 的实现

### dictht(hash table) & dictEntry

Redis 中 dict 结构其实封装了 hash table（ Redis 中叫做 dictht ），如下是 Redis4.0 中，[dictht 的定义](https://github.com/antirez/redis/blob/4.0/src/dict.h#L69-74)：

```C
typedef struct dictht {
    dictEntry **table;      // hash table 实际存储的位置 
    unsigned long size;     // table 的大小
    unsigned long sizemask;
    unsigned long used;     // 已经使用的长度
} dictht;
```

如上，table 属性指向一个 `dictEntry` 指针数组的开始位置，（其实就是 `dictEntry` 指针数组）用来存储每一组键值对。size 则记录 table 的长度，used 用来记录已经存储的节点数量。


如下是 [dictEntry 的定义](https://github.com/antirez/redis/blob/4.0/src/dict.h#L47-56)：

```C
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```
如上，key 则是一个键值对的键，v 是对应的值。每个 dictEntry 除了存储键值之外，还有一个 next 指针，用来指向 hash 相同时的下一个节点，以解决 hash 冲突问题。

如下图，假如 k1/k2 对应的 hash 相同，则会通过 next 指针连接起来：

```
+----------+
|  dictht  |
+----------+    +----------+
|  table   +--->| dictEntry|
|          |    +----------+
|  size    |    |    0     +->NULL  +---------+      +---------+
|          |    +----------+        |  entry  | next |  entry  | next
|  sizemask|    |    1     +------->+---------+----->+---------+----->NULL
|          |    +----------+        | k1 | v1 |      | k2 | v2 |
|  used    |    |    2     +->NULL  +---------+      +---------+
+----------+    +----------+
                |    3     +->NULL
                +----------+
```

### dict

Redis4.0 中 [dict 的定义](https://github.com/antirez/redis/blob/4.0/src/dict.h#L76-82)如下：

```C
typedef struct dictType {
    // 计算哈希值
    uint64_t (*hashFunction)(const void *key);
    // 复制 key
    void *(*keyDup)(void *privdata, const void *key);
    // 复制 value
    void *(*valDup)(void *privdata, const void *obj);
    // 比较 key
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    // 销毁 key
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁 value
    void (*valDestructor)(void *privdata, void *obj);
} dictType;


typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

先看一下 dictType，由于 dictEntry 中的 key 是 `void*`，v 也可以是 `void*`，所以需要某种方式来操作具体的键和值。dictType 就定义了一组函数指针，dict 对象的 type 指针关联本 dict 对应的 key-value pair 实现的 dictType，以实现具体类型的计算哈希、复制键值、比较和销毁键值等操作。dict 通过这样的方式实现了多态。

dict 结构体中，prevdata 保存了需要传给 dictType 里的函数的特定参数（dictType 中各个函数签名中的 prevdata 指针）。

ht 是包含两个 dictht 对象的数组，ht[0] 存储数据，ht[1] 在 rehash 的时候会用到，下面就会提到。

rehashidx 记录 rehash 进度，本文后面会详细介绍。

在没有发生 rehash 的时候，ht[1] 是一个空的 dictht。

## hash 冲突

所谓 hash table，本质是将 hash(key) 映射到自己的 table 数组中去。
对于一个 dict 对象，一个 dictEntry 经过下面的计算即可知道需要被映射到哪个位置：
```C
hash = dict->type->hashFunction(dictEntry->key);
position = hash & dict->ht[0].sizemask;
```

想象这样的情况，如果两个 key 的哈希值相同，或者哈希值 & sizemask 相同，即两个不同的 key 被映射到 dictht.table 的同一个位置，也就是发生了 hash 冲突。

如上 dictEntry 结构中的 next 指针，Redis 通过这个指针将 hash 冲突的 dictEntry 连接到一起，以解决冲突。

由于 dictEntry 只有 next 指针，所以处于性能考虑，当 dictht 遇到 hash 冲突时，新的节点总是会被添加到这个链表的表头节点，也就是只需要 O(1) 时间复杂度。

```
before: 包含 k1-v1 的 dictht：
+----------+
|  dictht  |
+----------+    +----------+
|  table   +--->| dictEntry|
|          |    +----------+
|  size    |    |    0     +->NULL  +---------+ 
|          |    +----------+        |  entry  |  
|  sizemask|    |    1     +------->+---------+ 
|          |    +----------+        | k1 | v1 | 
|  used    |    |    2     +->NULL  +---------+ 
+----------+    +----------+
                |    3     +->NULL
                +----------+

after: 当 k2 与 k1 的 hash 冲突时，k2 会被插入到链表表头节点：
+----------+
|  dictht  |
+----------+    +----------+
|  table   +--->| dictEntry|
|          |    +----------+
|  size    |    |    0     +->NULL  +---------+      +---------+
|          |    +----------+        |  entry  | next |  entry  | next
|  sizemask|    |    1     +------->+---------+----->+---------+----->NULL
|          |    +----------+        | k2 | v2 |      | k1 | v1 |
|  used    |    |    2     +->NULL  +---------+      +---------+
+----------+    +----------+
                |    3     +->NULL
                +----------+
```

## rehash 过程

当 dictEntry 被不断插入到 dictht 中或不断被删除时，dictht 对象的 table size 对于当前存储元素个数来讲可能太小或者太大。
衡量所谓「太大」和「太小」的标准，叫做**负载因子（ load factor ）**，这个值的计算规则为：

```C
loadFactor = dictht.used / dictht.size
```

比如对于上面的例子，负载因子就是 2 / 4 = 0.5。

当：
 1. 服务器执行 BGSAVE/BGREWRITEAOF 命令且负载因子大于 5 时，Redis 会对 dictht 扩容；
 2. 服务器没有执行 BGSAVE/BGREWRITEAOF 命令且负载因子大于 1 时，Redis 会对 dictht 扩容；
 3. 负载因子小于 0.1 时，Redis 会对 dictht 缩容。

当满足这些条件时，将出发 Redis rehash 操作，具体步骤为：
 1. 为 dict->ht[1] 分配空间，具体大小取决于目前是要扩容还是缩容，以及 ht[0].used（当前 dict 大小）：
     a. 当扩容时，新的大小为第一个大于 ht[0].used * 2 的 2^n 值；
     b. 当缩容时，新的大小为第一个 ≥ ht[0].used 的 2^n 值。
 2. 将 ht[0].table 中所有的键值对依次 rehash 到 ht[1].table 中去，即依次为每个 dictEntry 计算 key 的 hash，并映射到新的 dictht.table 中。
 3. 当 ht[0].table 所有 dictEntry 全部迁移到 ht[1].table 里之后，释放 ht[0]，并将 ht[1] 设置为 ht[0]，然后再在 ht[1] 创建空的 dictht。


## 渐进式 rehash

当 dict rehash 发生时，需要将 ht[0] 中所有 dictEntry 全部 rehash 到 ht[1] 中去。想象一下如果 dict 已经非常大时，这个操作将会非常慢，以至于影响 Redis 对外提供服务的性能。
所以在 Redis rehash 过程的实现中，这个过程并不是停机一次性完成的，而是会分多次进行，渐进式完成的。上面提到的 dict->rehashidx 属性，就是用来记录 rehash 流程的。

这个渐进式 rehash 大概的流程如下：

 1. 为 dict->ht[1] 分配合适的空间，dict 此时同时持有 ht[0] 和 ht[1]；
 2. 将 dict->rehashidx 设置为 0，代表 rehash 工作开始；
 3. 当 rehash 期间，对 dict 执行的增、删、改、查操作时，Redis 除了执行操作外，还会将 dict->ht[0].table[dict->rehashidx] 上的 dictEntry rehash 到 dict->ht[1] 中，并将 rehashidx 加一；
 4. 随着 dict rehash 不断进行，最终，dict->ht[0] 上的所有元素都被 rehash 至 dict->ht[1] 中，rehash 过程完成，将 rehashidx 置为 -1;
 5. 释放 ht[0]，并将 ht[1] 设置为 ht[0]，然后再在 ht[1] 创建空的 dictht。

在 rehash 过程中，如果有新的元素插入，则会直接被插入到 dict->ht[1] 中去，ht[0] 将不会再插入数据。
而这个过程中，dict->ht[0]、ht[1] 都有部分数据，因此在 rehash 进行时，dict 的查找、删除、更新都会在两个 dictht 对象上执行。比如删除一个 key，如果这个 key 在 dict->ht[0] 中不存在，还会再在 dict->ht[1] 中查找并删除（如果存在）。

## 小结

回顾了一下 Redis 中 dict 的实现，当遇到 hash 冲突时的解决办法，以及 Redis 中 dict 是如何扩容和缩容的。
