---
title: python常用内建模块2
categories: [技术]
tags: [python]
date: 2023-12-25 18:40:13
---

本章主要介绍 argparse， bass64， struct，hashlib， hmac的使用。

<!-- more -->

# argparse

内置的`sys.argv`保留了完整的参数列表，我们可以从中解析出需要的参数，但是如果参数复杂点，解析起来就很麻烦了，可以使用`argparse`

假设需要编写一个mysql命令行登录程序，需要的参数如下
> mysql -h localhost --port 3306 -u root -p password


```python
import argparse,sys

parser = argparse.ArgumentParser(
    prog='命令行登录',
    description='命令行登录mysql数据库',
    epilog='Copyright,2023'
)

# 定义关键字参数: ，此处不能用 -h简写，和 argparse的-h帮助参数冲突了。
parser.add_argument('--host', default='localhost')
# 此参数必须为int类型:
parser.add_argument('--port', default='3306', type=int)
# 允许用户输入简写的-u:
parser.add_argument('-u', '--user', required=True)
parser.add_argument('-p', '--password', required=True)

# 解析参数:
args = parser.parse_args()

print(sys.argv)

# 打印参数:
print('parsed args:')
print(f'host = {args.host}')
print(f'port = {args.port}')
print(f'user = {args.user}')
print(f'password = {args.password}')

```

输入有效的参数，能解析出所需要的全部参数。参数不对或者缺失的话，也会有详细的提示

```shell
python argparse-test.py --host localhost --port 3306 -u root
usage: 命令行登录 [-h] [--host HOST] [--port PORT] -u USER -p PASSWORD
命令行登录: error: the following arguments are required: -p/--password

```
输入参数 `-h` 会有对应的提示信息

```shell
D:\workspace\python>python argparse-test.py -h
usage: 命令行登录 [-h] [--host HOST] [--port PORT] -u USER -p PASSWORD

命令行登录mysql数据库

options:
  -h, --help            show this help message and exit
  --host HOST
  --port PORT
  -u USER, --user USER
  -p PASSWORD, --password PASSWORD

Copyright,2023
```

只有当参数全部有效时，才会返回一个 [NameSpace](https://docs.python.org/3/library/argparse.html#argparse.Namespace) 对象，获取对应的参数就把参数名当作属性获取，非常方便

# base64

Base64是一种用64个字符来表示任意二进制数据的方法。A-Z、a-z、0-9、"+"和"/"两个特殊字符，共计64个字符。因此，在URL安全的Base64编码中，常常将"+"替换为"-"，将"/"替换为"_"，以避免引起冲突。

Base64编码将每3个字节的数据转换为4个字符，因此每个字符提供了6个比特位的信息

Base64对二进制数据进行处理，每3个字节一组，一共是`3x8=24bit`，划为4组，每组正好6个bit：

这样我们得到4个数字作为索引，然后查表，获得相应的4个字符，就是编码后的字符串。

示例参考 [example](https://www.base64decode.org/)  页面的example部分。`Man`转换为base64结果为`TWFu`

![示例图](https://pic1.zhimg.com/80/v2-3b7c9b9441623c1c6b5b6099bb897cbc_720w.png) 

Base64编码会把3字节的二进制数据编码为4字节的文本数据，长度增加33%，好处是编码后的文本数据可以在邮件正文、网页等直接显示

如果要编码的二进制数据不是3的倍数，最后会剩下1个或2个字节怎么办？Base64用`\x00`字节在末尾补足后，再在编码的末尾加上1个或2个`=`号，表示补了多少字节，解码的时候，会自动去掉。

```shell
>>> e64 = base64.b64encode('hahaha\x00'.encode('utf-8'))
>>> print(e64)
b'aGFoYWhhAA=='
>>> d64 = base64.b64decode(b64)
>>> print(d64)
b'hahaha'
>>> print(d64.decode('utf-8'))
hahaha
>>>
```

`url safe`url安全的base64编码。把字符`+`和`/`分别变成`-`和`_`。使用`base64.urlsafe_b64encode()`和`base64.urlsafe_b64decode()`

由于`=`字符也可能出现在Base64编码中，但=用在URL、Cookie里面会造成歧义，所以，很多Base64编码后会把`=`去掉

去掉`=`后怎么解码呢？因为Base64是把`3`个字节变为`4`个字节，所以，Base64编码的长度永远是4的倍数，因此，需要加上=把Base64字符串的长度变为4的倍数，就可以正常解码了。

# struct字节和二进制的转换

准确地讲，Python没有专门处理字节的数据类型。但由于b'str'可以表示字节，所以，字节数组＝二进制str。Python提供了一个`struct`模块来解决`bytes`和其他二进制数据类型的转换

`struct`的`pack`函数把任意数据类型变成`bytes`

pack的第一个参数是处理指令，`'>I'`的意思是：

`>` 表示字节顺序是big-endian，也就是网络序，`I`表示4字节无符号整数。

后面的参数个数要和处理指令一致。

`unpack`把`bytes`变成相应的数据类型

```shell
>>> import struct
>>> struct.pack('>I', 2234324)
b'\x00"\x17\xd4'
>>>
>>> struct.unpack('>I', b'\x00"\x17\xd4')
(2234324,)
>>> struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80')
(4042322160, 32896)
>>>
```

根据`>IH`的说明，后面的`bytes`依次变为`I`：4字节无符号整数和`H`：2字节无符号整数。

所以，尽管Python不适合编写底层操作字节流的代码，但在对性能要求不高的地方，利用struct就方便多了。

struct支持的数据类型参考[format-characters](https://docs.python.org/3/library/struct.html#format-characters) , 

# hashlib

**摘要算法简介**

Python的hashlib提供了常见的摘要算法，如MD5，SHA1等等。

摘要算法，通过摘要函数，计算出固定长度的摘要，用于验证数据，防篡改。

md5 和 sha1

```shell
>>> import hashlib
>>> md5=hashlib.md5()
>>> md5.update('test'.encode('utf-8'))
>>> md5
<md5 _hashlib.HASH object @ 0x0000020B0560E8F0>
>>> print(md5.hexdigest())
098f6bcd4621d373cade4e832627b4f6
>>>
>>> sha1 = hashlib.sha1()
>>> sha1.update('test'.encode('utf-8'))
>>> print(sha1.hexdigest())
a94a8fe5ccb19ba61c4c0873d391e987982fbbd3
>>>
```

摘要可能出现hash碰撞的情况，不同的内容计算出来的hash值相同。

**摘要算法应用**

明文密码加盐，然后计算hash值，进行比对，判断用户输入的密码是否正确

# hmac

如果`salt`是我们自己随机生成的，通常我们计算MD5时采用`md5(message + salt)`。

这实际上就是Hmac算法：Keyed-Hashing for Message Authentication。它通过一个标准算法，在计算哈希的过程中，把key混入计算过程中。

Hmac算法针对所有哈希算法都通用，无论是MD5还是SHA-1。

```shell
>>> import hmac
>>> message = b'test'
>>> key = b'hahah'
>>> h = hmac.new(key, message, digestmod='SHA1')
>>> print(h.hexdigest())
5ab4f2d179d1243d6ff9084a0ca659a99bbfcf3f
>>>
```

可见使用hmac和普通hash算法非常类似。hmac输出的长度和原始哈希算法的长度一致。需要注意传入的key和message都是bytes类型，str类型需要首先编码为bytes