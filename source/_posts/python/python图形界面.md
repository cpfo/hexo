---
title: python图形界面
categories: [技术]
tags: [python]
date: 2024-01-09 15:24:57
---

主要介绍图形界面的操作。包括Tkinter和海龟绘图。

<!-- more -->

# Tkinter

Python支持多种图形界面的第三方库，包括

* Tk
* wxWidgets
* Qt
* GTK

等等。

Python自带的库是支持Tk的Tkinter，使用Tkinter，无需安装任何包，就可以直接使用。

我们编写的Python代码会调用内置的Tkinter，Tkinter封装了访问Tk的接口；

Tk是一个图形库，支持多个操作系统，使用Tcl语言开发；

Tk会调用操作系统提供的本地GUI接口，完成最终的GUI。

所以，我们的代码只需要调用Tkinter提供的接口就可以了。


使用Tkinter，第一步是导入Tkinter包的所有内容

> from tkinter import *

第二步是从`Frame`派生一个`Application`类，这是所有Widget的父容器：

```python

from tkinter import *
from tkinter import messagebox
class Application(Frame):
    def __init__(self, master=None):
        Frame.__init__(self, master)
        self.pack()
        self.createWidgets()

    def createWidgets(self):
        self.helloLabel = Label(self, text='这是gui界面的内容')
        self.helloLabel.pack()
        self.inputText = Entry(self)
        self.inputText.pack()
        self.submitBtn = Button(self, text='提交', command=self.hello)
        self.submitBtn.pack()
        self.quitButton = Button(self, text='退出', command=self.quit)
        self.quitButton.pack()

    def hello(self):
        info = self.inputText.get() or '你好'
        messagebox.showinfo('弹窗', 'Hello, %s' % info)
```

在GUI中，每个Button、Label、输入框等，都是一个Widget。Frame则是可以容纳其他Widget的Widget，所有的Widget组合起来就是一棵树。

pack()方法把Widget加入到父容器中，并实现布局。pack()是最简单的布局，grid()可以实现更复杂的布局。

在`createWidgets()`方法中，我们创建一个Label和一个Button，当Button被点击时，触发`self.quit()`使程序退出。

第三步，实例化Application，并启动消息循环：

```python
app = Application()
# 设置窗口标题:
app.master.title('gui界面')
# 主消息循环:
app.mainloop()
```

GUI程序的主线程负责监听来自操作系统的消息，并依次处理每一条消息。因此，如果消息处理非常耗时，就需要在新线程中处理。

# 海龟绘图

在1966年，Seymour Papert和Wally Feurzig发明了一种专门给儿童学习编程的语言——LOGO语言，它的特色就是通过编程指挥一个小海龟（turtle）在屏幕上绘图。

海龟绘图（Turtle Graphics）后来被移植到各种高级语言中，Python内置了turtle库，基本上100%复制了原始的Turtle Graphics的所有功能。

