---
layout:     post
title:      "python学习笔记 07 常用高阶函数 "
author:     "Huper"
date:       2017-12-23 15:00:00
tags:
    python
---

 python比较好的一点就是它引进了一些函数式编程特性，比如`高阶函数`，`嵌套函数`和`匿名函数`等等，虽然算不上地道的函数式编程，但是这些特性的确非常方便，对于提高代码简洁程度也有很大帮助。今天来学习python里几个有用的`高阶函数`，`高阶函数`简单说就是支持函数作为参数以及返回值，也可以用`label`去引用一个函数。
 >map/reduce
 >

这里的`map/reduce`借鉴了Google`map/reduce`思想，实现了一种便捷的`映射+规约`的组合数据处理方式。但是Google的`map/reduce`远比这里要讲的复杂，它主要实现将数据分发进行分布式处理，再将结果整合，涉及很多分布式和并行算法的理论。
python里的`map`操作是把某个映射函数作用到一些列数据上，使用方式如下：
```python
>>> def func(x):
...     return x * x
...
>>> r = map(func,[1,2,3,4,5])
>>> r
<map object at 0x00000154D5E9A550>
>>> list(r)
[1, 4, 9, 16, 25]
```
可以看到，`map`函数可以接受一个函数作为映射规则，还可以接受一个影射对象。使用`map`函数会得到一个`map`对象，可以使用`list()`方法获得相应的列表对象。我们也可以使用内置的工厂函数作为映射规则，比如如下的代码就是实现了将列表转换为字符列表：
```python
>>> list(map(str,[1,2,3,4,5]))
['1', '2', '3', '4', '5']
```
再来看下`reduce`函数，它的功能是将一个序列里的对象按照某种规则两两规约。也需要接受两个参数，一个是规约规则，另一个是规约数据：
```python
>>> from functools import reduce
>>> def add(x,y):
...     return x+y
...
>>> reduce(add, [1,2,3,4,5])
15
```
与`map`不同的是`reduce`函数在`functools`模块里，需要单独导入。事实上：
```
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```
`reduce`函数的序列部分还可以接受一个`map`对象作为参数，这两个东西搭配`lambda`表达式使用会将代码精简度提升到极致，比如说我们要计算某个字符串`ASSIC`码的总和：
```python
>>> reduce(lambda x,y : x + y, list(map(ord, '123456')))
309
# 错误写法：
>>> reduce(sum, map(ord, '123456'))
# 上述方法的改进：
>>> reduce(lambda x,y : x + y, map(ord, '123456'))
309
```
是不是发现了一个玄机？我们用自己写的函数作用于`reduce`是不要求对象可迭代的，但是`sum`这个函数作用于`reduce`就必须先将数据转换成可迭代对象。python里获取字符`ASSIC`码的操作有点不一样，需要使用`ord`函数，另外这里补充一下python和java都是强类型检查语言，不能容忍跨类别赋值和操作，python里的类型检查更强一点，因为它本身也是动态类型检查语言，就算遇到`+`这样的运算符，也是去检查对象的`__add__`和`__radd__`方法，类似java里在进行字符串相加时会检查对象的`tostring`方法。
>filter
>

顾名思义，这是个过滤器，根据指定的过滤规则，过滤指定的数据。过滤规则也是个函数，使用方式如下：
```python
>>> def is_odd(n):
...     return n % 2 == 1
...
>>> ans = filter(is_odd, [1,2,3,4,5,6,7,8,9])
>>> ans
<filter object at 0x00000154D5E9A470>
>>> list(ans)
[1, 3, 5, 7, 9]
```
可以看到，类似`map`，这里返回了一个`filter`对象，并且可以使用`list`将其转换成列表对象。我们可以用`filter`配合生成器非常轻松的实现`素数筛`的算法：
```python
>>> def _odd_gen():
...     n = 1
...     while True:
...     n += 2
  File "<stdin>", line 4
    n += 2
    ^
IndentationError: expected an indented block
>>> def _odd_gen():
...     n = 1
...     while True:
...             n += 2
...             yield n
...
# lambda实现的嵌套函数，用到了闭包
>>> def _not_divisible(n):
...     return lambda x:x % n > 0
...
>>> def primes():
...     yield 2
...     it = _odd_gen()
...     while True:
...             yield n = next(it)
>>> def primes():
...     yield 2
...     it = _odd_gen()
...     while True:
...             n = next(it)
...             yield n
...             it = filter(_not_divisible(n),it)
...
>>> for n in primes():
...     if n < 1000:
...             pass
...     else:
...             print('finished!')
...             break
...
finished!
```
>sorted
>

java里排序有很多方式，可以继承`Compareable`接口，也可以在使用`Collections.sort`方法并指定一个`Comparator`（可以使用匿名内部类）。python里的排序一般用到`sorted`函数：
```python
>>> sorted([1,5,5,8,6,1564,4])
[1, 4, 5, 5, 6, 8, 1564]
```
`sorted`也是个高阶函数，可以指定一个函数作为它的排序规则，但是要使用参数名`key`显示指定，比如：
```python
>>> sorted([1,3,-6,-2,5], key = abs)
[1, -2, 3, 5, -6]
```
想要定制化的排序？如下：
```python
# 使用匿名函数指定key
>>> sorted([(2,'123'),(1,'12'),(3,'456789'),(4,'12')], key = lambda x: len(x[1]))
[(1, '12'), (4, '12'), (2, '123'), (3, '456789')]
# 指定两个key，第一个相同时，看第二个
>>> sorted([(2,'123'),(1,'12'),(3,'456789'),(4,'12')], key = lambda x: (len(x[1]),x[0]))
[(1, '12'), (4, '12'), (2, '123'), (3, '456789')]
```
一个有趣的问题：第一个key升序，第二个key降序怎么办？不嫌麻烦的话可以排序两次；python2的话可以使用`cmp`参数，但是python3里的`sorted`取消了类似C++里的`cmp` 参数，这时可以使用`functools`模块的`cmp_to_key`函数将一个`cmp`转换成`key`再使用`sorted`；如果是需要降序的`key`是数字的话，可以直接使用：

```python
>>> sorted([(2,'123'),(1,'12'),(3,'456789'),(4,'12')], key = lambda x: (len(x[1]),-x[0]))
[(4, '12'), (1, '12'), (2, '123'), (3, '456789')]
```

