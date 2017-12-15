---
layout:     post
title:     "python学习笔记 02 多进程"
date:      2017-12-13 17:00:00
author:    "Huper"
tags:
    python
---

对于通用编程语言来说，多线程是其很重要的一个功能，但是有时候我们会有创建和管理进程的需要，所以越来越多的编程语言都添加了多进程编程的支持，比如java5之后引入的`ProcessBuilder`。事实上在java里很多时候是用不到多进程的，因为它的多线程已经很强大了，在多核情况下比较适合解决计算密集型问题。但是python的多线程就不一样了，由于`GIL`的原因，常常需要用多进程来弥补其不足。java里的多进程我也没有接触过，打算以后学习一下再贴出来，今天主要介绍一下python的多进程。值得注意的是进程创建管理和文件系统一样都是操作系统相关的属性，所以无论是java还是python的多进程支持，都只是将操作系统的接口封装了一下供用户使用的。关于进程的概念以及进程和线程的区别我就没必要说了，大家翻翻操作系统课本就补回来了。

我们首先退一步讲，操作系统里肯定都有自己的进程相关调用，所以我们只要导入os模块就可以随意使用这些调用了。学过操作系统的话对`fork()`这个系统调用肯定不陌生，先来回顾一下：Unix/Linux操作系统提供了一个`fork()`系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是`fork()`调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。子进程永远返回`0`，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用`getppid()`就可以拿到父进程的ID。

但是说了这么多，大家可能就有问题了，如果我在是在windows下使用多进程编程，肯定就不能使用`fork()`了，那是不是说我先得弄清楚windows那一套多进程系统调用？当然这样也可以，但是那样的话你就没必要再绕一步用python的os模块来进行多进程编程了。事实上python也精心地将多线程封装进了`multiprocessing`这个模块，就像java的`ProcessBuilder`一样。

***说到这里插播一条感想，就拿模块这个东西来说，模块的概念无疑是优秀的，将这个东西和面向对象结合起来真的非常好用。虽然java9之后也引入了模块机制，但是也不可能用模块把之前的lib体系全部重构一遍了。你可能会发现java对一些旧的甚至不好的东西是很“包容”的，比如说集合框架里的Vector，Stack和HashTable，还有进程里的suspend和stop，包括在nio影响下原有的io体系，java总是选择把不好的东西放在那，并提醒你别去使用，而不是将其果断的扔掉，这样的结果可想而知，一个大坛子，好的坏的东西都在里面。但是反观python就做得比较好，很多该放弃的东西就放弃。当然java肯定也是有苦衷的，考虑到生态原因和它在各种企业的普及度，如果直接选择扔掉一些它摒弃的东西，代价就太大了。***

又说远了^__^，总之，如果使用`multiprocessing`的话我们就可以完全忽视os相关的问题了，比直接使用os模块方便多了。所以我主要是拿着个模块来进行讲解。我们先来测试一下，开启一个进程并等待其结束：

```python
from multiprocessing import Process
import os

def run(label):
    print('Process %s(id %s) is running!' % (label, os.getpid()))

if __name__ == '__main__':
    print('Original process %s is running' % os.getpid())
    p = Process(target=run, args=('child1',))
    print('Child process is created!')
    p.start()
    p.join()
    print('Child process is end')
```

这段测试主要有三个地方需要分析：

- `Process`，很明显这是多进程模块内的一个类，我们创建新进程的时候需要用到它的构造方法。看下它的构造函数标识`def __init__(self, group=None, target=None, name=None, args=(), kwargs={})`这里比较重要的两个参数就是`target`和`args`，创建进程的时候需要指定它的执行体（某个函数）和他的参数列表。
- start()函数，容易理解，开启进程，进入就绪状态。
- join()函数与java中的join()含义相同，表示等待该进程结束进入就绪状态。

好的我们看下结果：

>Original process 11552 is running
>
>Child process is created!
>
>Process child1(id 8992) is running!
>
>Child process is end

我们接着来弄点更有意思的，以前在进行java多线程编程的时候，经常是想用线程的时候随手就新建一发，由于java的GC回收时间是不可预知的，导致某个进程在执行完成后未必会被及时回收，所以“用时则建”的想法很明显是不好的，可能会造成严重的资源浪费，多线程里良好的习惯是使用线程池来管理线程。池（pool）的概念实际上应用很广，包括线程池，连接池等等，因此当进程数目比较多的时候，也建议用进程池来管理，以下是一个简单的例子：

```python
from multiprocessing import Pool
import os, time, random

def task(label):
    print('Process %s(id %s) is running!' % (label, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Process %s has ran %0.2f seconds' % (label, (end-start)))

if __name__ == '__main__':
    print('Original process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(task, args=(i,))
    print("pool start!")
    p.close()
    p.join()
    print("pool done!")
```

注意这里的`close()`方法不是表示停止，而是表示不能再添加进程了。写完这段我有有点想吐槽java的线程池了，但是今天已经吐槽过一次了，所以留到下次将java线程池的时候再说吧。其实也建议大家在以后学习语言的时候这样对比着来，感觉有点爽呀。。。上面代码的执行结果如下：

>Original process 12112.
>
>pool start!
>
>Process 0(id 292) is running!
>
>Process 0 has ran 0.01 seconds
>
>Process 1(id 292) is running!
>
>Process 2(id 3084) is running!
>
>Process 3(id 5800) is running!
>
>Process 2 has ran 0.07 seconds
>
>Process 4(id 3084) is running!
>
>Process 3 has ran 1.18 seconds
>
>Process 1 has ran 1.62 seconds
>
>Process 4 has ran 1.69 seconds
>
>pool done!

`multiprocessing`模块基本的东西差不多就这么多，下面来说一下另一个模块`subprocess`，假设我们有另一个w外部程序是c写的而且已经编译好了，我们知道要执行它的话要使用命令行参数来完成。那么如何在python程序内部启动这个进程呢？

我们先来看下这个c的程序，如下：

```c
#include <stdio.h>
int main(int argc, char** argv)
{
    int i = 1;
    for(;i < argc; i++){
        printf("第%d个参数是：%s\n", i, argv[i]);
    }
    return 0;
}
```

然后我们试着在python程序里去启动它，代码如下：

```python
import subprocess

print('调用外部进程。。。')
r = subprocess.call(['subprogress', '123', '456', '789'])
print('exit code:', r)
```

直接向`subprocess`模块的`call`函数传递参数列表就行了，结果如下：

>调用外部进程。。。
>
>第1个参数是：123
>
>第2个参数是：456
>
>第3个参数是：789
>
>exit code: 0

接下来再进一步，如果这个外部进程需要接受输入怎么办，我们可以使用管道的方式来传递参数，我们先将之前的C程序修改成一个需要接收数据版本的：

```c
#include <stdio.h>
int main()
{
    int a , b, c;
    scanf("%d%d%d",&a,&b,&c);
    printf("your input is:%d  %d  %d",a,b,c);
    return 0;
}

```

然后这样在python里对其进行调用：

```python
import subprocess

print('start calling process...')
p = subprocess.Popen(['subprogress'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
output, err = p.communicate(b'123\n456\n789\n')
print(output.decode('utf-8'))
print('Exit code:', p.returncode)
```

我们可以看到，这次除了传递参数列表，还要规定标准输入，输出和错误文件，这里分别获取了外部进程的三个管道。然后使用`communicate`函数传递输入信息，这里记得要将字符串定义为字节流，可能是是因为前面的管道是字节流管道吧。运行结果如下：

>start calling process...
>
>your input is:
>
>123  456  789
>
>Exit code: 0

多进程编程的时候进程通信是很重要的一点。操作系统上学过各种和各样的进程通讯机制：消息队列，管道，共享内存，其实连信号量都是一种通信机制。python里的进程通信机制也比较多，常用的应该是使用`Queue`，这东西类似java里的`BlockingQueue`是个一种阻塞队列，可以不显示加锁，直接完成读写者，生产者消费者等模型。之后我会写两篇博客详细讲python和java线程通信，进程和线程通信机制大同小异，所以这里我暂时就不详细讲python的进程通信了。。。



