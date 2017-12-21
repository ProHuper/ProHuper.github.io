---
layout:     post
title:      "python学习笔记 06 迭代和生成"
author:     "Huper"
date:       2017-12-21 12:00:00
tags:
    python
---

前几天复习（预习）毛概，一直没时间更新博客，在此庆祝一下完成本科期间最后一场考试。今天来系统学习一下python里的迭代。学过java可能对迭代并不陌生，java里的`Enumeration`和`Itreator`都可以完成迭代，后者被广泛应用到`Collection`的遍历中，虽然前者已经不建议使用了，但是这两者的区别还是很重要的，这里我就不做过多介绍了。

>迭代

python中最简单的迭代是使用`for...in`来完成的，它可以直接迭代和遍历包括列表，元组，字典和字符串在内的众多类型。比如：

```python
>>> for key in {'A':1, 'B':2, 'C':3}:
...     print(key)
...
A
B
C
```

python里的字典类似java里的`map`，上述遍历方法对应到java里就麻烦多了，要首先获取`map`的`keyset`，在获取其对应的`Itreator`，然后才能遍历。并且我们知道，`map`里是不能直接获取键值对的`value`进行遍历的，最接近的方法也是使用`map`的内部类`Entry`才能按顺序获取`value`。但是在python就是可以这样‘为所欲为’：

```python
>>> for value in {'A':1, 'B':2, 'C':3}.values():
...     print(value)
...
1
2
3
```

当然也可以使用`for k, v in d.items()`这种类似java里`Entry`的遍历方式。这里我查了一下python里的`dict`也是通过哈希实现的，在正常情况下，哈希数组里的元素的存放不是按照投递顺序而是按照哈希值排的，但是在python里遍历字典的确是按照投递顺序打印的。而java里的`HashMap`则是按照哈希数组的下标进行遍历的：

```java
class test{
    public static void main(String... args){
        Map<String,Integer> map = new HashMap<>();
        map.put("asd",1);
        map.put("dgg",2);
        map.put("fhg",3);
        map.put("hghiu",4);
        map.put("eret",5);
        map.put("oioi",6);
        Iterator iterator = map.keySet().iterator();
        while(iterator.hasNext()){
            System.out.print(map.get(iterator.next())+" ");
        }
    }
}
----------------------
  6 1 5 3 2 4
```

python这样做会牺牲速度吗？应该不会，因为可以多开点空间维护一个链表把插入记录链起来就行了。就像java里的`LinkedHashMap`一样。还可以通过`collections`模块的`Iterable`来查看某个对象是否是可迭代的，这点和java里还是比较类似的：

```python
>>> from collections import Itreable
>>> isinstance('123',Iterable)
True
>>> isinstance(123,Iterable)
False
```

有时候我们需要使用原始的下标`for`循环来遍历集合的话，可以先用将目前对象转换成`enumerate`然后再进行遍历：

```python
>>> for i,value in enumerate([1,2,3]):
...     print(i,value)
...
0 1
1 2
2 3
```

如果要限制下标范围的话，可以使用‘切片’的方式：

```python
>>> for i,value in enumerate([1,2,3][1:2]):
...     print(i,value)
...
0 2
# 错误写法
for i,value in enumerate([1,2,3])[1:2]:
```

> 列表生成器

所谓的列表生成器，实际上就是列表的工厂化方法，我们来看下有哪些构造列表的tricks：

```python
>>> list(range(0,10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> L = []
>>> for x in range(0,10):
...     L.append(x*x)
...
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> [x*x for x in range(0,9)]
[0, 1, 4, 9, 16, 25, 36, 49, 64]
>>> [x*x for x in range(0,9) if x%2 == 0]
[0, 4, 16, 36, 64]
# 笛卡尔组合
>>> [x+y for x in '123' for y in '456']
['14', '15', '16', '24', '25', '26', '34', '35', '36']
>>>
```

>生成器

在生成列表的时候，是默认首先产生完全部元素，然后用所有元素去构建列表的，这样很明显比较消耗空间。可以使用生成器取代，一边计算新元素，一边添加。生成器的标志与元组相同：

```python
>>> gen = (x*x for x in range(0,9))
>>> gen
<generator object <genexpr> at 0x00000202AF87A620>
```

这样就构造了一个计算平方的生成器，使用方式如下：

```python
>>> next(gen)
0
>>> next(gen)
1
>>> next(gen)
4
```

使用`next`在遍历结束的时候会抛出异常，可以自己去处理异常，但是推荐使用另一种方法：

```python
>>> for i in gen:
...     print(i)
...
9
16
25
36
49
64
```

可以看到，这里由于我上次没有遍历完，所以遍历指针并不是从0开始的。下面以斐波那契数列生成为例，学习一下生成器里一个很重要的关键字`yield`：

```python
>>> def fib(n):
...  i,a,b = 0,0,1
...  while i < n:
...          yield b
...          a,b = b,a+b
...          i = i+1
...  return 1
...
>>> next(fib(5))
1
>>> for i in fib(5):
...     print(i)
...
1
1
2
3
5
```

使用`yield`可以将填充某个函数的迭代字段，使其具有迭代功能，值得注意的是，这种情况下如果之前使用过`next`再使用`for`遍历，迭代器的指针会重置。

> 迭代器

其实该讲的都讲得差不多了，这里我补充一点：python里跟java一样，也存在`Itreator`和`Itreable`两个类（在java里是两个接口）。`Itreable`对象会持有一个`Itreator`，自定义某个对象的迭代规则需要继承`Itreable` ，这样就可以获取它的迭代器。但是在python还有另一种神奇的操作，就是直接将一个集合对象转换成迭代器对象：

```python
>>> l = [1,2,3,4]
>>> l = iter(l)
>>> next(l)
1
```

事实上，如果不进行转换，直接使用`next(l)`，也是会先去找当前对象的`Itreator`，再去使用`next`方法。