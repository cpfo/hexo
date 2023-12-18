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





