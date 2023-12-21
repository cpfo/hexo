---
title: python进程和线程
categories: [技术]
tags: [python]
date: 2023-12-19 15:34:57
---

多任务操作系统，可以同时执行多个任务，单核CPU是通过进程调度，多个任务交替执行的。

一个任务就是一个进程，进程是操作系统分配资源的基本单位，具有独立的内存空间和资源，进程可以看做一个独立的程序，比如微信，qq等，一个进程可以包含多个线程，至少一个线程，线程是进程内的执行单位，共享进程的资源。

<!-- more -->

# 多进程

## fork

Unix/Linux 提供了 `fork()` 系统调用，调用一次返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回`0`，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。

Windows没有 `fork` 调用。

## multiprocessing

`multiprocessing` 模块是跨平台版本的多进程模块。

`multiprocessing`提供了一个`Process`类来代表一个进程对象。

```python
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('子进程', ))
    print('启动子进程')
    p.start()
    p.join()
    print('子进程执行结束')
```

创建子进程时，只需要传入一个执行函数和函数的参数，创建一个`Process`实例，用`start()`方法启动，这样创建进程比`fork()`还要简单。

`join()`方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

## Pool

如果要启动大量的子进程，可以用进程池的方式批量创建子进程：

```python
from multiprocessing import Pool, Process
import os,time,random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 5)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(3)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```

解读：
对`Pool`对象调用`join()`方法会等待所有子进程执行完毕，调用`join()`之前必须先调用`close()`，调用`close()`之后就不能继续添加新的`Process`了。

由于`Pool`的默认大小是CPU的核数。如果任务数量大于Pool的大小，就需要等前面的任务执行完后才能执行后面的任务。

## 子进程
很多时候，子进程并不是自身，而是一个外部进程。我们创建了子进程后，还需要控制子进程的输入和输出。

`subprocess`可以让我们方便的启动子进程，并控制其输入和输出。

如果子进程还需要输入，可以使用 `communicate()` 输入

```python
import subprocess

print('nslookup www.baidu.com')
r = subprocess.call(['nslookup', 'www.baidu.com'])
print('Exit code:', r)
```

## 进程间通信
`Process`之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python的`multiprocessing`模块包装了底层的机制，提供了`Queue`、`Pipes`等多种方式来交换数据。

我们以`Queue`为例，在父进程中创建两个子进程，一个往`Queue`里写数据，一个从`Queue`里读数据：

```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in range(1,10):
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```

在Unix/Linux下，`multiprocessing`模块封装了`fork()`调用，使我们不需要关注`fork()`的细节。

由于Windows没有`fork`调用，因此，`multiprocessing`需要“模拟”出`fork`的效果，父进程所有Python对象都必须通过**pickle序列化**再传到子进程去，所以，如果multiprocessing在Windows下调用失败了，要先考虑是不是pickle失败了。

# 多线程

多任务可以由多进程完成，也可以由一个进程内的多线程完成。Python的线程是真正的Posix Thread，而不是模拟出来的线程

## Thread
Python的标准库提供了两个模块：`_thread`和`threading`，`_thread`是低级模块，`threading`是高级模块，对`_thread`进行了封装。绝大多数情况下，我们只需要使用`threading`这个高级模块。

启动一个线程就是把一个函数传入并创建`Thread`实例，然后调用`start()`开始执行(和java有点像)：

```python
def task_run():
    print('当前线程名称是 %s' % threading.current_thread().name)
    time.sleep(random.random() * 5)
    print('线程 % s 在执行任务' % threading.current_thread().name)
    print('线程 %s 执行结束' % threading.current_thread().name)

print('当前线程是 %s ' % threading.current_thread().name)

t = threading.Thread(target=task_run, name='子线程')
t.start()
t.join()
print('线程 %s 执行结束 ' % threading.current_thread().name)

```

任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程，Python的`threading`模块有个`current_thread()`函数，它永远返回当前线程的实例。主线程实例的名字叫`MainThread`，子线程的名字在创建时指定，名字仅仅在打印时用来显示，完全没有其他意义，如果不起名字Python就自动给线程命名为Thread-1，Thread-2...

## Lock

线程共享变量，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容改乱了。

高级语言的一条语句在CPU执行时是若干条语句，即使一个简单的计算

> balance = balance + n

也分两步：

1. 计算balance + n，存入临时变量中；
2. 将临时变量的值赋给balance。

也就是可以看成：

```python
x = balance + n
balance = x
```

两个线程同时一存一取，就可能导致余额不对，所以，我们必须确保一个线程在修改balance的时候，别的线程一定不能改。

如果我们要确保`balance`计算正确，就要给`change_it()`修改的地方上一把锁，当某个线程开始执行`change_it()`时，我们说，该线程因为获得了锁，因此其他线程不能同时执行`change_it()`，只能等待，直到锁被释放后，获得该锁以后才能改。

由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁，所以，不会造成修改的冲突。创建一个锁就是通过`threading.Lock()`来实现：

当多个线程同时执行`lock.acquire()`时，只有一个线程能成功地获取锁，其他线程需要继续等待，执行完后需要释放锁，可以放到`try...finally`代码块中来确保一定释放。

## 多核CPU

启动与CPU核心数量相同的N个线程，在4核CPU上可以监控到CPU占用率仅有102%，也就是仅使用了一核。

但是用C、C++或Java来改写相同的死循环，直接可以把全部核心跑满，4核就跑到400%，8核就跑到800%，为什么Python不行呢？

因为Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：`Global Interpreter Lock`，任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

在Python中，可以使用多线程，但不要指望能有效利用多核。

Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响。

# ThreadLocal
多线程环境下，尽量使用局部变量，全局变量的修改操作需要加锁。但是局部变量在函数调用时传递比较麻烦。

正常情况下，需要函数一层层往下传递，比较麻烦，也可以使用全局变量dict来保存，使用线程信息作为key，代码看起来比较乱。可以使用 `ThreadLocal`

```python
lc = threading.local()

def print_info():
    name = lc.user
    print('当前线程 %s 输出用户名 %s' % (threading.current_thread(), name))

def set_info(name):
    lc.user = name
    time.sleep(random.random() * 3)
    print_info()

for i in range(10):
    t = threading.Thread(target=set_info, args=(i,), name='thread-' + str(i))
    t.start()

```

可以理解为全局变量`lc`是一个`dict`，不但可以用`lc.user`，还可以绑定其他变量，如`lc.job`等等。

`ThreadLocal`最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

一个`ThreadLocal`变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。`ThreadLocal`解决了参数在一个线程中各个函数之间互相传递的问题。

# 进程 vs. 线程

**线程切换**

多任务线程切换。

**计算密集型 vs. IO密集型**

计算密集型主要消耗CPU资源，减少线程数，减少线程切换。 IO密集型主要消耗在IO等待，任务数提升，可以提高CPU的利用率。如常见的web应用。

**异步IO**

CPU和IO有巨大的速度差异，一个任务在执行的过程中大部分时间都在等待IO操作，单进程单线程模型会导致别的任务无法并行执行。

现在操作系统一般都支持异步IO，如果充分利用操作系统提供的异步IO支持，就可以用单进程单线程模型来执行多任务，这种全新的模型称为事件驱动模型。Nginx就是支持异步IO的Web服务器，它在单核CPU上采用单进程模型就可以高效地支持多任务。

对应到Python语言，单线程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。

# 分布式进程

在Thread和Process中，应当优选Process，因为Process更稳定，而且，Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。

Python的`multiprocessing`模块不但支持多进程，其中`managers`子模块还支持把多进程分布到多台机器上。一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。由于`managers`模块封装很好，不必了解网络通信的细节，就可以很容易地编写分布式多进程程序。

`QueueManager`的使用。可以把任务分布到多台机器上进行就算。




