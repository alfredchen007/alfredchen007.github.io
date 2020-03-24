---
layout: post

title: "Linux中的Committed_AS与CommitLimit"

date: 2020-03-25 22:58:10 +0300

description:  

cover: 'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2020-03-24-%E6%88%AA%E5%B1%8F2020-03-25%E4%B8%8A%E5%8D%8812.01.54.png'

color: rgb(76,138,108)

tags: [linux]
---

 一次性能调优过程中涉及到的两个概念

Linux 中的 Committed_AS 与 CommitLimit

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2020-03-24-%E6%88%AA%E5%B1%8F2020-03-25%E4%B8%8A%E5%8D%8812.01.54.png)

### Memory Overcommit 与 OOM

Memory Overcommit 意味着操作系统承诺给进程的内存大小，超过了实际可用内存。那么表面看起来，要保证不发生内存溢出，一个操作系统显然不应该允许 Memory Overcommit. 

实际上 Linux 操作系统是允许 Memory Overcommit 的，WHY ? 

避免内存浪费。因为对于 Linux 来说，只有进程真正占用内存时才会发生物理内存页的分配，申请的时候并不会引起物理内存页的分配。而进程往往申请的内存要比实际使用内存要多，例如一个进程申请了 1024 MB 的内存，可能只使用了 512 MB，那么剩下的 512 MB 根本没有发生物理内存页的分配，也就造成了浪费。

所以 Linux 操作系统允许 Memory Overcommit，只要进程申请，我就允许，这样进程们就可以比较充分地利用系统资源了，而操作系统只能寄希望于程序实际使用的没有那么多。如果不幸，进程使用的内存确实比较多，超过了操作系统真实可用的内存，Linux 就通过 OOM killer(OOM=out-of-memory)机制选择杀死一些进程(用户态进程，不是内核线程)，以便释放内存。

> 选择哪些进程杀呢，选择进程的函数是oom_badness函数(在mm/oom_kill.c中)，该函数会计算每个进程的点数(0~1000)。点数越高，这个进程越有可能被杀死。每个进程的点数跟oom_score_adj有关，而且oom_score_adj可以被设置(-1000最低，1000最高)

Linux 还提供更决绝的机制，通过设置内核参数 vm.panic_on_oom 使得发生 OOM 时自动重启系统。

如果我就是希望不允许 Memory Overcommit 发生呢（内存有的是，不怕浪费。。。），当然也可以禁止 Memory Overcommit

Linux 2.6之后允许通过内核参数 vm.overcommit_memory 禁止memory overcommit

内核参数 vm.overcommit_memory 接受三种取值：

> 参考：https://www.kernel.org/doc/Documentation/vm/overcommit-accounting

```
0	-	Heuristic overcommit handling. Obvious overcommits of
		address space are refused. Used for a typical system. It
		ensures a seriously wild allocation fails while allowing
		overcommit to reduce swap usage.  root is allowed to 
		allocate slightly more memory in this mode. This is the 
		default.

1	-	Always overcommit. Appropriate for some scientific
		applications. Classic example is code using sparse arrays
		and just relying on the virtual memory consisting almost
		entirely of zero pages.

2	-	Don't overcommit. The total address space commit
		for the system is not permitted to exceed swap + a
		configurable amount (default is 50%) of physical RAM.
		Depending on the amount you use, in most situations
		this means a process will not be killed while accessing
		pages but will receive errors on memory allocation as
		appropriate.

		Useful for applications that want to guarantee their
		memory allocations will be available in the future
		without having to initialize every page.
```

- 0 – Heuristic overcommit handling. 这是缺省值，它允许 overcommit，内核利用某种算法计算你的内存申请是否合理，它认为不合理就会拒绝 overcommit
- 1 – Always overcommit. 允许 overcommit，对内存申请来者不拒
- 2 – Don’t overcommit. 禁止 overcommit

### Committed_AS 与 CommitLimit

好了，我们知道了 overcommit 的机制了，那么当“禁止 overcommit”时，到底使用了多少算是 overcommit 呢，

kernel 设有一个阈值，申请的内存总数超过这个阈值就算 overcommit，在`/proc/meminfo`中可以看到这个阈值的大小：

```shell
root@sdbserver1:~# grep -i commit /proc/meminfo
CommitLimit:    14073264 kB
Committed_AS:    9044744 kB
```

**CommitLimit** 就是 overcommit 的阈值，申请的内存总数超过 CommitLimit 的话就算是 overcommit

这个阈值是如何计算出来的呢？它既不是物理内存的大小，也不是 free memory 的大小，它是通过内核参数vm.overcommit_ratio 或 vm.overcommit_kbytes 间接设置的，公式如下：

CommitLimit = (Physical RAM * vm.overcommit_ratio / 100) + Swap

vm.overcommit_ratio 是内核参数，缺省值是50，表示物理内存的50%

`/proc/meminfo` 中的 **Committed_AS** 表示所有进程已经申请的内存总大小，（注意是已经申请的，不是已经分配的），如果 Committed_AS 超过 CommitLimit 就表示发生了 overcommit，超出越多表示 overcommit 越严重。Committed_AS 的含义换一种说法就是，如果要绝对保证不发生OOM (out of memory) 需要多少物理内存。

