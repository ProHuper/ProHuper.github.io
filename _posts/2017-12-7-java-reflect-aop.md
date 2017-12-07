---
layout:     post
title:      "java反射机制和AOP动态代理模式"
subtitle:   "简单介绍一下动态代理模式"
date:       2017-12-7 17:00:00
author:     "Huper"
tags:
    - java
---

>反射机制

如果看过java的RTTI（运行时类型识别）机制的话会发现，java的很多性质都与这个东西有关，比如多态和反射机制都是在RTTI的基础上实现的。RTTI和我们经常说到的动态类型检查并不是一个东西，因为java和C++并不支持动态类型检查，这里说的类型识别指的是在运行过程中获取某个类或者某个实例的信息。RTTI的核心实际上是Class这个类，刚开始java的时候对Class这个类并不是很了解，基本上也没用过，后来才发现这东西挺黑科技也挺重要的，比如说在类加载的开始，每个类的运行时数据会被首先加载到方法区里，然后在堆区会自动地new一个Class对象来封装方法区里的数据。所以说类加载过程就是在堆区里产生一个可以让你访问对应类的Class对象。

下面就来详细介绍一下Class这个类吧，首先，它是个泛型类（它不一定得是），并且它可以保存某个类的所有域和所有方法，简单的说就是可以完完全全的描述某个类。另外Class类还有个非常常用的静态方法：

```java
public static Class<?> Class.forName(String className){
  ...
}
```

这个函数相信很多人都用过，在加载JDBC驱动的时候，要使用这个函数来动态地导入类。事实上这个函数还会触发一系列的动作，在加载这个类的同时对这个类进行初始化等等，这些机制我会在后面讲解java类加载机制的时候详细说明。现在只要知道，只要一个类加载之后（对应的.class文件放入方法区），就会有一个Class对象作为访问它的入口，我们如果能获得指向某个类方法区的这个Class对象就等于获得了这个类。

java反射机制的思想差不多也就是基于这个，简单描述起来，反射就是在运行时候对于某个类或者该类的任意对象，可以获得它的一切域和方法。为什么强调运行时呢？就像我之前说的，只有在某各类加载之后你才有机会获得它的Class，但是很多时候有些类的加载常常是动态的，比方说JDBC的驱动。

那么具体如何获得某各类的Class引用呢？有三种方法：

1. 使用Class.forname()方法
2. 使用Object.class域
3. 使用instance.getClass()方法

这三中方法区别还是蛮大的，下面逐个解释：

1. 该类未加载或已加载，加载（如果未加载的话）并初始化该类然后获得其Class引用。
2. 该类已加载，直接通过其class属性获得其Class引用。
3. 该类已加载，直接使用其某个实例的getClass()方法获得其Class引用。

是不是恍然大悟了？一般来说，对于编译期间已经确定的类，使用后两种方法就足够了，方法一主要适用于需要动态加载类的时候。那么或得到某个类的Class引用可以干嘛呢？很简单，我就可以访问你的一切东西了，代码示例：

```java
/**
 * Created by Huper on 2017/12/7.
 */
public class Huper {

    private int age;
    public int height;

    static {
        System.out.println("you just used forName!");
    }

    public Huper(){
        System.out.println("Huper was born!, and he's a robot.");
    }

    public Huper(int age, int height){
        this();
        this.age = age;
        this.height = height;
        System.out.println("Age set and height set!");
    }

    public int getHeight(){
        return this.height;
    }

    private void dance(){
        System.out.println("Huper is Dancing!");
    }
}

class ReflectTest{
    public static void main(String... args) throws ClassNotFoundException {
        Class huper = Huper.class;
        System.out.println("----------------------");
        Class.forName("Huper");
        System.out.println("----------------------");
        Huper huper1 = new Huper(20,180);
        huper1.getClass();
    }
}
```

运行结果如下：

> \---------------------- 
>
> you just used forName! 
>
> \---------------------- 
>
> Huper was born!, and he's a robot. 
>
> Age set and height set!

一目了然对吧，使用forName的时候顺便进行了静态初始化，使用.class知识获得Class引用而已，当时使用getClass的时候初始化和构造都已经完成了。然后我们尝试获取这个Class引用的所有属性，代码如下：

```java
class ReflectTest{
    public static void main(String... args) throws ClassNotFoundException {
        Class huper = Huper.class;
        System.out.println("----------------------");
        Field[] fields = huper.getDeclaredFields();
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append(Modifier.toString(huper.getModifiers()) + "   " + huper.getSimpleName() +"\n");
        for(Field field : fields){
            stringBuffer.append(Modifier.toString(field.getModifiers())+" ");
            stringBuffer.append(field.getType().getSimpleName() + " ");
            stringBuffer.append(field.getName()+";\n");
        }
        System.out.println(stringBuffer);
    }
}
```

运行结果如下：

>\---------------------- 
>
>public Huper 
>
>private int age; 
>
>public int height;

这里我以获取该类的属性（Field）为例进行说明，其中Modifier是访问权限类。可以看到我们甚至可以获取某个类的私有域，是在是一件很龌龊的事。当然还可以去获得它的Method和Constructor等等，我这里先贴出以下部分，感兴趣的话自己可以去查API。

|                 方法关键字                  |     含义      |
| :------------------------------------: | :---------: |
|          getDeclaredMethods()          |   获取所有的方法   |
|            getReturnType()             |  获得方法的放回类型  |
|          getParameterTypes()           | 获得方法的传入参数类型 |
| getDeclaredMethod("方法名",参数类型.class,……) |   获得特定的方法   |
|       getDeclaredConstructors()        |  获取所有的构造方法  |
| getDeclaredConstructor(参数类型.class,……)  |  获取特定的构造方法  |
|            getSuperclass()             |   获取某类的父类   |
|            getInterfaces()             |  获取某类实现的接口  |
|                  ...                   |     ...     |

>面向切面编程AOP

相信比较熟悉Spring开发模式的人对IOC（反转控制）和AOP（面向切面编程）都比较了解，这两个思想可以说是Spring的核心了，其作用都是为了降低模块和代码耦合。三大框架SSH本人也没有深究过，因为毕竟行业目标不是这方面的，如果比想做Java开发的话建议一定要吃透相关的东西。今天仅仅举一个比较简单的AOP的例子。AOP应该算是对OOP（面向对象编程）的一种补充吧。我们来想象一下这样的情况，假设有三种车：卡车，汽车和火车。并且这三种车都有自己的run方法和start方法，根据java的继承和多态的思想，我们可以用一个抽象类或者接口来实现代码的复用和多态。但是如果存在这种情况呢，假设上述start方法和run方法如下：

```java
public void run(){
  System.out.println("order received!");
  System.out.println("running...");
  System.out.println("finished");
}

public void start(){
  System.out.println("order received!");
  System.out.println("starting...");
  System.out.println("finished");
}
```

就是说两种方法在执行真的有用的东西（running和starting）之前和之后都执行了“received”还有“finished”操作，这个实际上是很常见的，在执行某个任务之前和之后都要记录日志什么。此时对于run和start方法来说，怎么解决他们之间的代码复用问题？这就是AOP的神奇之处，利用反射机制，它可以将某个方法“抽取”出来，然后在合适的时候又“织入”到你想要的“点”。java里的动态代理模式实际上就是用到了AOP的思想。代理模式是java里经常用到的一种设计模式，其实现方式主要包括静态代理模式，动态代理模式以及cglib原生代理模式，这里我简单介绍下第二种，以后在介绍设计模式的时候再详细补充。

```java
interface Car {
    void run();
    void start();
}

class Train implements Car{
    public void run(){
        System.out.println("running...");
    }
    public void start(){
        System.out.println("starting...");
    }
}

class myProxyHandler  implements InvocationHandler {
    private Object targetObject;

    public Object createProxyInstance(Object targetObject) {
        this.targetObject = targetObject;
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),
                targetObject.getClass().getInterfaces(),
                this);
    }

    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        received();
        Object ret = method.invoke(targetObject, args);
        finished();
        return ret;
    }

    private void received() {
        System.out.println("received!");
    }

    private void finished(){
        System.out.println("finished!");
    }
}

class Test {
    public static void main(String[] args) {
        myProxyHandler handler = new myProxyHandler();
        Car train = (Car) handler.createProxyInstance(new Train());
        train.run();
        System.out.println("==================");
        train.start();
    }
}
```

结果如下：

>received! 
>
>running...
>
> finished!
>
> ==================
>
> received!
>
> starting... 
>
> finished!

恩，就是这样。可以看到，上述例子里的train实例实际就是动态生成的代理，run和start方法是它代理的业务。在myProxyHandler的内部实际上完成了对切面（received和finished）的提取，它们被切入到了invoke方法中，其中createProxyInstance方法实际上就用到了反射机制和类加载机制，感兴趣的朋友可以进去里面具体研读一下。