---
layout:     post
title:      "java装饰器模式"
subtitle:   "详解java装饰器模式"
date:       2017-12-12 19:30:00
author:     "Huper"
tags:
    java
    设计模式
---

java里大名鼎鼎的23中设计模式很多人应该早就听说过了，这些模式作为OOP的衍生，对java开发者来说真的非常重要。正确使用这些设计实模式对代码风格，代码复用，模块解耦都是有很大提升的。想成为一名优秀的java工程师，熟练掌握这些设计模式绝对是非常有用的。

平时进行java编程大家接触的可能比较多的应该是工厂模式，代理模式和单例模式（这个面试java的时候真的巨容易考到），但是这个专题我还是打算从装饰器模式讲起，可能是因为最近正好看到了python的装饰器了吧，这两者的区别我会在以后介绍python的装饰器时进行说明。

下面正式进入讲解，首先从装饰器（decorator）这个名字上来理解，它是用来进行装饰的，具体来说就是动态地给某个对象添加一些附加的功能，可以理解为插件吧。和很多设计模式一样 ，装饰器模式也是基于接口的，不得不说接口这个东西用处真的很广，一方面它常常是代码完全解耦的关键，另一方面java很多特性包括注解都是用接口来实现的，这也算是java到现在为止为数不多的不容吐槽的优点了吧。总之具体来说装饰器具有以下特征：

>1.一个被装饰对象。
>
>2.装饰器和被装饰对象必须具有相同的接口。
>
>3.它给被装饰对象增加了新的功能。

这样一看其实和适配器模式有点像呀，编过安卓的宝宝可能对适配器模式比较熟悉，在对列表内容进行适配的时候经常会用到Adapter这个东西。这里先简单说一下两者之间的不同，等到以后将适配器模式的时候再具体展开：

>相同点：都拥有一个目标对象。
>
>不同点：适配器模式需要实现另外一个接口，而装饰器模式必须实现该对象的接口。适配器模式重点在“转换”，装饰器模式重点在“装饰”。

这里我在网上找到一张比较好装饰器模式说明图，大家可以结合着看一下：

![装饰器模式说明图](/img/in-post/1.png)

最后动手实际联系一下吧，按照装饰器模式的定义，我们首先需要一个公共接口，随便定义一个如下：

```java
interface Paintable{
    public void paint();
}
```

然后我们来定义一个被装饰的类：

```java
class Wall implements Paintable{

    @Override
    public void paint() {
        System.out.println("I've been painted into black!");
    }
}
```

接着我们再来实现两个装饰器类：

```java
class DecoratorI implements Paintable{

    private Paintable paintable;

    public DecoratorI(Paintable paintable){
        this.paintable = paintable;
    }

    @Override
    public void paint() {
        paintable.paint();
        System.out.println("Put on a picture!");
    }
}

class DecoratorII implements Paintable{

    private Paintable paintable;

    public DecoratorII(Paintable paintable){
        this.paintable = paintable;
    }

    @Override
    public void paint() {
        paintable.paint();
        System.out.println("Put on a poster!");
    }
}
```

然后测试一发：

```java
class DecoratorTest {
    public static void main(String[] args) {
        Paintable target = new Wall();
        target = new DecoratorI(new DecoratorII(target));
        target.paint();
    }
}
```

结果如下：

> I've been painted into black! 
>
> Put on a poster!
>
>  Put on a picture!

装饰的作用达到了！不仅墙壁被粉刷了，还被装饰上了一些饰品。其实装饰器模式的实质无非就是利用了多态，依赖注入这些东西。所以说java的设计模式另一个有趣的地方就是它实际上不是在凭空地引入某个特性，而是在用java的一些已有特性去创造新的功能。就像之前在《thingking in java》中看到的很多例子一样，java作为一门纯粹的面向对象编程语言，你可以用它的类和接口机制“伪”组装出很多看似不支持的东西，比如闭包，混型，多路分发等等。
