---
title: python的IO编程
categories: [技术]
tags: [python]
date: 2023-12-18 17:45:09
---

IO在计算机中指Input/Output，也就是输入和输出。如磁盘IO，网络IO等。Stream（流）是一个很重要的概念，输入流和输出流。

由于CPU和内存的速度远远高于外设的速度，所以，在IO编程中，就存在速度严重不匹配的问题。可以分为同步和异步处理。

异步IO又包含了回调模模式和轮询模式等，在网络IO编程中比较常用。

<!-- more -->

# 文件读写

读写文件是最常见的IO操作。Python内置了读写文件的函数，用法和C是兼容的。

操作系统不允许普通程序直接操作磁盘，读写文件就是请求操作系统打开一个文件对象(通常称为文件描述符)，然后通过操作系统的接口进行读写操作。

## 读文件

读文件，用Python内置的`open()`函数，传入文件名和标示符

```shell
>>> f=open('D:/imgUrl.txt', 'r')
>>>
>>> f=open('D:/imgUrl.txtaa', 'r')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileNotFoundError: [Errno 2] No such file or directory: 'D:/imgUrl.txtaa'
>>>
```

标示符`'r'`表示读，如果文件不存在，`open()`函数就会抛出一个`IOError`的错误，并且给出错误码和详细的信息告诉你文件不存在

调用`read()`方法可以一次把文件内容读取到内存中，用一个`str`对象表示。

最后一定要调用`close()`关闭文件，因为文件对象会占用操作系统的资源，并且操作系统同一时间能打开的文件数量也是有限的。和java一样。

文件读取出错后，后面的close就不会执行了。所以，为了保证无论是否出错都能正确地关闭文件，我们可以使用`try ... finally`

```python
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```

但是每次都这样写太麻烦， Python引入了`with`语句来自动帮我们调用`close()`方法

```python
with open('/path/to/file', 'r') as f:
    print(f.read())
```

这和`try ... finally`是一样的，但是代码更简洁。

调用`read()`会一次将文件内容读取到内存中，如果文件过大，内存就爆了，可以使用分治的方法，每次读取一部分内容。反复调用`read(size)`方法，每次最多读取`size`个字节的内容

另外，调用`readline()`可以每次读取一行内容，调用`readlines()`一次读取所有内容并按行返回`list`。因此，要根据需要决定怎么调用。

如果文件很小，`read()`一次性读取最方便；如果不能确定文件大小，反复调用`read(size)`比较保险；如果是配置文件，调用`readlines()`最方便：

可以使用 `line.strip()` 把末尾的`\n` 换行符去掉

例子

```python
# 读文件遍历 
with(open('D:/imgUrl.txt', 'r')) as f:
    # 直接迭代文件对象
    for line in f:
        #print(len(line))
        pass

# 一次全部读入内存
with(open('D:/imgUrl.txt', 'r')) as f:
    l = f.readlines()
    print(len(l))
    for s in l:
        pass
        #print(s.strip())

# 按行读取
with(open('D:/imgUrl.txt', 'r')) as f:
    line = f.readline()
    while line:
        #print(line.strip())
        line = f.readline()

# 按size读取
with(open('D:/imgUrl.txt', 'r')) as f:
    size = 50
    data = f.read(size)
    while data:
        print(data)
        data = f.read(size)
```

## file-like Object

像`open()`函数返回的这种有个`read()`方法的对象，在Python中统称为file-like Object。除了file外，还可以是内存的字节流，网络流，自定义流等等。file-like Object不要求从特定类继承，只要写个read()方法就行。

`StringIO` 就是在内存中创建的file-like Object，常用作临时缓冲。

## 二进制文件

上面的默认都是读取的`utf-8`编码的文本文件，如果是二进制文件，比如图片等，需要用`'rb'` 模式打开文件。

```shell
>>> f = open('D:/123.png', 'rb')
>>> f.readline()
b'\x89PNG\r\n'
>>>
```

## 字符编码

要读取非UTF-8编码的文本文件，需要给`open()`函数传入`encoding`参数

```shell
>>> f=open('D:/gb2312-test.txt', 'r', encoding='gb2312', errors='ignore')
>>> f.readline()
'中文编码测试\n'
```

如果有一些非法的编码，可能会报错`UnicodeDecodeError`，可以指定`errors`参数忽略错误

## 写文件

写文件和读文件是一样的，唯一区别是调用`open()`函数时，传入标识符`'w'`或者`'wb'`表示写文本文件或写二进制文件：

```python
f = open('D:/write-test.txt', 'w')
f.write('哈哈哈哈\n')
f.close()

```

可以反复调用`write()`来写文件，但是一定要调用`f.close()`方法来关闭文件，否则可能发会导致要写入的数据只写了一部分。只有调用`close()`方法时，操作系统才保证把没有写入的数据全部写入磁盘。

所以还是使用`with`最保险

要写入特定编码的文本文件，可以给`open()`函数传入`encoding`参数

以 `'w'`模式写文件时，每次写入会覆盖，可以将`'w'` 改为 `'a'` 来使用追加模式写入。

# StringIO和BytesIO


## StringIO

很多时候，数据读写不一定是文件，也可以在内存中读写。

StringIO就是在内存中读写str。要把str写入StringIO，我们需要先创建一个StringIO，然后，像文件一样写入即可

```shell
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('大')
1
>>> f.write('\n')
1
>>> f.write('家好')
2
>>> f.getvalue()
'大\n家好'
```

`getvalue()`方法用于获得写入后的str。

要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取

```python
f = StringIO('大\n家\n好')
while True:
    s = f.readline()
    if s == '':
        break
    print(s.strip())

```

## BytesIO

StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

写入的不是str，而是经过UTF-8编码的bytes

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取

```python
f = BytesIO()
f.write('中文'.encode('utf-8'))
print(f.getvalue())
print(f.getvalue().decode('utf-8'))

# 使用bytes初始化
f = BytesIO('中文'.encode('utf-8'))
print(f.read())
```

# 操作文件和目录

内置的`os`模块可以直接调用操作系统提供的接口操作文件和目录。

```shell
>>> import os
>>> os.name
'posix'
>>> os.uname()
('Linux', 'dev', '3.10.0-1160.90.1.el7.x86_64', '#1 SMP Thu May 4 15:21:22 UTC 2023', 'x86_64')
>>> 

>>> os.name
'nt'
```

如果是`posix`，说明系统是`Linux`、`Unix`或`Mac OS X`，如果是`nt`，就是`Windows`系统。

要获取详细的系统信息，可以调用`uname()`函数, windows系统不支持该函数。`os`模块的某些函数是跟操作系统相关的

## 环境变量

`os.environ` 可以查看全部的环境变量。要获取某个环境变量的值，可以调用`os.environ.get('key')`：

## 操作文件和目录

操作文件和目录的函数一部分放在os模块中，一部分放在os.path模块中。

把两个路径合成一个时，不要直接拼字符串，而要通过`os.path.join()`函数，这样可以正确处理不同操作系统的路径分隔符。

拆分路径的时候，要使用`os.path.split()`，这样可以把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名。

`os.path.splitext()`可以直接得到文件的扩展名。

这些合并、拆分路径的函数并不要求目录和文件要真实存在，它们只对字符串进行操作。

`os.rename()` 对文件重命名。`os.remove` 删除文件。

# 序列化



程序运行的过程中，所有的变量都是在内存中。在程序运行结束后，变量就会被操作系统回收。

把变量从内存中变成可存储或传输的过程称之为序列化，在Python中叫pickling，在其他语言中也被称之为serialization，marshalling，flattening等等，都是一个意思

序列化之后，就可以把序列化的内容写入到磁盘，或者通过网络传输。

把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。

## pickle

Python提供了`pickle`模块来实现序列化。

`pickle.dumps()`方法把任意对象序列化成一个`bytes`，然后，就可以把这个`bytes`写入文件。

或者用另一个方法`pickle.dump()`直接把对象序列化后写入一个file-like Object：

当我们要把对象从磁盘读到内存时，可以先把内容读到一个`bytes`，然后用`pickle.loads()`方法反序列化出对象，也可以直接用`pickle.load()`方法从一个file-like Object中直接反序列化出对象。

通过这种方式反序列化的变量内容相同，但是实际上不是同一个变量了。

Pickle的序列化只能用于python，可能python的不同版本也不兼容，也不能跨语言。所以不能保存重要的数据。

```python
l = list(range(1,10))
print(pickle.dumps(l))

# 序列化到文件中
f = open('D:/tmpdump.txt', 'wb')
pickle.dump(l, f)
f.close()

# 反序列化 先读取成bytes
f = open('D:/tmpdump.txt', 'rb')
b = f.read()
data = pickle.loads(b)
print(type(data))
print(data)

#直接从文件反序列化
f = open('D:/tmpdump.txt', 'rb')
d = pickle.load(f)
print(d)
```

结果

```shell
b'\x80\x04\x95\x17\x00\x00\x00\x00\......省略....'
<class 'list'>
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## JSON

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但更好的方法是序列化为JSON。

JSON和Python内置的数据类型对应如下

|  JSON类型   | Python类型  |
|  :---  | :---  |
| {}  | dict |
| []  | list |
| "string" | str |
| 12.3  | int或float |
| true/false | True/False |
| null | None |

Python内置的json模块提供了非常完善的Python对象到JSON格式的转换。

```shell
>>> d = dict(name='张三', age=20, height=130.5, interest=['篮球', '唱歌'])
>>>
>>> d
{'name': '张三', 'age': 20, 'height': 130.5, 'interest': ['篮球', '唱歌']}
>>> json.dumps(d)
'{"name": "\\u5f20\\u4e09", "age": 20, "height": 130.5, "interest": ["\\u7bee\\u7403", "\\u5531\\u6b4c"]}'

>>> json.dumps(d,ensure_ascii=False)
'{"name": "张三", "age": 20, "height": 130.5, "interest": ["篮球", "唱歌"]}'
>>>
>>> s=json.dumps(d)
>>> json.loads(s)
{'name': '张三', 'age': 20, 'height': 130.5, 'interest': ['篮球', '唱歌']}
>>>
>>>
```

`dumps()`方法返回一个`str`，内容是标准的json，类似的 `dump()`方法可以直接把json写入到`file-like Object`

使用`loads()`方法把json字符串反序列化，或者使用`load()`方法从`file-like Object`中读取字符串并反序列化。

## JSON进阶

`dict`对象可以直接序列化为JSON的`{}`，不过，很多时候，我们更喜欢用class表示对象，比如定义Student类，然后序列化：

```python
import json

class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)
print(json.dumps(s))
```

运行时，会报`TypeError`，是因为`Student`对象不是一个可序列化为JSON的对象。

`dumps()`函数有很多可选参数，可以用来定制json序列化。

可选参数`default`就是把任意一个对象变成一个可序列为JSON的对象，我们只需要为`Student`专门写一个转换函数，再把函数传进去即可

```python
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }

print(json.dumps(s, default=student2dict))
```

不过，如果遇到其他类的实例，依然无法序列化，可以默认把任意`class`的实例变为`dict`

> print(json.dumps(s, default=lambda obj: obj.__dict__))

因为通常`class`的实例都有一个`__dict__`属性，它就是一个`dict`，用来存储实例变量。也有少数例外，比如定义了`__slots__`的class。

如果我们要把JSON反序列化为一个`Student`对象实例，`loads()`方法首先转换出一个`dict`对象，然后，我们传入的`object_hook`函数负责把`dict`转换为`Student`实例

```shell
>>> s = '{"name": "张三", "age": 20, "height": 130.5, "interest": ["篮球", "唱歌"]}'
>>> stu = json.loads(s, object_hook=dict2student)
>>> print(stu)
<__main__.Student object at 0x000002615C4962A0>
>>>
>>> print(json.dumps(stu, default=lambda obj: obj.__dict__, ensure_ascii=False))
{"name": "张三", "age": 20, "score": 130.5}
>>>
```

`ensure_ascii`用于指定在对JSON进行编码时是否对非ASCII字符进行转义，确保生成的JSON字符串中只包含ASCII字符。

`ensure_ascii`参数默认值为True，表示会对非ASCII字符进行转义，将其表示为`\uXXXX`的形式。 如果将`ensure_ascii`参数设置为`False`，则表示不对非ASCII字符进行转义。非ASCII字符将以原样包含在生成的JSON字符串中。

通常情况下，需要保持默认值，这样可以确保生成的JSON字符串是有效的ASCII字符串，可以在不同的系统之间进行传输和解析。

