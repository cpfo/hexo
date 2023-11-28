---
title: python函数
categories: [技术]
tags: [python]
date: 2023-11-27 17:19:14
---

主要介绍python中函数的定义，调用， python的内置函数等。

<!-- more -->

## 调用函数

python中有许多内置函数 [Built-in Functions](https://docs.python.org/3/library/functions.html)

调用函数时，传入的参数数量不对 或者 参数数据类型不对， 都会报错。

内置函数还包含数据类型转换的函数

## 定义函数

定义函数需要使用`def` 语句，后面依次写出函数名，括号，括号中的参数和冒号: ， 在缩进块中写函数体，返回值使用 `return` 返回

在Python交互环境中定义函数时，注意Python会出现...的提示。函数定义结束后需要按两次回车重新回到>>>提示符下：

```shell
>>> def func1(x):
...   print(x)
...
>>>

```

**空函数**

函数体使用 `pass` 语句，说明什么也不做。

可以用来做占位符，方便后续再补充函数体的内容。

**参数检查**

函数中可以使用参数检查函数 `isinstance` 来确保参数类型的准确性。

**返回多个值**

返回多个值，实际上返回的是一个tuple， 多个变量可以同时接收一个tuple，按位置赋给对应的值。

## 函数的参数

**位置参数**

比如函数 `fun(x,y)`, 这两个参数都是位置参数，调用函数时，传入的两个值按照位置顺序依次赋值给 `x` 和 `y`

**默认参数**

```shell
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

默认参数注意事项：
1. 必选参数在前，默认参数在后
2. 变化大的参数在前，变化小的参数在后，变化小的可以作为默认参数。
3. 默认参数可以用来兼容旧的函数，避免修改后其他调用的地方报错。
4. 默认参数可以降低调用函数的难度。
5. 默认参数的坑 **默认参数必须指向不变的对象**

**可变参数**

在参数前面加一个 `*` 号， 在函数内部，参数接收到的是一个tuple

Python允许你在list或tuple前面加一个`*`号，把list或tuple的元素变成可变参数传进去。

**关键字参数**

可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

关键字参数， 可以扩展函数的功能，比如注册功能，将一些可选项，通过关键字参数传入进去。

```shell
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```
函数除了接受必选参数外，还可以接受关键字参数 `kw`, 调用函数时， 可以传入任意个数的关键字参数

```shell

>>> person('Michael', 30)
name: Michael age: 30 other: {}
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```
也可以先组装一个dict， 把dict作为关键字参数传递进去。

```shell
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```
`**extra` 表示将`extra`这个dict的所有key-value用关键字参数传入到函数的`**kw`, `kw`将获得一个dict，是`extra`的一份拷贝，对`kw`的改动将不影响`extra`里面的数据。

**命名关键字参数**

对于关键字参数，函数调用者可以传入任意不受限制的关键字参数，如果要检查传入了哪些，就需要在函数内部通过 `kw` 进行检查，但是调用者依然可以传入任意参数。

如果要限制关键字参数的参数名，可以使用命名关键字参数。比如只接收`city`和`job`作为关键字参数，方式如下

```shell
def person(name, age, *, city, job):
    print(name, age, city, job)
```
命名关键字参数需要一个特殊的分隔符 `*`, `*` 后面的参数被视为命名关键字参数。

调用方式

```shell
>>> person('Jack', 24, city='Beijing', job='Engineer')
Jack 24 Beijing Engineer
```
两个参数必须都要传入， 否则就会报错

> TypeError: person1() missing 1 required keyword-only argument: 'job'

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了

```shell
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```
命名关键字参数必须传入参数名，这和位置参数不同。如果没有传入参数名，调用将报错：

```shell
>>> person('Jack', 24, 'Beijing', 'Engineer')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: person() missing 2 required keyword-only arguments: 'city' and 'job'
```

由于调用时缺少参数名`city`和`job`，Python解释器把前两个参数视为位置参数，后两个参数传给`*args`，但缺少命名关键字参数导致报错。

命名关键字参数也可以有缺省值，从而简化调用

```shell
def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)

>>> person('Jack', 24, job='Engineer')
Jack 24 Beijing Engineer
```

使用命名关键字参数时，如果没有可变参数，就必须加上分隔符`*` ， 如果缺少`*`，会被当做位置参数处理。

**参数组合**

可以使用上面几种参数类型进行组合，但是顺序要保证是： 必选参数，默认参数，可变参数，关键字参数和命名关键字参数。

通过一个`tuple` 和 `dict` 可以调用任意函数。`func(*args, **kw)`


**总结**

1. 默认参数一定要用不可变参数， 否则会有逻辑错误
2. `*args` 是可变参数， 接收的是一个tuple
3. `**kw` 是关键字参数， 接收的是一个dict
4. `*args` 和 `**kw` 是python的习惯写法， 最好使用习惯写法
5. 注意调用可变参数和关键字参数时候的传值方式。

## 递归函数

使用递归函数要防止栈溢出。 函数调用是通过栈`stack` 这种数据结构实现的。

解决递归调用栈溢出的方式是使用`尾递归` 优化，在函数返回的时候，调用自身本身，并且，return语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。


