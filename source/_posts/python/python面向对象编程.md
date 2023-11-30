---
title: python面向对象编程
categories: [技术]
tags: [python]
date: 2023-11-29 16:10:51
---

面向对象编程——Object Oriented Programming，简称OOP，是一种程序设计思想。

<!-- more -->

# OOP概念

OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。

面向过程的程序设计把计算机程序视为一系列的命令集合，即一组函数的顺序执行。

OOP设计把计算机程序视为一组对象的集合，而每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间传递。

下面举例说明面向过程和面向对象的在程序流程上的不同。

假设我们要处理学生的成绩表，为了表示一个学生的成绩，面向过程的程序可以用一个dict表示：

```shell
std1 = { 'name': 'Michael', 'score': 98 }
std2 = { 'name': 'Bob', 'score': 81 }

```
而处理学生成绩可以通过函数实现，比如打印学生的成绩：

```shell
def print_score(std):
    print('%s: %s' % (std['name'], std['score']))
```
如果采用OOP的设计思想，首先思考的不是程序的执行流程，而是 `Student` 这种数据类型应该被视为一个对象。这个对象拥有`name`和`score`这两个属性（Property）。如果要打印一个学生的成绩，首先必须创建出这个学生对应的对象，然后，给对象发一个`print_score`消息，让对象自己把自己的数据打印出来。

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
```
给对象发消息实际上就是调用对象对应的关联函数，我们称之为对象的方法（Method）。面向对象的程序写出来就像这样：

```python
bart = Student('Bart Simpson', 59)
lisa = Student('Lisa Simpson', 87)
bart.print_score()
lisa.print_score()
```

类（Class）和实例（Instance）, Class是一种抽象概念，比如我们定义的Class——Student，是指学生这个概念，而实例（Instance）则是一个个具体的Student，比如，Bart Simpson和Lisa Simpson是两个具体的Student。

所以，面向对象的设计思想是抽象出Class，根据Class创建Instance。

面向对象的抽象程度又比函数要高，因为一个Class既包含数据，又包含操作数据的方法。

封装，继承，多态，三大特点，和java一样。

# 类和实例

面向对象最重要的概念就是类（Class）和实例（Instance），类是抽象的模板。

在Python中，定义类是通过`class`关键字

```python
class Student(object):
    pass
```
`class`后面紧接着是类名，即`Student`，类名通常是大写开头的单词，紧接着是`(object)`，表示该类是从哪个类继承下来的，通常，如果没有合适的继承类，就使用`object`类，这是所有类最终都会继承的类。

定义好了类，就可以通过类创建出实例。创建实例是通过类名+()实现的。

```shell
>>> Student
<class '__main__.Student'>
>>> s = Student()
>>> s
<__main__.Student object at 0x000001A965ADA690>
```

实例创建后可以自由地给实例变量绑定属性。

```shell
>>> s.name = '张三'
>>> s.name
'张三'
>>>
```

由于类可以起到模板的作用，因此，可以在创建实例的时候，把一些我们认为必须绑定的属性强制填写进去。通过定义一个特殊的`__init__`方法，在创建实例的时候，就把`name`，`score`等属性绑上去：

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score
```

> 特殊方法“__init__”前后分别有两个下划线！！！

`__init__`方法的第一个参数永远是`self`，表示创建的实例本身，因此，在`__init__`方法内部，就可以把各种属性绑定到`self`，因为`self`就指向创建的实例本身。

有了`__init__`方法，在创建实例的时候，必须传入与`__init__`方法匹配的参数，但`self`不需要传，Python解释器自己会把实例变量传进去：

和普通的函数相比，在类中定义的函数只有一点不同，就是第一个参数永远是实例变量`self`，并且，调用时，不用传递该参数。

**数据封装**

封装通过将数据和对数据的操作（方法）捆绑在一起，形成一个独立的、可复用的单位，对外部隐藏内部实现细节，只暴露出简单的接口供其他代码使用。

```shell
>>> class Student(object):
...   def hello(self):
...     print('hello')
...
>>> s1 = Student()
>>> s1.hello
<bound method Student.hello of <__main__.Student object at 0x0000015DEB716B10>>
>>>
>>> s1.hello()
hello
```
通过实例变量访问方法时，如果不加括号，输出的是方法的信息，要调用方法必须要加上`()`

# 访问限制

如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线__，在Python中，实例的变量名如果以__开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问。

这样就确保外部无法随意修改对象内部的状态。

如果想要访问或者修改对象内部的数据，可以给类增加 get 和 set 方法， 和java类似。可以在方法中对参数做校验。

变量名类似`__xxx__`的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是private变量，所以，不能用`__name__`、`__score__`这样的变量名。

以一个下划线开头的实例变量名，比如`_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，这样的变量不能随意访问。

Python解释器对外把`__name`变量改成了`_Student__name`，所以，仍然可以通过`_Student__name`来访问`__name`变量，但是强烈不建议这么做。

# 继承和多态

**继承**

当我们定义一个class时，可以从某个现有的class继承，新的class称为子类（Subclass），而被继承的class称为基类、父类或超类（Base class、Super class）

继承有什么好处？最大的好处是子类获得了父类的全部功能。继承可以实现代码的重用和扩展。

**多态**

子类重写父类方法。

子类的实例的数据类型也可以看做是父类的数据类型，反过来不行。

多态的好处，当函数需要接收子类时，只需要定义时接收父类就行。调用函数时，可以传入不同的子类的实例。

> java中叫 父类引用指向子类对象？

**静态语言 vs 动态语言**

对于静态语言（例如Java）来说，如果需要传入`Animal`类型，则传入的对象必须是`Animal`类型或者它的子类，否则，将无法调用`run()`方法。

对于Python这样的动态语言来说，则不一定需要传入`Animal`类型。我们只需要保证传入的对象有一个`run()`方法就可以了：

这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。

Python的“file-like object“就是一种鸭子类型。

# 获取对象信息

**type()**

使用 `type()` 方法， 可以判断某个对象是什么类型。

```shell
>>> type(11)
<class 'int'>
>>> L=[1,2,3,4]
>>> type(L)
<class 'list'>
>>> type('aa')
<class 'str'>

>>> def func():
...   pass
...
>>>
>>> type(func())
<class 'NoneType'>
>>> type(func)
<class 'function'>
>>>
>>> type(abs)
<class 'builtin_function_or_method'>
>>>
```

`type()` 返回的是对象的Class类型。可以进行 `==` 比较，判断是否是相同的类。

**使用isinstance()**

如果有继承关系，使用`type()` 就不方便判断了， 可以使用 `isinstance()`

`isinstance()`判断的是一个对象是否是该类型本身，或者位于该类型的父继承链上。

能用`type()`判断的基本类型也可以用`isinstance()`判断。并且还可以判断一个变量是否是某些类型中的一种。

```shell
>>> isinstance([1,2,3], (list, tuple))
True
>>> isinstance([1,2,3], (dict, tuple))
False
>>> isinstance((1,2,3), (set, tuple))
True
>>>
```

所以可以优先使用 `isinstance()`来判断类型。

**使用dir()**

使用`dir()` 方法可以获取一个对象的所有属性和方法，返回的是一个包含字符串的list。

仅仅把属性和方法列出来是不够的，配合`getattr()`、`setattr()`以及`hasattr()`，我们可以直接操作一个对象的状态

如果试图获取不存在的属性，会抛出AttributeError的错误， 可以在方法中加上默认值。 例如：`getattr(obj, 'z', 404)`

也可以使用 `hasattr()`，`getattr()` 判断和获取对象的方法。

# 实例属性和类属性


给实例绑定属性的方法是通过实例变量，或者通过self变量。

```python
class Student(object):
    def __init__(self, name):
        self.name = name

s = Student('Bob')
s.score = 90
```

但是，如果`Student`类本身需要绑定一个属性呢？可以直接在class中定义属性，这种属性是类属性，归`Student`类所有

```python
class Student(object):
    name = 'Student'
```

类属性，类的所有实例都可以访问到。和java中继承很相似。

在编写程序的时候，千万不要对实例属性和类属性使用相同的名字，避免出现错误。

