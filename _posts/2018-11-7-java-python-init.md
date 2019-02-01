---
layout:     post
title:      "继承体系下类的初始化"
subtitle:   "java和python类的初始化对比"
date:       2018-11-7 17:00:00
author:     "Huper"
tags:
    - java
    - python
---

### 一个小问题

最近阿亮学长问了一个关于python比较有意思的问题，先来看一下:

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        self.__dict__ = self
if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
    print(a.a)
```

定义了一个继承于dict的AttrDict的类，然后实例化这个对象，居然可以直接使用a.a而不是a['a']获取value值。`__init__` 首先调用dict的构造函数，这个比较好理解，但是接着来了这么一句：`self.__dict__ = self`。 一开始可能不明白这到底是想干嘛，没关系，我们一步一步来分析。

首先为什么要调用`super`？python中的`super`和`java`中有类似的意义：调用父类的构造方法。但是由于python多继承的特性，以及java自动调用父类初始化方法的特性，这两个语言的初始化过程还是有很大差异的，后面我会详细讲到。总的来说python中并不能像java那样隐式地调用父类的构造函数，因为它是多继承的，如下：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        # super(AttrDict, self).__init__(*args, **kwargs)
        pass

if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
    print(a['a'])  # __init__方法中没有显式调用dict的初始化方法，所以这里是不行的。
    
----------------------------------------------

class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        # 使用super关键字显式调用父类方法。
        super(AttrDict, self).__init__(*args, **kwargs)

if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
    print(a['a'])
```

py中的super相比java中的super用起来也很诡异，为什么要传入类名和`self`两个参数：`super(AttrDict, self)`，我们看下以下代码：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        print(type(self))
        print(type(AttrDict))
        super(AttrDict, self).__init__(*args, **kwargs)

if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
---------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
<class '__main__.AttrDict'>
<class 'type'>
```

可以看出self实际上是一个当前类的实例，而AttrDict是一个type类型。我没有读过python的内存管理，但是如果按照java的内存管理方式，self应该在堆中，AttrDict应该在方法区或者静态区中。总之初始化过后，self中才会有实际的值：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        print(len(self))
        super(AttrDict, self).__init__(*args, **kwargs)
        print(len(self))
if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
--------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
0
3
```

具体的值会保存在self的`items`属性中：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        print(self.items())
if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
-------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
dict_items([('a', 123), ('b', 456), ('c', 789)])
```

我们可以发现比如说`('a', 123)`这个pair它是，self的items属性里的值，并不是self本身的属性。所以用self.a是不能获得对应的值的，必须使用self['a']来获取items属性里的值。那么有没有办法把‘a’变成self的属性呢？这就是之前初始化函数里第二句的作用。我们首先看一下调用这句之前self的属性：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        print(dir(self))
if __name__ == '__main__':
   	a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
--------------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
['__class__', '__contains__', '__delattr__', '__delitem__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'clear', 'copy', 'fromkeys', 'get', 'items', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']
```

可以看到由于继承自dict，所以默认是self的属性默认是dict的全部属性，可以看到里面是没有a,b,c这三个属性的，注意实际上属性的继承不需要调用super就可以完成。那么怎么添加新的属性呢？如下：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        self.m = 100
        print(dir(self))
if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
----------------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
['__class__', '__contains__', '__delattr__', '__delitem__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'clear', 'copy', 'fromkeys', 'get', 'items', 'keys', 'm', 'pop', 'popitem', 'setdefault', 'update', 'values']
```

可以发现，调用`self.m`之后，self多了一个m属性。事实上，这个新添加的属性还会同时被添加到self的`__dict__`属性中，`__dict__`专门保存人为添加的属性，所以`__dict__`中的内容是dir(self)的子集：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        # print(dir(self))
        self.m = 100
        # print(dir(self))
        print(self.__dict__)
        # self.__dict__ = self
        # print(dir(self))
        # print(self.__dict__)

if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
-------------------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
{'m': 100}
```

那么回归原来的问题，要把传进来的参数加入到self的属性中要怎么办？，因为`__dict__`是`dir(self)`的子集，所以只要把参数的dict赋值给`self.__dict__`就行了，那么传进来的参数在哪？可以直接从可变参数列表args[0]获得，因为self已经调用过父类的初始化方法了，所以self本身的值就是当时传进来的参数，所以这样写：

```python
class AttrDict(dict):
    def __init__(self, *args, **kwargs):
        super(AttrDict, self).__init__(*args, **kwargs)
        self.__dict__ = self
        # self.__dict__ = args[0] 与前一句是等价的
        print(dir(self))

if __name__ == '__main__':
    a = {'a': 123, 'b': 456, 'c': 789}
    a = AttrDict(a)
    print(a.a)
-------------------------------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
['__class__', '__contains__', '__delattr__', '__delitem__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a', 'b', 'c', 'clear', 'copy', 'fromkeys', 'get', 'items', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']
123
```

可以看到`self.__dict__`中成功多出了a,b,c三个属性，这样我们就可以直接用self.a获取相应属性的值了。注意a.a是获取a对象名为a的属性值，而a['a']是获取a对象的items属性并在其中查找key为a的值，两者的意义完全不一样。

### java中的类初始化问题

好了，这个问题解释完了，我们现在进入正题，对比`python`,`java` 两种语言在各自的继承机制下的初始化流程（c++相关的东西我最近还在看，有时间再补上来）。之前讲了这么多`python`，初始化这块我就由`java`讲起吧。我们知道`java`是单继承的，在调用父类初始化方法时可能不会遇到太多的矛盾，但是还是有很多坑的，我们首先来从`无参构造方法`讲起。

首先，最简单的，java中的每个类都会被默认添加一个无参构造方法，在用户没有指定有参构造方法时，可以保证无参构造不出错，但是如果用户仅仅指定了有参构造方法，就会出问题：

```java
// right
class Son{
}
public class test {
    public static void main(String[] args){
        Son son = new Son();
    }
}
// wrong
class Son{
  Son(int a){}
}
public class test {
    public static void main(String[] args){
        Son son = new Son();
    }
}
// right
class Son{
    Son(){}
    Son(int a){}
}
public class test {
    public static void main(String[] args){
        Son son = new Son();
    }
}
```

在没有继承发生的时候，这个规则很容易理解，但是在继承环境下，就需要注意更多问题了。java中任务，子类在初始化的时候必须首先调用父类的构造方法，再调用自己的构造方法。不像python中的父类的构造方法需要由用户显示调用，在单继承体系的java中这一过程几乎是强制的，以至于父类中如果没有无参构造方法会发生编译错误，可以参考以下几个例子：

```java
// right
class Father{}
class Son extends Father{
    Son(){
        System.out.println("son");
    }
}
// right 
class Father{
    Father(){}
    Father(int a){
        System.out.println("Father");
    }
}
class Son extends Father{
    Son(){
        System.out.println("son");
    }
}
// wrong
class Father{
    Father(int a){
        System.out.println("Father");
    }
}
class Son extends Father{
    Son(){
        System.out.println("son");
    }
}
--------------------------------------------------
public class test {
    public static void main(String[] args){
        Son son = new Son();
    }
}
```

和python一样，java中也有`super`关键字和`this`关键字，但是它们比在python中拥有更多意义，首先是`this`关键字：

>1.this可以代表当前对象的一个实例，这一点与python的self相同，不做太多介绍了。
>
>2.在构造方法中使用this关键字来调用其他的构造方法，必须出现在构造方法的第一行，并且只能出现一次，并且为了避免递归陷入，不能在两个构造方法中相互调用this()。

```java
// right
class Son{
    Son(int a){
        this();
    }
    Son(){
        System.out.println("son");
    }
}
// wrong
class Son{
    Son(int a){
        ;
        this();
    }
    Son(){
        System.out.println("son");
    }
}
// wrong
class Son{
    Son(int a){
        this();
    }
    Son(){
        this(10);
        System.out.println("son");
    }
}
```

然后是super关键字，和python中super的功能一样，但是这里super使用限制是不一样的，我们还是从子类和父类的初始化说起，主要有以下限制：

>1.如果子类有构造方法，但子类的构造方法中没有super关键字，则系统默认执行该构造方法时会产生super()代码，即该构造方法会调用父类无参数的构造方法。
>
>2.对于父类中包含有参数的构造方法，子类可以通过在自己的构造方法中使用super关键字来引用，而且必须是子类构造函数方法中的第一条语句。
>
>3.Java语言中规定当一个类中含有一个或多个有参构造方法，系统不提供默认的构造方法（即不含参数的构造方法），所以当父类中定义了多个有参数构造方法时，应考虑写一个无参数的构造方法，以防子类省略super关键字时出现错误。

```java
// right
class Father{
    Father(int a){}
}
class Son extends Father{
    Son(){super(10);}
}
// wrong
class Father{
    Father(int a){}
}
class Son extends Father{
    Son(){super(;10);}
}
```

好了，前面讲的这些想必大家其实都是知道的，我们来点新花样，我们知道java中有`static`关键字，并且有自己独特的`RTTI`（可以参考我之前的一篇博客：https://prohuper.github.io/2017/12/07/java-reflect-aop/），那么在这两个东西的限制下，多继承的初始化会发生什么变化呢？先来看一下没有继承时，java的初始化机制是怎么处理`static`的。

```java
class Son{
    Son(){
        System.out.println("Huper");
    }
    static {
        System.out.println("Professor");
    }
}
public class test {
    public static void main(String[] args){
        Son son = new Son();
    }
}
```

思考一下，上述代码中为什么先执行了`static`代码块？不应该是`new`任何类都先执行构造函数吗？事实上参考java的类加载机制，当一个类被加载到jvm的时候，它的static成员就被分配了空间，并且static代码块已经被执行了，这些操作时类级别的。而构造函数是等到new的时候才会执行，也就是说它是实例级别的。

这么一说大家可能聚疑惑了，难道还存在static代码块执行了，但是构造函数没有执行的状态吗？答案是肯定的，实时上`new`操作会触发类加载同时是触发初始化，而`RTTI`中的一些操作可以实现仅仅完成类加载，比如我之前说到的`Class.forname("...")`也就是`反射`中经常用到的动态类加载函数，来看下效果：

```java
class Son{

    int a = 10;
    static int b = 100;
    Son(){
        System.out.println("Huper");
    }
    static {
        System.out.println("Professor" + b);
    }
}
public class test {
    public static void main(String[] args) throws ClassNotFoundException {
        Class.forName("Son");
    }
}
```

是不是一目了然了，我们仅仅是完成了类加载，并执行了所有静态代码块，完成了所有静态变量的内存分配和赋值。需要注意的是反射中经常也会用到`.class`来获取某个对象在jvm中的类指针（更确切的应该叫引用，保存在栈中），但是这种方法并不能触发类加载。

接下来我们看一下如果存在继承的话，会怎么样，先看以下代码：

```java
class Father{
    static int b = 100;
    static{System.out.println("Father " + b);}
}
class Son extends Father{

    int a = 10;
    static int b = 100;
    Son(){System.out.println("Huper"); }
    static {System.out.println("Son " + b);}
}
public class test {
    public static void main(String[] args) throws ClassNotFoundException {
        Class.forName("Son");
    }
}
----------------------------------------------------------------
Father 100
Son 100
----------------------------------------------------------------
class Father{
    static int b = 100;
    Father(){ System.out.println("Huper1");}
    static{System.out.println("Father " + b);}
}
class Son extends Father{

    int a = 10;
    static int b = 100;
    Son(){System.out.println("Huper2"); }
    static {System.out.println("Son " + b);}
}
public class test {
    public static void main(String[] args) throws ClassNotFoundException {
        Son son = new Son();
    }
}
--------------------------------------------------------------------------
Father 100
Son 100
Huper1
Huper2
```

也就是说，类加载的时候，还是会首先加载父类，并完成父类的静态变量初始化。这里还有一个有趣的地方，可能有人会问，这是存在静态代码块的时候，如果是普通代码块呢？我们试一下:

```java
class Father{
    static int b = 100;
    Father(){ System.out.println("Huper1");}
    {System.out.println("father_block");}
    static{System.out.println("Father " + b);}
}
class Son extends Father{
    int a = 10;
    static int b = 100;
    Son(){System.out.println("Huper2"); }
    {System.out.println("son_block");}
    static {System.out.println("Son " + b);}
}
-----------------------------------------------
Father 100
Son 100
father_block
Huper1
son_block
Huper2
```

可以看到，有点出乎意料，代码块的执行竟然在构造方法之前，这其实不难理解，想想java的内存管理，代码块和方法体都是要被放到方法区的，加载到方法区的操作应该在执行方法之前。深入浅出，我们可以将java的初始化总结为如下流程：

>1.根据继承体系，自顶向下执行静态代码块，分配静态空间。
>
>2.根据继承体系，首先分配父类的变量空间 -> 执行父类的代码块 -> 执行父类构造函数，然后分配子类的变量空间 -> 执行子类的代码块 -> 执行子类构造函数。

更具体的推荐大家去看一下jvm的内存管理模型，一定会有很多收获。

### python中的类初始化问题

由于python是多继承的，所以它在初始化时需要解决的主要问题是父类判断，并且python中并没有采用c++相关的机制，而是有自己的一套体系。按照之前的流程，我们还算是由最简单地非继承初始化讲起。由于python中并没有函数重构，每次最多指定一个构造函数，并且如果用户制定了有参构造函数，就必须传入相应的的参数。因为python有默认参数值（java没有），可以使用这点来避免有参构造下的参数遗忘。

```python
# wrong
class A:
    # print('outside')
    def __init__(self, a):
        pass
# right 
class A:
    # print('outside')
    pass
if __name__ == '__main__':
    a = A()
```

在进行下面的内容之前，我们先来思考一个问题：由于python中是没有`static`关键字的，那么python中存不存在类似java中那样直接使用类名引用属性的行为呢？答案是肯定的，但是是不是真正意义上的静态变量我就不知道了：

```python
class A:
    m = 10
    def __init__(self):
        self.n = 100
if __name__ == '__main__':
    print(id(A.m))
    print(id(A().m))
    print(id(A().n))
    print(id(A.n)) # wrong
```

可以发现n属性只有实例才能引用。我们可以把这种属性看做一种伪static属性。那么既然存在伪的static属性，有没有伪的static方法呢？就是可以直接通过类名引用对应的方法，这点是不能直接支持的，因为不像java这门纯粹的面向对象语言，灵活的python支持嵌套函数和外部函数，所以它推荐大家都将类的静态方法抓换成外部方法使用。

```python
class A:
    def pro(self):
        print("inside")
if __name__ == '__main__':
    A.pro()  # wrong，pro方法必须有实例作为参数。
```

那如果有哪些强迫症重度患者非得实现这种操作怎么办？没关系，py是很灵活的，我们可以根据高阶函数的特性diy出这个特性，就像我在开始讲的那个问题一样。但是它本质上还是一个外部函数：

```python
def pro():
    print("outside!")
class A:
    __pro__ = pro
    def pro(self):
        print("inside")
if __name__ == '__main__':
    A.__pro__()
```

好了，说完这个，我们来看看继承环境下的python类初始化，首先由单继承看起，这部分很简单，大家可以通过下面几个例子看完，我就不解释了：

```python
# 1 -> right
class C:
    pass
class A(C):
    def __init__(self):
        print("Son")
    pass
# 2 -> right
class C:
    def __init__(self, n):
        print("father")
        print(n)
    pass
class A(C):
    def __init__(self):
        print("Son")
    pass
if __name__ == '__main__':
    a = A()
```

你一定发现了，就像之前说的，不像java，python的子类并不会主动去调用父类的构造方法，所以这里不管你怎么定义`__init__`都没有问题。那么python的继承到底做了啥？这个例子太空了，父类里除了构造方法什么都没有，我们试着在父类里定义一些“宝藏”，看看有没有传递给子类。

```python
class C:
    a = 1
    b = 2
    def __init__(self):
        self.c = 10
    pass
class A(C):
    print(C.__dict__)
    def __init__(self):
        print(dir(self))
    pass
if __name__ == '__main__':
    a = A()
```

观察结果可以发现，C的属性包括a和b还有一些核心属性，C的实例self的属性还多出了一个c，这三个属性中只有a和b能传递给子类。也就是说，python的继承中会把属性分为实例属性和类属性，只有类属性会被传递给子类。我们再来看一个有意思问题：

```python
class C:
    a = 1
    b = 2
    pass
class A(C):
    pass
if __name__ == '__main__':
    print(C.a)
    C.a = 100
    print(C.a)
-------------------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
1
100
```

我们在外部修改了C的内部属性，并且根据输出结果我们的确成功了。但是别急，我们再看下面的代码：

```python
class C:
    a = 1
    b = 2
    pass
class A(C):
    print(C.a)
    pass
if __name__ == '__main__':
    print(C.a)
    C.a = 100
    print(C.a)
----------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
1
1
100
```

看到这个你可能有点懵，我先告诉你第一个后两个输出分别对应我们在`__main__`中的两个print。这就是说在我们修改C.a的值之前，类A中的print语句已经执行了。到这里我们可以发现python和java的另一个不同，python的`__main__`函数在执行前会加载所有已定义的类，而不需要java中那样显式地加载类。我们再给上述代码加一句实例化A，会发现输出并没有变，也就是说，实例化类的时候，并不会执行类内静态语句。

```python
class C:
    a = 1
    b = 2
    pass
class A(C):
    print(C.a)
    pass
if __name__ == '__main__':
    print(C.a)
    C.a = 100
    print(C.a)
    a = A()
---------------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
1
1
100
```

先来个小小的对比：

**python的类内代码和变量类似java里的static变量和static代码，这些代码是在类加载的时候执行，不同的是python会在主函数前完成类加载，而java则需要用`new`或者`forname`显示加载类。**

好了，再次回到主题，python中需要用`super`显式地调用父类的构造方法，在单继承情况下很容易使用，那么在多继承情况下怎么使用呢？这里盗用网上一张图，python获取父类信息的方法有两种方法，分别是旧式的深度优先和新式的广度优先。

![img](http://segmentfault.com/img/bVq1gN)

```python
class B:
    def pro(self):
        print("B")
    pass
class C:
    pass
class A(C, B):
    pass
if __name__ == '__main__':
    A().pro()
------------------------------
"D:\Program Files\Python\python.exe" E:/workplace/for_pyCharm/practice/test.py
B
```

按照广度优先搜索，搜索完C没有发现pro这个类成员，继续向B搜索，然后调用其pro方法。我们接着看下含有`__init__`方法的时候：

```python
class B:
    # def __init__(self):
    #     print("B")
    pass
class C:
    def __init__(self):
        print("C")
    pass
class A(B, C):
    def __init__(self):
        super(A, self).__init__()
    pass
if __name__ == '__main__':
    A()
-----------------------------------
C
```

搜索路线与之前一样，按照广度优先的方法，遍历自底向上遍历继承树，寻找最近的一个有`__init__`方法并调用，并且参数一定要严格对其，为什么这么说，看下面：

```python
class B:
    def __init__(self, a):
        print("B")
        print(a)
    pass
class C:
    def __init__(self):
        print("C")
    pass
class A(C, B):
    def __init__(self):
        super(A, self).__init__(a)
    pass
if __name__ == '__main__':
    A()
```

这样是错误的，可能你java或者c++用多了就会觉得，解释器会自己决定调用最近的含参`__init__`方法，但事实上它并没有这么聪明。不过python中的super的确有它好玩的地方，比如你把上述代码中的`super(A, self)`改成`super(C, self)`就能获得当前广度优先搜索路径里，C的下一个节点：

```python
class B:
    def __init__(self):
        print("B")
    pass
class C:
    def __init__(self):
        print("C")
    pass
class A(C, B):
    def __init__(self):
        super(C, self).__init__()
    pass
if __name__ == '__main__':
    A()
------------------------------------
B
```

其实super还有另外一个神奇的地方，与java的super只能在构造函数中使用不同，python中的super是一个外部函数，它可以在任何地方使用，请看如下代码：

```python
class B:
    def __init__(self):
        print("B")
    pass
class C:
    def __init__(self):
        print("C")
    pass
class A(C, B):
    def __init__(self):
        print("A")
    pass
if __name__ == '__main__':
    a = A()
    super(C, a).__init__()
--------------------------------------
A
B
```

是不是很酷？其实，我个人觉得由于没有了static，代码块，以及强制父类初始化这些东西的限制，python中的继承初始化体系相比java要简单很多，总结起来就是：

>1.显示调用super完成初始化，初始化搜索路径为自底向上的广度优先。
>
>2.子只继承父类的类变量，并不继承实例变量。
>
>3.由于解释器的特性，类定义完成后会立即加载，并执行类代码块。

本来想再对比一下java和python里的反射，但是没时间了。。。