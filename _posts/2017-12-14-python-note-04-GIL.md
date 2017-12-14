---
layout:     post
title:      "python学习笔记 04 关于GIL"
date:       2017-12-14 18:30:00
author:     "Huper"
tags:
    python
---

前面几篇博客中在对比java和python的时候可能有点偏激，这其实只是因为我比较喜欢python简洁的语法而已，有这种简洁语法的支持，编码效率是很高的。即使如此，也完全不能说明python是比java优秀的语言。那为什么经常可以看到网上有人随便就列举出一大堆python比java好的地方？其实里面大多数都是从语法细节这一个大的层面展开讲的，没有从其他大的层面来说。但是很不幸语法特性是跟开发效率和维护门槛联系最直接和密切的一点，所以大多数Programmer对这个很在意，java在这点被嫌弃就意味着失去了权重很大的一票。那么大的层面是什么？随便举些例子吧：

- python作为一种解释执行的脚本语言，意在快速响应，首先它的速度跟编译型的java是没法比的（在不考虑IO密集任务的情况下）。当然这句话不严谨，因为JVM内部的字节码也是解释执行的，但是解释字节码指令的代价要小很多，而且java发展这么长时间以来JVM的各种优化比如JIT已经给java的执行效率带来很大提高了。
- java的虚拟机JVM质量远远胜于python的解释器CPyhton，据说可以用前者轻易构建起后者。很多优秀的语言比如：scala，kotlin，groovy都是运行在JVM上的。不得不说JVM真的是很强大的一个东西，里面有太多优秀的思想和机制了，很可惜自己没有时间去钻研了。
- 还有一点就是java的多线程性能也是是要比python好的，为什么呢？就是因为python的一个历史遗留问题，也是我今天想说的东西---GIL。

>什么是GIL?

在讨论这个问题前，我已经默认你的电脑是多核的，当然如果你还在用一个单核的电脑，那么恭喜你不用去看GIL相关的问题了，对你来说有GIL是比没有GIL好的。还是先退一步讲，在多核技术的支持下，多进程和多线程都是提高CPU利用率的好方法。进程的维护代价大但是有自己的地址空间资源私有，其同步和通信只在有相应功能需求的时候会用到，线程的维护代价小但是资源，同步和通信常常是必要选项。因此线程安全对所有通用编程语言来说都是个很重要的东西，一般来好的线程体系是要符合之前说过的`Pthread`设计规范的。python的线程的确也是`Pthread`标准呀，它到底怎么鸡肋了？事实上python在语言层面本身维护着一个全局的锁机制，用来保证线程安全，这就是它的全局解释锁GIL(Global Interpreter Lock)。简单的说GIL想要确保的就是：

**每一个interpreter进程,只能同时仅有一个线程来执行, 获得相关的锁, 存取相关的资源**。

也就是解释器的使用权现在变成临界资源了，那么大家很容易就会发现，如果一个解释器进程只能有一个线程来执行，多线程的并行则成为不可能，即使这几个线程之间不存在资源的竞争。在GIL存在的情况下，Cpython事实上是这样处理多线程的：

>1. 设置GIL
>2. 切换到一个线程去运行
>3. 运行
>4. 如果有其他线程join，把线程设置为睡眠状态
>5. 解锁GIL
>  6.  再次重复以上所有步骤

但是有个例外，就是在做`IO`操作时，GIL总是会被释放。对所有面向`IO` 的程序来说，GIL 会在这个`IO` 调用之前被释放，以允许其它的线程在这个线程等待`IO` 的时候运行。如果是纯计算的程序，没有 `IO` 操作，解释器会每隔 100 次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 `sys.setcheckinterval` 来调整）如果某线程并未使用很多`IO` 操作，它会在自己的时间片内一直占用处理器（和GIL）。也就是说，`IO` 密集型的Python 程序比计算密集型的程序更能充分利用多线程环境的好处。

>举个例子

我们都知道在单核的情况下，一个死循环会占用100%的资源，那么在多核情况下，如果有两个死循环线程，可以监控到会占用200%的CPU，以此类推。

```python
import threading, multiprocessing

def deadloop():
    a = 0
    while True:
        a = a + 1

for i in range(multiprocessing.cpu_count()):
    t = threading.Thread(target=deadloop)
    t.start()
```

我的电脑是四核的，但是这段代码事实上占用我100%（400%）的效率，但是用java实现一个相同功能的代码就可以吞掉所有的CPU资源。

>GIL的支持者和反对者

有人可能会问，既然这东西会影响多线程的并行，不要它不就行了？保持python的优良传统，去其糟粕，取其精华。但是问题是这可不像`print`加不加括号这个问题这么简单。`Cpython`内部很多地方就是根据GIL写的，直接扔掉这个东西改动代价是很大的，而且也会大幅度影响单线程程序的执行速度，一个简单的尝试是在1999年(十年前)，最终的结果是导致单线程的程序速度下降了几乎2倍（这个速度对于本身已经很慢的py来说是不能接受的）。

所以我来总结一下GIL的支持和反对观点：

>反对：
>
>1. 不顺应计算机的发展潮流。
>2. 大幅度提升多线程程序的速度。
>
>支持：
>
>1. 去掉GIL会影响很多机制的使用，比如在使用module的时候都由程序员自己频繁加锁。
>2. 会较大幅度地减低单线程程序的速度。

最后来看一下 [Bruce Eckel](http://en.wikipedia.org/wiki/Bruce_Eckel) 对GIL的看法，这个人是谁就不用我多说了吧，强烈推荐他的《Thinking in java》和《Thinking in C++》：

```
I actually don't think removing the GIL is a good solution.
But I don't think threads are a good solution, either.
They're too hard to get right, and I say that after spending literally years studying threading in both C++ and Java.
Brian Goetz has taken to saying that no one can get threading right.
```

针对GIL引起的问题，一个常用的解决方案就是尽量使用`multiprocessing`模块，这也就是为什么在学习python的时候多进程很重要，而在学习java的时候基本上学好多线程就行了，很少有人去用java的多进程，因为它的多线程没那么多问题。