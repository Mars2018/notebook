高效程序指南(伪)

常见的效率优化误区

前言
===
自编程入门以来，我很长时间迷惑于程序的运行速度，有时候认为很快运行结束的程序，真正运行起来却让我感觉很慢，又有时候觉得运行很慢的程序，实际运行起来速度又让我大跌眼镜。我常纠结于写不写一个 if else，也会因为多读一次文件担心。

开发者总是对程序效率有着莫名的执著，有的人不在意代码兼容性，有的人不关心代码可读性，可是写出高效率的程序是所有人的追求。可是追求总要有目标，程序为什么运行得慢呢，又怎么让它运行得更快呢？

本文只从宏观上讲程序运行的高效，`i++ 和 ++i`，`静态变量和类变量`类似的问题不在讨论范围。


---
程序运行到底有多快
===
首先要贴上一个图：


上面的值都是估算值，而且硬件技术的发展日新月异，这张图可能已经不再适用了，但是它给我们程序运行速度提供了一个标准量。

我们从图上可以看出，在访问速度上，`CPU cache > 内存 > 文件存储 I/O > 网络延迟`，且它们之间的速度有极大差别，可以高达数千倍。由此，我们优化程序效率时的顺序就很明显了，在其他方面无明显问题时，优先级 `网络 I/O > 文件 I/O > 代码`。

除此之外，我们还需要知道：

- 由于程序`局部性原理`，cache 命中率很高，通常在 80% 以上；
- 虽然 CPU 访问 cache 或内存速度影响很小，但是它访问次数非常多，量大到足以产生质变；
- 由于 `分支预测`（见 stack overflow 上的著名问题 [Why is it faster to process a sorted array than an unsorted array?](https://stackoverflow.com/questions/11227809/why-is-it-faster-to-process-a-sorted-array-than-an-unsorted-array)， 中文翻译：[CPU 分支预测](https://xinyo.org/archives/65922/) ）技术的存在，`if else` 语句对效率的影响并不可怕，可怕的是多重嵌套的分支语句影响代码的可读性；

---
网络 I/O
===
作为互联网开发者，开发和维护的程序几乎没有纯单机的，总要依赖别的机器的服务，而网络 I/O，作为代码执行时间的最大吞噬者，是我们最先要解决的问题。有时候我们会发现，自己辛辛苦苦又是缓存又是多线程并发折腾了很久，还不如运维调整一下机房配置来得有效。

由于 CPU 的速度非常快，是 纳秒级的，对于我们最常用的 HTTP 协议，发出一次普通的 Http 请求后，客户端 CPU 就要空等对它来说非常漫长的一段时间，毫秒级，况且 http 协议建立连接就需要 1.5个往返，https 更是需要2.5个往返。所以减少网络 I/O 的消耗，可以从两方面考虑：

- 从架构上可以将服务尽量布在内网，外网服务尽量使用 CDN，数据交互使用轻量级的协议，减少一次网络往返消耗的时间。
- 而在代码方面，只能从减少网络 I/O 次数入手了，合理地将多个网络 I/O 请求合并为一次请求，并适当添加本地缓存避免重复请求。

我这里写了两段代码通过对比显示了网络 I/O 的消耗：

- A：根据 id 进行 n 次查询，返回了 id 的和；
- B：一次查询取出 10n 次记录，遍历返回 n 个 id 的和；

结果如下表：

|n|耗时|
|---|---|
|1||
|100||
10000||

从上面可以看得出来，虽然一次性多取了 `9n` 条记录，但节约的网络开销是非常可观的。如果单条记录不大的话，节约的协议头等数据还会使使用的流量降低。

---
文件 I/O
===
从图中我们可以看出来访问硬盘的速度也是毫秒级的，有些同学可能会认为减少

文件缓冲区



---
算法复杂度
===
首先是大家公认的算法复杂度，大家都追求复杂度更低

program: x^2: x: log(x)

在n很小时花时间优化并不划算 嵌套的for循环并不可怕，可怕在可读可理解性上

---
并发
===

redis concurrent

1-500000累加 10次:10个线程 1-500000

并发不是银弹，引入了复杂度

在需要同一个变量的并发时，并发一直在阻塞并没有意义

---
小结
===

加机器

最终还是要权衡：效率，安全，开发时间，维护 等