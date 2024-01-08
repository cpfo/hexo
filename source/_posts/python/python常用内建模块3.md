---
title: python常用内建模块3
categories: [技术]
tags: [python]
date: 2023-12-26 10:40:13
---

本章主要介绍itertools， contextlib， urllib，XML，HTMLParser的使用。

<!-- more -->

# itertools

`itertools`提供了非常有用的用于操作迭代对象的函数

`itertools.count()` 自然数无限迭代。

`itertools.cycle('abc')`， 将传入的序列无限重复下去

`repeat()` 把一个元素重复下去，可以指定次数。`itertools.repeat('a', 5)`

无限序列虽然可以无限迭代下去，但是通常我们会通过`takewhile()`等函数根据条件判断来截取出一个有限的序列

`chain()`， 可以把一组迭代对象串联起来，组成一个更大的迭代器。`itertools.chain('ABC', 'XYZ')`

`groupby()` 把迭代器中相邻的重复元素挑出来放在一起

```shell
>>> import itertools
>>> for key, group in itertools.groupby('aabbccaaeeabc'):
...    print(key, list(group))
...
a ['a', 'a']
b ['b', 'b']
c ['c', 'c']
a ['a', 'a']
e ['e', 'e']
a ['a']
b ['b']
c ['c']
```

# contextlib

在Python中，读写文件这样的资源要特别注意，必须在使用完毕后正确关闭它们。正确关闭文件资源的一个方法是使用`try...finally`，也可以使用`with`语句进行简化。

并不是只有`open()`函数返回的fp对象才能使用`with`语句。实际上，任何对象，只要正确实现了上下文管理，就可以用于`with`语句。

实现上下文管理是通过`__enter__`和`__exit__`这两个方法实现的。

编写`__enter__`和`__exit__`仍然很繁琐，因此Python的标准库`contextlib`提供了更简单的写法。`@contextmanager`这个decorator接受一个generator，用`yield`语句把`with ... as var`把变量输出出去，然后，`with`语句就可以正常地工作了

很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用`@contextmanager`实现

```python
@contextmanager
def tag(name):
    print("<%s>" % name)
    yield
    print("</%s>" % name)

with tag("h1"):
    print("hello")
    print("world")

```

执行结果

```shell
<h1>
hello
world
</h1>
```

执行顺序

1. with语句首先执行yield之前的语句，
2. yield调用会执行with语句内部的所有语句，因此打印出hello和world；
3. 最后执行yield之后的语句。

**@closing**

如果一个对象没有实现上下文，我们就不能把它用于`with`语句。这个时候，可以用`closing()`来把该对象变为上下文对象。例如，用`with`语句使用`urlopen()`

`closing`也是一个经过`@contextmanager`装饰的generator，这个generator编写起来其实非常简单

```python
@contextmanager
def closing(thing):
    try:
        yield thing
    finally:
        thing.close()
```

# urllib

urllib提供了一系列用于操作URL的功能。

## Get

urllib的`request`模块可以非常方便地抓取URL内容，也就是发送一个GET请求到指定的页面，然后返回HTTP的响应：

```python
with request.urlopen('https://api.map.baidu.com/location/ip') as f:
    data = f.read()
    print(f.status)
    print(f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))
```

## Post

如果要以POST发送一个请求，只需要把参数`data`以bytes形式传入。 例如

```python
reqData = login_data = parse.urlencode([
    ('ak', 'xxxxx'),
    ('ip', '117.131.61.81')
])

req = request.Request('https://api.map.baidu.com/location/ip')
req.add_header('user-agent', 'chrome')
with request.urlopen(req, data=reqData.encode('utf-8')) as f :
    data = f.read()
    print(f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))
```
## Handler

如果还需要更复杂的控制，比如通过一个Proxy去访问网站，我们需要利用`ProxyHandler`来处理，示例代码如下：

# XML

操作XML有两种方法：DOM和SAX。DOM会把整个XML读入内存，解析为树，因此占用内存大，解析慢，优点是可以任意遍历树的节点。SAX是流模式，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件。

正常情况下，优先考虑SAX，因为DOM实在太占内存。

# HTMLParser
Python提供了HTMLParser来非常方便地解析HTML，只需简单几行代码



