---
title: python高级特性
categories: [技术]
tags: [python]
date: 2023-11-28 14:11:03
---

主要介绍切片，迭代，列表生成式，生成器，迭代器等高级特性。

<!-- more -->

## 切片

取一个list或tuple的部分元素，对于指定索引范围的操作，python提供了切片(Slice)操作符。

比如一个list如下
> L=[1,2,3,4,5,6,7,8,9]

可以如下操作

```shell
>>> L[0:4]
[1, 2, 3, 4]
```
`L[0:4]` 表示 从索引`0` 开始，直到索引`4` 为止，不包含结尾的索引`4` ， 所以取的是索引位置  `0` `1` `2` `3` 对应的元素。

如果第一个索引位置是0 ， 也可以省略

```shell
>>> L[:2]
[1, 2]
```
python支持按照倒序通过索引获取元素，同样也支持按照倒序进行切片。 倒序的第一个索引是 `-1`

```shell
>>> L[-2:]
[8, 9]
>>> L[-4:-2]
[6, 7]
>>> L[0:-2]
[1, 2, 3, 4, 5, 6, 7]
```

也可以对空集合进行切片, 对空集合直接只用索引访问会报错，但是切片不会。 

```shell
>>> L=[]
>>> L[0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> L[0:]
[]
>>> L[0:1]
[]
>>> L[:1]
[]
```

使用场景

比如创建一个0-99的数列。

> L=list(range(100))

取前10个

```shell
>>> L[:10]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

后10个
```shell
>>> L[-10:]
[90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
```
前10个数，每两个取一个：
```shell
>>> L[:10:2]
[0, 2, 4, 6, 8]
```
所有数，每5个取一个：

```shell
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```

只写 `[:]` 原样复制一个list

```shell
>>> L[:]
[0, 1, 2, 3, ..., 99]
```
tuple 也可以做上述操作，只是tuple切片的结果依然是tuple。

字符串 `xxxxx` 也可以看成是一种list，每个元素就是一个字符,  也可以进行切片。

```shell
>>>
>>> 'abcdefg'[0:1]
'a'
>>> 'abcdefg'[2:]
'cdefg'
>>> 'abcdefg'[::]
'abcdefg'
>>> 'abcdefg'[::3]
'adg'
>>>
```

## 迭代

通过for循环来遍历集合，称为迭代。

在python中， 迭代是通过 `for ... in`  来完成的。和 java的迭代类似。

dict的迭代

```shell

# 迭代 key
d = {"k1":"v1", "k2" : "v2", "k3":"v3"}
for k in d:
    print(k)

# 迭代value
for v in d.values():
    print(v)
    
# 同时迭代 key和value
for k, v in d.items():
    print(k, v)

```
dict 默认迭代的是 `key`， 迭代出的顺序可能不一样。

字符串也可以进行迭代，输出每一个字符。  

只要一个对象是可迭代的， `for` 循环就可以运行。

通过 `collections.abc` 模块的 `Iterable` 判断对象是否是可迭代的。

```shell
>>> from collections.abc import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

对list使用下标循环， 可以使用 `enumerate` 将list转换为 索引-元素对

```shell
L=['a', 'b', 'c', 'd']
for i, v in enumerate(L):
    print(i, v) 
```

## 列表生成式

要生成list `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]` 可以用`list(range(1, 11))`

```shell
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
生成 `[1x1, 2x2, 3x3, ..., 10x10]`

```shell
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```
把要生成的元素`x*x` 放在前面， 后面跟上 `for` 循环。 

for循环后面还可以加上if判断，这样我们就可以筛选出仅偶数的平方

```shell
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```

**if else**

列表生成式， for 后面的`if` 是筛选条件， 所以不能加上 `else`， 否则无法筛选。

`for` 前面的`if` 是表达式， 必须加上else 

## 生成器

列表生成式，直接把元素都创建出来了，会占用内存，数量也会受到内存的限制。

所以，如果元素按照某种算法推算出来，在循环中不断推算出后面的元素，就不必创建完整的list， 这种一边循环一边计算的机制，称为生成器 generator

创建 generator 有很多种方法，可以把生成式的 `[]` 改成 `()` ，就创建了一个 generator

```shell
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x000001A96593F780>
>>> for i in g:
...   print(i)
...
```
可以调用 `next(g)` 方法获取下一个元素， 也可以使用for 循环进行迭代。

要把普通函数改造成 generator函数，需要加上 `yield` 关键字。

generator函数在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行。

**调用generator函数会创建一个generator对象，多次调用generator函数会创建多个相互独立的generator。**


## 迭代器

可以直接作用于`for` 循环的对象，统称为可迭代对象`Iterable`

可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：`Iterator`. 可以使用`isinstance()`判断一个对象是否是`Iterator`对象

把`list` `dict`  `str` 等 `Iterable`变成 `Iterator` 可以使用 `iter()` 函数。 和java 的`iterator()`方法类似。

`Iterator`对象表示的是一个数据流， Iterator对象可以不断被`next()`函数调用并返回下一个数据，计算是惰性的，只有在需要返回下一个对象时才进行计算。

`for`循环本质上就是通过不断调用`next()`函数实现的