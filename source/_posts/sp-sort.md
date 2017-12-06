---
title: 几种无用但有趣的排序算法
date: 2016-07-07 04:03:54
tags: nonsense
---

　　常见的排序算法——诸如快排、堆排或归并等——都是基于比较的，除了这种正统意义上的排序算法，最近了解了几种令人啼笑皆非的排序算法，与大家分享一下。虽然这些算法都基本不可能用到生产环境，不过，平时拿出来恶搞一下还是比较有意思的。

![New sorted logo.png][1]

<!--more-->


### 意大利面条排序(Spaghetti Sort)

　　意大利面条排序(Spaghetti Sort)的思路是，将输入分别对应到不同长度的面条上，每根面条的长度即为对应的数字的大小。比如，对于`[1, 4, 2, 8, 9]`这个输入，则分别做出长度为1cm、4cm、2cm、8cm、9cm的面条。然后，将这些面条的一头对其，用手抓住，另一头向下。然后慢慢地将手向下垂直下降，第一个触碰到桌面的面条对应的数字则为最大的数字，第二个触碰到的就是第二大的，依次类推。


### 睡眠排序(Sleep Sort)

　　睡眠排序(Sleep Sort)可以认为是意大利面条排序的计算机实现。它的算法思路是：对于输入数组`array`，开辟`array.length`个线程，对于数组中的每个元素，则对应一个线程，这个线程在睡眠所表示数字的长度之后，再将自己所对应的数字报出来即可。比如对于上面的输入`[1, 4, 2, 8, 9]`，则开辟5个线程，分别睡眠1s、4s、2s、8s、9s。这个算法虽然不太可能应用到真实开发中，不过却真的可以通过代码实现： 

```Python
#!/usr/bin/env python
# encoding=utf-8

from multiprocessing.dummy import Pool as ThreadPool
from time import sleep

def sleep_sort(array):
    rst_list = []

    def task(n):
        sleep(n / 1000.)
        rst_list.append(n)

    pool = ThreadPool(len(array))
    pool.map(task, array)
    pool.close()
    pool.join()

    return rst_list


if __name__ == '__main__':
    print sleep_sort([10, 4, 7, 2, 1, 5, 9, 8, 3, 6])

```

### 猴子排序(Bogo Sort)

　　如下关于猴子排序(Bogo Sort)的描述摘自[维基百科][2]：

> 在计算机科学中，Bogo排序（Bogo-Sort）是个既不实用又原始的排序算法，其原理等同将一堆卡片抛起，落在桌上后检查卡片是否已整齐排列好，若非就再抛一次。其名字源自Quantum bogodynamics，又称bozo sort、blort sort或猴子排序（参见[无限猴子定理][3]）。

　　所谓*无限猴子定理*，即是：让一只猴子在打字机上随机地按键，当按键时间达到无穷时，几乎必然能够打出任何给定的文字，比如莎士比亚的全套著作。

　　猴子排序的Python实现如下：
```Python
from random import shuffle
from itertools import izip, tee

def in_order(array):
    it1, it2 = tee(array)
    it2.next()
    return all(a<=b for a,b in izip(it1, it2))

def bogo_sort(array):
    while not in_order(array):
        shuffle(array)


if __name__ == '__main__':
    print bogo_sort([10, 4, 1, 5, 9, 8, 3, 6])
```

　　如果传入的是已排序的数组，猴子排序可以直接返回结果。如果不是已排序的数组，猴子排序的平均时间复杂度为O(n*n!)，最优情况下只需要一次shuffle操作，但是最差情况下则需要无限久的时间。所以，这种排序算法，基本大家就在吹牛的时候说说就好了，写在代码里，基本上就是分分钟被打死的后果。


### 量子猴排(Quantum Bogo Sort)

　　量子猴排(Quantum Bogo Sort)可以算是<u>概念上</u>对猴子排序的一种优化：洗牌的时候，使用量子化随机排列（quantumly randomized）。这样的话，我们在观测这组数之前，这组数的状态是叠加的，参见[Schrรถdinger's cat][4]。通过这种量子化随机排列，我们划分出来了个平行宇宙。接下来，在某个宇宙A中，观测一下这组数，发现运气不好，没有排序好，那么我们就销毁掉这个宇宙。然后再看看其他宇宙的运气怎么样。终于，在一个宇宙Z中，发现刚好是排好序的数组。那么我们就保留这个宇宙。最后，没有被销毁的宇宙中，数组都是恰好一次被排好序的。

　　量子猴排的时间复杂度是O(n)。

 

### 小结

　　几种无用蛋疼但是有趣的排序算法~~~


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2016/03/14/476167382.png
  [2]: https://zh.wikipedia.org/wiki/Bogo%E6%8E%92%E5%BA%8F
  [3]: https://zh.wikipedia.org/wiki/%E7%84%A1%E9%99%90%E7%8C%B4%E5%AD%90%E5%AE%9A%E7%90%86
  [4]: https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat?oldformat=true
