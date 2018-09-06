---
title: Redis data structure overview
date: 2018-09-01 01:52:13
tags: 
 - Redis
 - data-structure
---

所有后端开发的同学，一般都会使用到 Redis 作为数据存储或缓存。在我所知的很多互联网公司，Redis 都发挥着难以替代的作用。本文试图简单介绍下，Redis 实现中用到的一些数据结构。

---

## Redis 用户侧支持的数据结构

完整的 Redis 命令参考可以查看 [redisdoc.com](http://redisdoc.com/)。

### string

通过 key-value pair 的方式，存储字符串、整数、浮点数等对象。
```
+---------+    +---------+
|   key   +--->|  value  |
+---------+    +---------+
```
虽然叫做「 string 」，但其实更像是一个字节序列（底层存储也是一个字节数组），所以其实你可以存储任何东西到 string 中去。比如把 Python pickle 序列化后的对象、一张图片的二进制序列等任何东西存储为 string。
之所以说 string 是字节数组，一部分原因是其提供了直接操作 bit 的指令。
不完全等同于 byte array 的是，string 对象某些场景下可以直接在服务端被解释为一个 int/double 对象，然后直接进行一些数字相关的运算（加、减）。
string 对象是 Redis 中最基础的类型，因为后面几乎所有数据类型存储的值，都是 string 类型。

### hash

Hash table，某些地方又被称为 `map`，概念上有点类似 Python 中的 dict 或 Golang 中的 map 等。其存储的是一系列 key-value 对。

### list

List 对象概念上可以理解为 Python 中的 list、Java 中的 List、Golang 中的 slice 等。之所以说概念上，是因为这几者底层实现上其实并不相同，只是都是对一组数据的集合的抽象。
```
+-------+   +--------+--------+--------+--------+
|  key  +-->| value1 | value2 | ...... | value n|
+-------+   +--------+--------+--------+--------+
```
（逻辑上 list 是这样的，实现上下面再讲）
Redis 中 list 对象可以插入数据到 list 头或尾上，由于其底层实现是一个双向链表（某些场景下不是），所以插入两端都是 O(1) 的。

### set

Set 对象有点像是 Python 里的 set，其存储的是多个**互不相同**的元素。由于 set 底层使用 hash table 存储（同上，某些场景下不是），所以其大部分操作都是 O(1) 的。

### zset

zset 是有序集合，同 set 相似的是，其内部存储的元素也是不允许重复的。不同的是，set 中存储的元素是无序的，但是 zset 存储的元素是有序的。
通过为 zset 中每个元素设置一个 score，zset 根据元素的 score 排序。

### 其他（略）
 - HyperLogLog
 - GEO

---

## 实现用户侧数据结构的底层结构

Redis 是通过 C 语言实现的，由于 C 语言的朴素，Redis 并没有直接实现上面提到的数据结构，而是通过构件了一系列基础的数据结构，经过对象系统对下层结构的封装，来实现上层面向用户的各种结构。
下面，先介绍下这些底层结构。

### SDS
SDS 是「 simple dynamic string 」的缩写，是对 C 字符串的抽象（其实 C 语言没有字符串…… 2333）。
SDS 的定义如下（ [Redis4.0 sds 定义](https://github.com/antirez/redis/blob/4.0/src/sds.h#L44-73)，相比 3.0 及之前版本，目前版本包含多种格式的 sdshdr 定义）：
```C
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```
相比 C char array，sds 有以下优点：
 1. 获取字符串长度效率更优。C 字符串只是一个 '\0' 结尾的 char 数组，如果需要获取字符串长度，需要遍历整个数组，遍历操作时间复杂度为 O(N)。而 sds len 属性记录了本身的长度，获取长度只需要 O(1) 复杂度。
 2. 避免数组长度溢出。类似 strcat(dst, src) 等函数，如果 dst 数组剩下的空间小于 src 的长度，则在字符串连接的时候会导致数组溢出。而在 sds 中，执行字符串拼接等修改操作时，会先通过 len、alloc 属性检查剩下的空间是否足够。当空间不足时，会先分配足够的空间。
 3. 减少内存分配次数。sds 会通过预申请内存，在连接字符串等操作时，减少对内存的申请操作。同时，如果 sds 所保存的字符串变短了，也并不会立即释放内存，而是通过 len 记录已使用，剩余空间作为 buffer 暂时保留。
 4. 二进制安全。因为 C 字符串会以 '\0' 作为结束符，所以如果在 char array 中存储图片等二进制数据时，空字符会被认为是结束符。而 sds 通过 len 属性记录 buf 使用的长度，则可以避免这样的问题。
 5. 兼容部分 C 字符串函数。sds 也会在 buf 已使用的最后一位后（`sds->buf[sds->len]`）插入一个 '\0'，这样在 sds 存储文本数据时，可以方便地复用一些 `string.h` 已有的函数。

### linkedlist

Redis 中 linkedlist 是一个双向链表（[Redis4.0 linkedlist 定义](https://github.com/antirez/redis/blob/4.0/src/adlist.h#L36-54)）：
```C
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```
每个节点类似这样：
```
+---------+           +----------+        +----------+        +----------+
|  list   |           | listNode |        | listNode |        | listNode |
+---------+           +----------+  next  +----------+  next  +----------+  next
|  head   +----------->          +-------->          +-------->          +------->NULL
|         |           |  value   |        |  value   |        |  value   |
|  tail   +-+  NULL<--+          <--------+          <--------+          |
|         | |     prev+----------+  prev  +----------+  prev  +-^--------+
|  len: 3 | |                                                   |
|         | +---------------------------------------------------+
|  dup    +----> ...
|         |
|  free   +----> ...
|         |
|  match  +----> ...
+---------+
```
list 结构通过 head、tail 记录了链表头尾指针，配合每个节点的 next、prev，方便从头或者从尾遍历等操作。
另外，dup/free/match 等函数指针，则是用于实现链表的多态特性：
 1. dup 函数用于复制链表节点保存的值
 2. free 函数用于释放链表节点保存的值
 3. match 函数用于比较节点的值与另一个输入 key 是否相同

### dict

dict 类似 Python 中的 dict 或 Golang 中的 map。
在 Redis 中，dict 通过一个 dict 结构实现，底层通过一个 hashtable<dictEntry> 存储数据。
tashtable 和 dictEntry 的定义如下（[Redis4.0 dict 定义](https://github.com/antirez/redis/blob/4.0/src/dict.h#L69-74)）：
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

typedef struct dictht {
    dictEntry **table;       // hash table 实际存储的位置
    unsigned long size;      // table 的大小
    unsigned long sizemask;
    unsigned long used;      // 已经使用的长度
} dictht;
```

其中，dictEntry 是每个 key-value 对存储的结构，其 next 指针用于在 hash 冲突时，将多个 entry 连接一起：
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

dict 的定义如下（[Redis4.0 dict 定义](https://github.com/antirez/redis/blob/4.0/src/dict.h#L76-82)）：
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
其中：
 1. dictType 是一个包含一组针对不同类型 entry 特定操作函数的结构体。不同类型的 entry 通过不一样的实现，来达到多台的目的。
 2. prevdata 保存了需要传给 dictType 里的函数的特定参数（如上函数签名的 prevdata 指针）
 3. ht 是包含两个 dictht 对象的数组，ht[0] 存储数据，ht[1] 在 rehash 的时候会用到（这里只提一下，dict rehash 过程下次单写）
 4. rehashidx 记录 rehash 进度，这里不做过多介绍。

关于 dict 结构的一些细节，下次再详细介绍。

### skiplist

skiplist（跳跃表） 是一种有序的结构，通过在每个节点中维护多个指向其他节点的指针来实现快速访问节点的目的。

skiplist 的定义如下（[Redis4.0 skiplist 定义](https://github.com/antirez/redis/blob/4.0/src/server.h#L760-774)）：

```C
typedef struct zskiplistNode {
    sds ele;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned int span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;  // 头、尾指针
    unsigned long length;                 // 长度
    int level;
} zskiplist;
```
如上，zskiplistNode 由于保存每个节点的数据和各种指针等，zskiplist 用于保存整个 skiplist 相关信息。


### intset

当 set 中只包含整数元素时且元素不多时，底层的数据结构便是 intset。
intset 的定义如下（[Redis4.0 intset 定义](https://github.com/antirez/redis/blob/4.0/src/intset.h#L35-39)）：
```C
typedef struct intset {
    uint32_t encoding;
    uint32_t length;
    int8_t contents[];
} intset;
```

其中 contents 数组用于存储数据，intset 按照存储数字的大小有序排列在 contents 数组中。length 属性记录集合中元素的个数。
encoding 记录 contents 数组中存储的元素的类型：
 - `INTSET_ENC_INT16` 存储 int16 类型整数
 - `INTSET_ENC_INT32` 存储 int32 类型整数
 - `INTSET_ENC_INT64` 存储 int64 类型整数

```
+----------+
|  intset  |
+----------+
|  encoding|
|   INTSET_ENC_INT16
|          |
|  length  |
|   5      |
|          |       +----+---+---+---+------+
|  contents+------>+ -4 | 0 | 3 | 6 | 1024 |
+----------+       +----+---+---+---+------+
```

当新增元素到 intset 中时，如果新元素比现有元素类型长时，比如向 INTSET_ENC_INT16 编码的 intset 插入一个 32 位整数时，intset 需要先升级（upgrade），才能添加元素。所谓 upgrade 是将此 intset 的 enconding 更新为更长 bit 的编码格式上。当 intset 升级后不会降级，哪怕删除长 bit 元素后剩下全是短 bit 元素。

### ziplist

ziplist 是 list 和 hash 的一种底层实现。当 list 元素较少且只包含整数或短字符串时，底层会通过 ziplist 存储数据。
ziplist 是为了节约内存而设计的一种结构，本质上就是一个约束了特殊格式的 char array，或者更正确的说法，是字节序列。展开这个字节序列，大致约束的格式是这样：
```
+---------+--------+-------+--------+-----+--------+-------+ 
| zlbytes | zltail | zllen | entry1 | ... | entryN | zlend | 
+---------+--------+-------+--------+-----+--------+-------+ 
```
每部分的含义是：

field | type | sizeof | 用途
---- | --- | --- | ---
 zlbytes | uint32_t | 4 | 记录 ziplist 长度（bytes）
 zltail | uint32_t | 4 | 记录 ziplist 尾节点距开始的字节数，通过 zltail 可以方便地找到尾节点地址
 zllen | uint16_t | 2 | 记录 ziplist 节点数量：当超过 2bit 长度时，真正的节点数量需要遍历整个 ziplist 才能得到
 entry | ziplist 存储的元素 | / | ziplist 存储的元素，具体长度由具体存储的内容决定
 zlend | uint8_t | 1 | 值衡为 0xFF，标记 ziplist 结束

对于每个 entry 节点，又可以展开为这样的格式：
```
+-----------------------+----------+---------+ 
| previous_entry_length | encoding | content |
+-----------------------+----------+---------+ 
```
每部分的含义是：

field | sizeof | 用途
---- | --- | ---
 previous_entry_length | 1 or 5 | 前一个元素的长度（bytes），分两种情况：1. 前一个元素小于 254 bytes，则使用一个字节记录；2. 前一个元素长度大于 254 bytes，则这个字段第一字节衡为 0xFE，后面 4 位表示前一个元素长度
 encoding | 1 or 2 or 5 | 记录当前元素的数据类型和长度（具体本文暂略）。
 content | / | 保存节点存储的数据

---

## Redis 是如何通过底层结构构件上层数据类型的

上面介绍了用户侧使用的几种常见数据类型，也介绍了 Redis 底层用于支持上层结构而实现的一些结构。下面介绍下 Redis 是如何通过下层的结构构件上层的数据类型的。

### redisObject 对象

Redis 不直接实现上层的数据类型，是为了方便在不同场景下可以替换下层合适的数据结构，同时对上层使用屏蔽下层实现细节。在不同场景下，面对性能和内存占用不同而使用不同的下层结构支持同一个上层对象。

`redisObject` 对象完成了这层转换（[Redis4.0 redisObject 定义](https://github.com/antirez/redis/blob/4.0/src/server.h#L585-593)）：
```C
typedef struct redisObject {
    unsigned type:4;       // 类型
    unsigned encoding:4;   // 编码
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;             // 指向底层实现数据结构的指针
} robj;
```
如上结构，`type` 属性记录了对象的类型，对应上层面向用户的那些数据类型（string/list/hash 等）。对应的类型，可以通过在 redis-cli 中调用 `TYPE key` 查看每个 key 对应的类型。

而 encoding 则对应着这个 redisObject 下层使用的数据类型（如上 sds/ziplist/dict 等），常见的 encoding 有：


encoding | 对应的底层结构
---- | ---
REDIS_ENCODING_INT | 整数
REDIS_ENCODING_RAW | sds
REDIS_ENCODING_EMBSTR | embstr 编码的 sds
REDIS_ENCODING_HT | dict
REDIS_ENCODING_LINKEDLIST | linkedlist
REDIS_ENCODING_ZIPLIST | ziplist
REDIS_ENCODING_INTSET | intset
REDIS_ENCODING_SKIPLIST | skiplist

对应的下层结构，可以通过在 redis-cli 中调用 `OBJECT ENCODING key` 查看每个 key 对应的底层实现的数据结构。


### string --> int/raw/embstr

string 类型在不同场景下，下层分别由 int/raw/embstr 编码方式来实现（ embstr 是经过优化的用于保存短字符串的编码方式）。
 1. 如果 value 是一个整数，且整数长度在 8 bytes 以内，则 string 对象的编码类型为 int，redisObject 的 ptr 指针将指向一个 long 型对象。
 2. 如果 value 是一个字符串值，且长度大于 32 字节，则 string 对象编码类型为 raw，对应 redisObject 的 ptr 指针将指向一个 sds 对象。
 3. 如果 value 是一个字符串值，且长度小于等于 32 字节，则会通过 embstr 编码保存。

当 int 编码的 value 被重新赋值为字符串或通过 incr 等命令自增到超过 64 位长度时，则 Redis 会将其编码方式从 int 转换为 raw。
当 embstr 编码的 value 发生修改时，编码方式会变为 raw 方式，换言之，embstr 是 read only 的。


### list --> ziplist/linkedlist

list 的底层实现则分为 ziplist 和 linkedlist 两种。

 1. 当 list 中所有元素长度都小于 `list-max-ziplist-value` 字节，且元素数量少于 `list-max-ziplist-entries` 时，底层会选择使用 ziplist。
 2. 否则，使用 linkedlist。

当 ziplist 编码存储的 list 不满足上面 `1` 的两个条件任意一个时，Redis 就会将对应 value 的编码方式从 ziplist 转换为 linkedlist。

### hash --> ziplist/hashtable

hash 的底层实现可以为 ziplist 或者 hashtable 两种格式。

 1. 当 hash 对象所有 key-value pair 长度都小于 `hash-max-ziplist-value`，且 key-value pair 数量小于 `hash-max-ziplist-entries` 时，底层会使用 ziplist 保存 hash 对象。
 2. 否则，使用 hashtable。

当使用 ziplist 作为 hash 底层存储结构时，每个 key-value 对会连续地放置在 ziplist 中：
```
+---------+--------+-------+------+--------+------+--------+-----+------+--------+-------+
| zlbytes | zltail | zllen | key1 | value1 | key2 | value2 | ... | keyN | valueN | zlend | 
+---------+--------+-------+--^---+---^----+------+--------+-----+------+--------+-------+
                              |       |
                              +-------+
                            key-value 对
```

同 list，当 hash 对象不满足如上 `1` 的两个条件任意一个时，编码方式就会从 ziplist 转换为 hashtable。

### set --> intset/hashtable

set 的底层实现由 intset 和 hashtable 两种。

 1. 当 set 所有元素都是整数对象，且元素数量小于 `set-max-intset-entries` 时，使用 intset 作为底层编码方式。
 2. 否则，使用 hashtable。

如下分别是使用 intset 和 hashtable 时，set 对象的存储方式：
```
+---------------+
|  redisObject  |
+---------------+         +-------------+
|  type         |         |  intset     |
|   REDIS_SET   |         +-------------+
+---------------+         |  encoding   |
|  encoding     |         |   INTSET_ENC_INT16
|   REDIS_ENCODING_INTSET +-------------+ 
+---------------+         |  length     |
|  ptr          +-------->|   3         |
+---------------+         +-------------+   +---+---+---+
|  ...          |         |  contents   +-->| 1 | 3 | 5 |
+---------------+         +-------------+   +---+---+---+


+---------------+
|  redisObject  |
+---------------+         +-------------+
|  type         |         |  dict       |
|   REDIS_SET   |         +-------------+
+---------------+         | StringObject|
|  encoding     |         |  "haha"     +-->NULL
|   REDIS_ENCODING_HT     +-------------+
+---------------+         | StringObject|
|  ptr          +-------->+  "hehe"     +-->NULL
+---------------+         +-------------|
|  ...          |         | StringObject|
+---------------+         |  "keke"     +-->NULL
                          +-------------+
```

### zset --> ziplist/skiplist

有序集合有 ziplist 和 skiplist 两种方式作为底层的存储结构。

 1. 当 zset 保存的元素小于 `zset-max-ziplist-entries` 个，且所有元素长度都小于 `zset-max-ziplist-value` 字节时，zset 底层通过 ziplist 存储。
 2. 否则，使用 skiplist 存储。

同上面其他结构一样，当 `1` 条件任一不满足时，底层的数据存储结构将转换为第二种。


---

## 小结

简单总结了下 Redis 用户端常用的数据结构，以及底层抽象的各种数据结构，以及二者是如何组合起来的。

Redis 面向用户侧的各种数据结构，并不直接实现，而是通过对象系统，在特定的条件下选择特定的底层结构，以在效率和存储空间之间平衡。

```
     +----------+         +----------+               +----------+         +----------+
     |  string  |         |   list   |               |   zset   |         |   ...    |
     +----+-----+         +----+-----+               +----+-----+         +----------+
          |                    |                          |                                          
         / \                   |      +-------------------+                             
        / | \                  |      |                   |                                  
       /  |  \                 |      |                   |                                                           
    +-/   |   \----+           +--+---+-------+           +----------------------------+                              
    |     |        |              |   |       |                                        |                
+---v-+  +v----+  +v-------+  +---v---v-+  +--v--------+  +-----------+  +--------+  +-v--------+  +-----+
| int |  | raw |  | embstr |  | ziplist |  | linkedlist|  | hashtable |  | intset |  | skiplist |  | ... |
+-----+  +-----+  +--------+  +----^----+  +-----------+  +---^-----^-+  +-^------+  +----------+  +-----+
                                   |                          |     |      |   
                                   |                          |     |      |   
                                   |                          |     |      |  
                                   |   +----------+           |   +-+------+-+ 
                                   +---+   hash   +-----------+   |    set   |
                                       +----------+               +----------+ 
```










