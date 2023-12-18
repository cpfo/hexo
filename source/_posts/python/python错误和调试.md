---
title: python错误和调试
categories: [技术]
tags: [python]
date: 2023-12-04 18:29:11
---

在程序运行过程中，总会遇到各种各样的错误。有代码逻辑错误，有运行时错误，有用户输入错误等。

此外，还可以通过使用`pdb`进行调试。

<!-- more -->


# 错误处理

## try

python提供了 `try...except...finally...` 来处理异常。

不同的错误类型，可以由不同的`except`来处理。 `finally` 可以没有。

如果没有错误发生，可以在except语句块后面加一个`else`，当没有错误发生时，会自动执行`else`语句

错误其实也是class，所有的错误类型都继承自`BaseException`，所以在使用`except`时需要注意的是，捕获父类之后，下面捕获子类的代码就不会执行到了。 和java类似。

python常见的错误类型和继承关系 [exception-hierarchy](https://docs.python.org/3/library/exceptions.html#exception-hierarchy)

示例如下

```python
def error_test(num):
    try:
        print('try...')
        result = 10 / int(num)
        print('结果是 %s' % result)
    except ValueError as ve:
        print('ValueError: ', ve )
    except ZeroDivisionError as zde:
        print('ZeroDivisionError: ', zde)
    else:
        print('无异常')
    finally:
        print('finally...')

    
error_test('3')
error_test(2)
error_test('a')
error_test('0')

```

## 调用栈

如下面错误的堆栈信息

```shell
Traceback (most recent call last):
  File "err.py", line 11, in <module>
    main()
  File "err.py", line 9, in main
    bar('0')
  File "err.py", line 6, in bar
    return foo(s) * 2
  File "err.py", line 3, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
```
> Traceback (most recent call last):

第一行，告诉我们这是错误的跟踪信息。 后面可以逐行分析， 最终找到错误的根源。

> 出错的时候，一定要分析错误的调用栈信息，才能定位错误的位置。

## 记录错误

`logging` 模块可以记录错误信息。

捕获错误，然后记录错误堆栈信息，让程序继续往下运行。

## 抛出错误

因为错误是class，捕获一个错误就是捕获到该class的一个实例。因此，错误并不是凭空产生的，而是有意创建并抛出的。Python的内置函数会抛出很多类型的错误，我们自己编写的函数也可以抛出错误。


只有在必要的时候才定义我们自己的错误类型。如果可以选择Python已有的内置的错误类型（比如`ValueError`，`TypeError`），尽量使用Python内置的错误类型。

我们还可以捕获异常，记录之后将异常重新抛出，也可以转换为其他异常(要符合逻辑，不能随意转换)

```python
def bar():
    try:
        pass
    except ValueError as e:
        print('ValueError!')
        raise


try:
    10 / 0
except ZeroDivisionError:
    raise ValueError('input error!')
```


程序也可以主动抛出错误，让调用者来处理相应的错误。但是，应该在文档中写清楚可能会抛出哪些错误，以及错误产生的原因。

# 调试

## print

使用`print()`将相关变量的值打印出来，但是将来还需要删除它。

## 断言

凡是用`print()`来辅助查看的地方，都可以用断言`（assert）`来替代：

> assert n != 0, 'n is zero!'

`assert`的意思是，表达式`n != 0`应该是`True`，否则，根据程序运行的逻辑，后面的代码肯定会出错。

如果断言失败，`assert`语句本身就会抛出`AssertionError`

启动Python解释器时可以用`-O`(大写的英文O)参数来关闭`assert`， 关闭后 `assert` 可以当做 `pass`

## logging

把`print()`替换为`logging`是第3种方式，和`assert`比，`logging`不会抛出错误，而且可以输出到文件

```python
import logging
logging.basicConfig(level=logging.INFO)
```

`logging` 可以控制日志的级别， 有`debug`，`info`，`warning`，`error`, 也可以通过配置，将日志输出到不同的地方。

## pdb

第4种方式是启动Python的调试器pdb，让程序以单步方式运行，可以随时查看运行状态。

以参数`-m pdb` 启动后，pdb定位到下一步要执行的代码，可以输入命令 `l`(小写的L) 来查看代码。输入命令`n`可以单步的执行代码。

可以输入`p 变量名`来查看变量。输入命令`q` 来结束调试。

## pdb.set_trace()

`pdb` 单步执行太麻烦，我们只需要import pdb，然后，在可能出错的地方放一个pdb.set_trace()，就可以设置一个断点：

```python
# err.py
import pdb

s = '0'
n = int(s)
pdb.set_trace() # 运行到这里会自动暂停
print(10 / n)
```
运行代码，程序会自动在`pdb.set_trace()`暂停并进入`pdb`调试环境，可以用命令`p`查看变量，或者用命令`c`继续运行

## IDE调试

VS Code 插件。

虽然用IDE调试起来比较方便，但是最后你会发现，logging才是终极武器。

# 单元测试

单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。

单元测试通过后，如果对代码做了修改，可以重新跑一遍单元测试，看是否通过，来判断修改是否对原来的逻辑造成了影响。

编写单元测试时，我们需要编写一个测试类，从`unittest.TestCase`继承

以`test`开头的方法就是测试方法，不以`test`开头的方法不被认为是测试方法，测试的时候不会被执行。

对每一类测试都需要编写一个`test_xxx()`方法。由于`unittest.TestCase`提供了很多内置的条件判断，最常用的断言就是`assertEqual()`
```python
self.assertEqual(abs(-1), 1) # 断言函数返回的结果与1相等
```

另一种重要的断言就是期待抛出指定类型的Error

**运行单元测试**

最简单的运行方式是在`mydict_test.py`的最后加上两行代码，这样就可以把文件当做正常脚本运行。

```python
if __name__ == '__main__':
    unittest.main()
```

另一种方法是在命令行通过参数`-m unittest`直接运行单元测试。

这是推荐的做法，因为这样可以一次批量运行很多单元测试，并且，有很多工具可以自动来运行这些单元测试

**setUp与tearDown**

可以在单元测试中编写两个特殊的`setUp()`和`tearDown()`方法。这两个方法会分别在每调用一个测试方法的前后分别被执行。

`setUp()`和`tearDown()`方法有什么用呢？设想你的测试需要启动一个数据库，这时，就可以在`setUp()`方法中连接数据库，在`tearDown()`方法中关闭数据库，这样，不必在每个测试方法中重复相同的代码

类似 java单元测试中的 `@Before` 和 `@After`


# 文档测试

Python官方文档中的示例代码，比如 [re模块](https://docs.python.org/3/library/re.html) 就带了很多示例代码：

```shell
>>> import re
>>> m = re.search('(?<=abc)def', 'abcdef')
>>> m.group(0)
'def'
>>>
```

可以把这些示例代码在Python的交互式环境下输入并执行，结果与文档中的示例代码显示的一致。

这些代码与其他说明可以写在注释中，然后，由一些工具来自动生成文档。

既然这些代码本身就可以粘贴出来直接运行，也可以自动执行写在注释中的这些代码。

当我们编写注释时，如果写上这样的注释

```python

def abs(n):
    '''
    Function to get absolute value of number.
    
    Example:
    
    >>> abs(1)
    1
    >>> abs(-1)
    1
    >>> abs(0)
    0
    '''
    return n if n >= 0 else (-n)
```

可以明确地告诉函数的调用者该函数的期望输入和输出

并且，Python内置的“文档测试”（`doctest`）模块可以直接提取注释中的代码并执行测试。

doctest严格按照Python交互式命令行的输入和输出来判断测试结果是否正确。只有测试异常的时候，可以用...表示中间一大段烦人的输出。

示例

```python

def fact(n):

    '''
    Calculate 1*2*...*n
    
    >>> fact(1)
    1
    >>> fact(10)
    3628800
    >>> fact(-1)
    Traceback (most recent call last):
        ...
    ValueError

    '''
    if n < 1:
        raise ValueError()
    if n == 1:
        return 1
    return n * fact(n - 1)


numList = list(range(1, 20))

for i in numList:
    print('%s的阶乘结果是%s' % (i  ,fact(i) ))

if __name__ == '__main__':
    import doctest
    doctest.testmod()

```

doctest不仅可以用来测试，还可以直接作为示例代码，通过某些文档生成工具，可以自动把包含doctest的注释提取出来。
