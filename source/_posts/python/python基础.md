---
title: python基础
categories: [技术]
date: 2023-11-23 17:21:51
tags: [python]
---

python输入输出，基础数据类型和变量。

<!-- more -->

# 输入输出

* 输入
> name = input('请输入xx')

* 输出

```shell
>>> print('hello aa')
hello aa
>>> print('hello', '张三', 'haha')
hello 张三 haha
```
`print()` 输出时，遇到逗号 `,` 会转换成空格

# python基础

## 数据类型和变量

* **整数**

十六进制，使用 `0x` 开头

比较大的数，允许使用下划线 `_` 进行分隔，如 `10_000_000_000`

- **浮点数**

浮点数可以用数学写法，比如 `1.231`, 比较大的浮点数需要用科学计数法，把10用e替代， 比如 1.23x10^9，就是`1.23e9`

- **字符串**
 
字符串用单引号 `'` 或者 双引号 `"` 括起来，转义使用 `\` ，如果字符串里面有多个字符需要转义，可以使用 `r''` ，表示`''` 里面的字符串不转义。

```shell
>>> print(r'\r\n\t\r\n\t')
\r\n\t\r\n\t
>>>
```
字符串转数字， int(123)

* **布尔值**

使用 `True` 和 `False` 表示，注意大小写。

布尔值可以使用 `and` `or` 和 `not` 进行运算。 `and`是与运算，`or`是或运算，`not`是非运算，单目运算，取反。

* **空值**

空值使用 `None` 表示， 空值是一种特殊值。

* **变量**

变量在程序中就是用一个变量名表示了，变量名必须是大小写英文、数字和`_`的组合，且不能用数字开头，和 java 一样

在Python中，等号`=`是赋值语句，动态语言，同一个变量可以多次赋值，也可以赋不同类型的值。

变量的指向:

```shell
>>> a = 'ABC'
>>> b = a
>>> a = 'XYZ'
>>> print(b)
ABC
>>>
```

1. 内存中创建了 `ABC` 字符串
2. 内存中创建了名为 `a`的变量，并把它指向了 `ABC`
3. 创建了`b`，并把`b`指向了`a`所指向的数据`ABC`
4. 重新把 `a` 指向了 `XYZ`

* **常量**

用全部大写的变量名表示常量

除法 分为 `/` 和 `//` 地板除， `/`结果是浮点数，即使能够整除，结果也是浮点数，`//` 结果是整数。

```shell

>>>
>>> 10/3
3.3333333333333335
>>> 9/3
3.0
>>> 10//3
3
>>>
```

## 字符串和编码

一个字节(byte)使用8位(bit)， `UTF-8` 变长编码。

python的字符串类型是 `str`， 在内存中以Unicode表示，一个字符对应若干个字节。

对 `bytes`类型的数据，使用 `b`前缀的单引号或者双引号表示。

对单个字符的编码，`ord()` 函数获取字符的整数表示，`chr()`函数把编码转换为对应的字符。

```shell
>>> ord('a')
97
>>> ord('人')
20154
>>> chr(78)
'N'
>>>
>>> '\u4e2d\u6587'
'中文'

```

以 `Unicode`表示的 `str` 可以使用 `encode()`函数编码为指定的 `bytes`

```shell
>>>
>>> 'abc'.encode('ascii')
b'abc'
>>> 'abc'.encode('utf-8')
b'abc'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> 'abc12'.encode('ascii')
b'abc12'
>>>
```

相反， 从网络或者磁盘上读取到的字节流就是`bytes`，可以使用 `decode()` 方法转换为 `str`

`len()` 函数可以计算 `str`的字符数，也可以计算 `bytes` 的字节数

```shell
>>>
>>> len('中文')
2
>>> len('abc')
3
>>> len('中文'.encode('utf-8'))
6
>>>
```

要坚持使用 UTF-8 格式的编码， 文件需要添加下面的开头

```shell
 #!/usr/bin/env python3
 # -*- coding: utf-8 -*-
```

**格式化问题**

格式化方法和C语言一致，用 `%` 实现。 `%s` 代表用字符串替换，`%d`代表用整数替换，`%f`代表用浮点数替换，`%x`代表用十六进制替换。

> 'Hi, %s, you have $%d.' % ('Michael', 1000000)

格式化整数和浮点数还可以指定是否补0和整数与小数的位数

format()

也可以使用字符串的`format()` 方法来实现，传入的参数依次替换字符串内的占位符。

## 条件判断

**if**

注意和 java中条件判断写法的区别。

```shell
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>
```

**模式匹配**

match case

````shell
age = 15

match age:
    case x if x < 10:
        print(f'< 10 years old: {x}')
    case 10:
        print('10 years old.')
    case 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18:
        print('11~18 years old.')
    case 19:
        print('19 years old.')
    case _:
        print('not sure.')

````

## 循环

python的循环有2种，一种是 for...in 循环，亿次把集合中的元素迭代出来。

```shell
L = ['Bart', 'Lisa', 'Adam']
for n in L:
    print('Hello, %s!' % n)
```
**while**

只要条件满足，就不断循环，条件不满足时，退出循环。

循环控制， `break` 提前结束循环， `continue` 跳过当次循环，执行下一次循环。

## list和tuple

**list**

list是python内置的数据类型，列表，是有序集合。

相关操作有
1. 使用 len(list) 获取元素个数
2. 使用索引访问元素，正序从 `0` 开始，倒序从 `-1` 开始
3. list是一个可变的有序表，使用 `.append(element)`往list中添加元素
4. 也可以使用insert(index, 元素) 方法，将元素插入指定位置。
5. 使用 pop()方法删除末尾元素，pop(i) 方法删除指定位置的元素。
6. 使用 list[i] = xxx， 直接替换对应位置的元素。
7. list中元素的数据类型可以不同。
8. list中的元素也可以是另一个list的引用。
9. 空list [] ， 长度为0

```shell
>>>
>>> names = ['a', 'b', 'c']
>>> names
['a', 'b', 'c']
>>> names[1]
'b'
>>> len(names)
3
>>> names[-1]
'c'
>>> names.append('d')
>>> names
['a', 'b', 'c', 'd']
>>> names.insert(2, 'f')
>>> names
['a', 'b', 'f', 'c', 'd']
>>> names.pop()
'd'
>>> names
['a', 'b', 'f', 'c']
>>> names.pop(1)
'b'
>>> names
['a', 'f', 'c']
>>> names[1]= '张三'
>>> names
['a', '张三', 'c']
>>>


```

**tuple**

另一种有序列表叫元组，`tuple` ，tuple和list非常相似，但是tuple一旦被初始化，就不能修改。

1. tuple 使用小括号进行初始化 `()`
2. 只有一个元素的tuple需要加个逗号`,`，避免和数据公式中的小括号造成歧义。 (1,)
3. tuple不可变 意思是元素的指向不可变。


## dict和set

**dict**

dict全称dictionary，在其他语言中也称为map，是键值对的数据类型。

>  d={'k1':1, 'k2':2, 'k3':3}

相关操作

1. dict中的key必须是 **不可变对象**
2. d[key]=xxx 进行赋值。
3. d[key]来获取对应的value，key不存在时会报错。
4. 用 `in` 判断key是否存在。
5. 可以使用 `get(key)`方法获取元素，key不存在会返回 `None`
6. 可以使用 `get(key, -1)`方法获取元素，key不存在会返回指定的默认值。
7. 使用 `pop(key)` 来删除key。
8. 元素存放位置，使用hash算法。

**set**

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

要创建一个set，需要提供一个list作为输入集合：

`s=set([1,2,3,4,5])` 或者 `s={1,2,3,4,5}`

注意要加上 set， 否则就变成了 list了。

set的特点
1. 元素无序，不可重复。
2. `add(key)` 添加元素。
3. `remove(key)` 移除元素。
4. set 可以进行交集 `&` 和 并集 `|` 的操作。
5. set和dict的唯一区别仅在于没有存储对应的value。

list, tuple, dict, set 的区别

list [] , tuple (), dict {k:v} , set {k1,k2}

