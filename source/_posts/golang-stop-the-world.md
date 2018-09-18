---
title: Golang 里一个有趣的小细节
date: 2018-09-18 00:43:55
tags: Golang
---

前几天一个小伙伴在公司 slack 问到如下 Golang 代码为什么会卡死（[Go Playground](https://play.golang.org/p/nxo4D832JCo)）：

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	var i byte
	go func() {
		for i = 0; i <= 255; i++ {
		}
	}()
	fmt.Println("Dropping mic")
	// Yield execution to force executing other goroutines
	runtime.Gosched()
	runtime.GC()
	fmt.Println("Done")
}
```

这个问题很有意思，大概涉及到 Golang 中以下三个概念：
 1. byte 是什么
 2. goroutine 如何调度
 3. Golang GC 时会发生什么

本文尝试简单解释下为什么上面的程序会卡死。

首先，先看下 main 函数里启动的 goroutine 事实上是什么东西：
```go
var i byte
go func() {
	for i = 0; i <= 255; i++ {
	}
}()
```

Golang 中，byte 其实被 alias 到 uint8 上了。所以上面的 for 循环会始终成立，因为 i++ 到 i=255 的时候会溢出，`i <= 255` 一定成立。也即是， for 循环永远无法退出，所以上面的代码其实可以等价于这样：

```go
go func() {
	for {}
}
```

其次，Goroutine 的调度是一个非常复杂的问题，这里并不打算详细介绍完整细节。
大概描述一下，目前版本的 Golang 中 goroutine 的调度（[Scalable Go Scheduler Design Doc](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)）基于 GPM 模型，G 代表 goroutine，M 可以看做真实的资源（OS Threads）。P 是 G-M 的中间层，组织多个 goroutine 跑在同一个 OS Thread 上。大概的模型如下（图偷自 Google 图片搜索）：

![image](https://user-images.githubusercontent.com/5506906/45667496-fdcfde80-bb4b-11e8-9c03-d35ef0897685.png)

如上图可以看到，一个 P 上会挂着多个 G，当一个 G 执行结束时，P 会选择下一个 G 继续执行。而当一个 G 执行太久没有结束，总也要给后面的 G 运行的机会吧。所以，Go scheduler 除了在一个 goroutine 执行结束时会调度后面的 goroutine 执行，还会在正在被执行的 goroutine 发生以下情况时让出当前 goroutine 的执行权，并调度后面的 goroutine 执行：
 - IO 操作
 - Channel 阻塞
 - system call
 - 运行较长时间

前三种这里我们不关心，最后一种情况下，如果一个 goroutine 执行时间太长，scheduler 会在其 G 对象上打上一个标志（ preempt），当这个 goroutine 内部**发生函数调用的时候**，会先主动检查这个标志，如果为 true 则会让出执行权。（这里说得比较粗略，实际会复杂一些，不过并不是本文重点所以暂不关注细节。）

回到本文开始时的例子，main 函数里启动的 goroutine 其实是一个没有 IO 阻塞、没有 Channel 阻塞、没有 system call、没有函数调用的死循环。也就是，它无法主动让出自己的执行权，即使已经执行很长时间，scheduler 已经标志了 preempt。

如上图所示，一旦这个 G （ goroutine ） 拿到执行权，它后面的 G 将无法再被当前 P 调度获得执行权。上面程序为了让这个 G 对象一定拿到执行权，在 main goroutine 中主动执行 `runtime.Gosched()` 让出了执行权。

P 的数量由 GOMAXPROCS 设置，默认为机器的 CPU 数量。

这里又分为两种情况：

1. 当这个程序跑在单核机器上的时候，P 默认只有一个，所以一旦调度到这个 G 对象就会卡死，因为永远没有机会再调度回 main goroutine 了。
2. 当这个程序跑在多核机器上的时候，程序到这一步并不会卡死，因为另一个 P 所关联的 G 队列执行完了之后，会通过 Work-Stealing 算法偷取别的 P 对象上的 G，所以 main goroutine 还是有机会被别的 P 调度到。

可是文章开始时的代码，不论是在单核的机器上，还是在多核的机器上，都会卡死。

这就涉及到第三个点了：Golang 的 GC。

Golang 的 GC 本质上是基于**标记-清除**实现的（基于此不断改进过）。
见名知意，标记-清除分为两个阶段：
 - 标记
 - 清除

其中，标记阶段是需要 STW（ Stop The World ）的，也就是会让所有正在运行的 goroutine 停下来。大概源码在[这个位置](https://github.com/golang/go/blob/release-branch.go1.11/src/runtime/mgc.go#L1316)：
```go
func gcStart(mode gcMode, trigger gcTrigger) {

	// ......
	systemstack(stopTheWorldWithSema)
	// ......

}

```

到这一步，死循环这个 goroutine 由于上面介绍的原因永远无法停下来，但是 main goroutine 阻塞在 GC STW 这里，等待所有 goroutine 停止执行。main goroutine 在等待一个永远不会为它停下的 G，于是，程序卡死了。

类似的，在设置 GOMAXPROCS 的时候，也需要 STW，所以下面的代码，和本文开始时的代码，卡死的原因是一样一样的（[Go Playground](https://play.golang.org/p/vvuD1smj9RM)）：

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func forever() {
	for {
	}
}

func main() {
	go forever()

	time.Sleep(time.Millisecond)  // 让出执行权
	runtime.GOMAXPROCS(1926)      // 等待 stw
	fmt.Println("Done")           // 永远执行不到
}
```

区区几行代码，里面的奥妙真不少呀。