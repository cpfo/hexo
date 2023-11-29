---
title: python函数式编程
categories: [技术]
tags: [python]
date: 2023-11-28 16:29:28
---

我们通过把大段的代码拆分成函数，通过一层层的函数调用，来把复杂的任务分解成简单的任务，这种分解可以称为面向过程的程序设计。

函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。

<!-- more -->

# 高阶函数

变量名可以指向函数，函数名其实就是指向函数的一个变量。

既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

## map/reduce

> MapReduce 的原理在于将大规模的数据处理任务划分为多个并行的 Map 和 Reduce 操作，充分利用了集群中的计算资源，实现了高效的分布式计算。通过将数据处理过程分解为 Map 和 Reduce 两个阶段，并在其中引入数据的分组和排序操作，MapReduce 能够有效地处理大规模数据集，并具备容错性和可扩展性。

`map()`函数接收两个参数，一个是函数，一个是`Iterable`，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。

比如将一个整数集合转换为字符串
```shell
>>>
>>> L=[1,2,3,4,5,6,7,8,9]
>>> r = map(str, L)
>>> list(r)
['1', '2', '3', '4', '5', '6', '7', '8', '9']
```

`map()` 作为高阶函数，把运算规则抽象化了。

`reduce` 把一个函数作用在一个序列`[x1, x2, x3, ...]`上, 这个函数必须接收两个参数，`reduce`把**结果**继续和序列的下一个元素做累积计算，其效果就是

> reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)

## filter

`filter()` 函数用于过滤序列。

`filter()`也接收一个函数和一个序列。`filter()`把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

filter的关键在于正确实现一个筛选函数。

## sorted

内置的`sorted()`函数可以对list进行排序。默认按照升序排列。

`sorted()`函数也是一个高阶函数，它还可以接收一个key函数来实现自定义的排序, key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序。

`sorted()`对字符串排序，是按照ASCII的大小比较的, 可以使用`ord()`函数将字母转为数字来查看大小。由于`'Z' < 'a'`，结果，大写字母Z会排在小写字母a的前面。

反向排序，可以传入第三个参数 `reverse=True`

```shell
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```

# 返回函数

高阶函数可以把函数作为结果返回，不是直接返回的值，调用函数时，返回的是函数，只有当再次调用返回函数时，才会进行计算。例如 

```shell
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```

在函数 `lazy_sum`中又定义了函数`sum`，内部函数可以引用外部函数的参数和局部变量，当lazy_sum返回函数sum时，相关参数和变量都保存在返回的函数中，这种称为 "闭包(Closure)"

每次调用 `lazy_sum()`时，返回的是一个新的函数，两次调用的结果不相等。

**闭包**

>  返回闭包时牢记一点：返回函数不要引用任何循环变量，或者后续会发生变化的变量。

**nonlocal**

使用闭包时，内层函数引用了外层函数的局部变量，读取时没问题，如果修改，会报错

> UnboundLocalError: cannot access local variable 'x' where it is not associated with a value

例如
```shell
def inc():
    x = 0
    def fn():
        # nonlocal x
        x = x + 1
        return x
    return fn

f = inc()
print(f()) # 1
print(f()) # 2
```
原因是 `x` 作为局部变量没有初始化，需要在fn函数内部加上一个 `nonlocal x` 的声明，加上后，解释器会把`fn()`内部的 `x` 看做外层函数的局部变量。

>  使用闭包时，对外层变量赋值前，需要先使用nonlocal声明该变量不是当前函数的局部变量。

# 匿名函数lambda

我们在传入函数时，有时候不需要显示的定义函数，直接传入匿名函数更方便。例如

```shell
list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
```
匿名函数 `lambda x: x * x` 实际上就是

```shell
def f(x):
    return x * x
```
关键字 `lambda` 表示匿名函数，冒号前面的 `x` 表示参数。匿名函数只能有一个表达式，不用return， 返回值就是表达式的结果。

匿名函数也是一个函数对象，也可以把匿名函数赋值给一个变量，再利用变量来调用该函数

```shell
>>> f = lambda x : x + x
>>> f
<function <lambda> at 0x000001A9659FA160>
>>> f(5)
10
```
也可以把匿名函数作为返回值返回
```shell
def build(x, y):
    return lambda: x * x + y * y
```
# 装饰器

由于函数是一个对象，函数可以赋值给一个变量，通过变量也能调用该函数。函数对象有一个 `__name__` 属性，可以拿到函数名。

假设要增加函数的功能，不修改函数的定义，这种在代码运行期间动态增加功能的方式，称为 "装饰器"(Decorator)。 类似java的 `AOP`

````shell

def log(func):
    # @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call function %s' % func.__name__)
        return func(*args, **kw)
    return wrapper

@log
def now():
    print('2023-11-01')

now()
````

把`@log` 放到 `now()`函数的定义处，相当于执行了
> now = log(now)

原来的`now`函数依然存在，只是现在的 `now` 指向了新的函数。就是 `log` 中返回的 `wrapper` 函数。

如果decorator本身需要传入参数，就需要再包一层函数。

```shell

def log(text):
    def decorator(func):
        # @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s()' % (text, func.__name__))
            return func(*args, **kw)
        
        return wrapper
    return decorator

@log('执行了')
def now():
    print('2023-11-01')

now()
```

上面的2种写法，会导致调用函数时，返回的`__name__` 发生变化，因为实际上函数名已经指向新的函数了。所以需要把原始函数的`__name__`等属性赋值给新函数， 否则有些依赖函数签名的代码会执行错误。

使用内置的 `functools.wraps` 来进行处理。 如上面代码块中注释掉的部分。

# 偏函数

`functools` 模块提供了很多功能， 其中一个就是偏函数(Partial function)

函数通过设定参数的默认值，可以降低调用难度，偏函数也可以做到，比如 `int()`类型转换

**偏函数就是将某些参数固定住，简化调用**

```shell
>>> int('1111')
1111
>>> int('1111', base=2)
15
>>> int('1111', base=16)
4369
>>> int('1111', base=8)
585
```

假设我们需要做大量的二进制转换，每次传入 `base=2`会很麻烦， 可以定义一个 int2() 函数， 默认把`base=2` 传入进去。

```shell
def int2(x, base=2):
    return int(x, base)
```

`functools.partial`就是帮助我们创建一个偏函数，不需要自己定义 `int2()`

```shell
>>> import functools
>>> int8=functools.partial(int, base=8)
>>> int8('1111')
585
>>> int8('111100')
37440
>>> int8('11001')
4609
```

所以`functools.partial`的作用就是， 把一个函数的某些参数固定住(设置默认值)， 返回一个新函数，调用新函数会更简单。

新函数仅仅是把参数设置了默认值，但是调用的时候依然可以传入其他的值.
```shell
>>> int8('1111',  base=10)
1111
>>>
```

创建偏函数时，实际上可以接收函数对象、`*args`和`**kw`这3个参数
当传入
> max2 = functools.partial(max, 10)

实际上会把`10`作为`*args`的一部分自动加到左边


```shell
max2(5, 6, 7)
# 相当于:
args = (10, 5, 6, 7)
max(*args)
```
实际的结果是 `10`