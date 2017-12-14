---
layout:     post
title:      "python学习笔记 03 多线程"
date:       2017-12-14 17:00:00
author:     "Huper"
tags:
    python
---

昨天看了python的多进程，今天理所应当地来学习多线程。廖老师专门强调了python里的线程是`POSIX thread`也叫`Pthreads`，以前是c里的一套线程API，现在实际上是一套线程设计规范，包括调度，同步，互斥和协同工作等等。其实现在主流编程语言的线程库基本都支持这套规范，感觉尤其是java在多线程这块做得真的比较完善甚至有些复杂了。

Python的标准库提供了两个模块：`_thread`和`threading`，`_thread`是低级模块，`threading`是高级模块，对`_thread`进行了封装。绝大多数情况下，我们只需要使用`threading`这个高级模块。直接举个最简单的用例:

```python
import time, threading

def task():
    print('thread %s is running...' % threading.current_thread().name)
    for i in range(5):
        print('thread %s ----> %s' % (threading.current_thread().name, i))
        time.sleep(1)
    print('thread %s is end...' % threading.current_thread().name)

if __name__ == '__main__':
    print('thread %s is running' % threading.current_thread().name)
    t = threading.Thread(target=task, name='newT')
    t.start()
    t.join()
    print('thread %s is end' % threading.current_thread().name)
```

使用方式基本和之前创建进程一样，需要在构造函数里绑定方法执行体和线程名。然后`start()`就绪，`join()`调度。以上代码的运行结果如下：

>thread MainThread is running
>
>thread newT is running...
>
>thread newT ----> 0
>
>thread newT ----> 1
>
>thread newT ----> 2
>
>thread newT ----> 3
>
>thread newT ----> 4
>
>thread newT is end...
>
>thread MainThread is end

java和python多线程变量同步问题有点不一样，根据java的线程内存模型，我们知道每个线程会将公共变量拷贝一份到线程私有空间进行操作，如果是使用了`volatile`关键字，就会却保公有变量的变动可见性。但是在python里的内部变量默认是会覆盖全局变量的，所以同步的一般都是`global`的变量，也就是变动可见的。java里的同步机制包括`volatile`和`synchronized`关键字，还有庞大的`Lock`框架和`Concurrent`框架，可以说非常完善了。python的话常用的就是一个`Lock`类，不像java里的`Lock`单独作为一套继承体系的根，这个类是整合在`threading`模块内部的，用起来非常方便。先来举个例子：

```python
import time, threading

balance = 0;

def change(n):
    global balance
    balance += n
    balance -= n

def task(n):
    for i in range(100000):
        change(n)

t1 = threading.Thread(target=task, args=(5,))
t2 = threading.Thread(target=task, args=(6,))
t1.start()
t2.start()
t1.join()
t2.join()
print(balance)
```

一个典型的存取钱问题，按照我们的需求，这段代码应该执行结果为`0`，但是实际上两个线程建立并`start()`之后，cup调度哪个线程是不确定的，所以即使t1使用了`join`方法，也不能保证t1没有打断t2的执行。我们都知道高级语言里的大多数语句都是不满足原子性的，比如据我所知在java里只有`x=10`这样右边是常量的语句才满足原子性。所以上面这段代码是有可能得到脏读结果的。然后我们就来看一下如何使用加锁解决这个问题：

```python
balance = 0
lock = threading.Lock()

def change(n):
    global balance
    balance += n
    balance -= n

def task(n):
    for i in range(100000):
        lock.acquire()
        try:
            change(n)
        finally:
            lock.release()
```

直接通过`threading`初始化一个`Lock`然后在需要保证原子性或者互斥性的’临界区‘前后获得和释放锁就行了。使用方式基本和java里的锁相同，并且同样会抛出异常。熟悉java的话会记得java里有个可重入锁`ReentrantLock`，python里也有这个东西。再比如java里和`Lock`一起使用充当同步监视器的`Condition`，在python里也能找到。再包括java的`Concurrent`包里的一些神器在python里也有对应，但是种类是远远没有java多的。这些我就不详细说了。好了，python的多线程基本的就是这么多，事实上它的多线程有个历史性的问题，我打算专门写一篇博客讲一下。

