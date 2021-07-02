# Python的GIL

为什么Python多线程是伪多线程？

## GIL的存在

### python解释器概述

编译型语言：C/C++，Golang
解释器语言：Python,PHP,Js

### 程序翻译与程序解释

图

Python解释器概述

Python语言——作为输入——Python解释器——字节码执行

Python解释器：
CPython，Jython, Pypy, IronPython

## 初探Python GIL

临界资源
- 临界资源指的是一些虽作为共享资源却又无法同时被多个线程共同访问的共享资源。当有进程在使用临界资源时，其他进程必须依据操作系统的同步机制等待占用进程释放该共享资源才可重新竞争使用共享资源。

锁的作用：挡住了进程或线程使用这些临界资源。


GIL锁是什么
- Global Interpreter Lock:全局解释器锁
- GIL锁存在于CPython版本的解释器中
- 使用GIL的解释器**只允许同一时间执行一个线程**
- （Cpython里，GIL类似互斥锁）

有GIL锁，以至于每次只有一个核在运行线程，图

Python的GIL锁就是互斥锁，保证在某一个时刻，只有一个线程在真正的运行

## 体验GIL锁

countDown的例子
多线程，时间反而长了

生产者-消费者，看CPU占用率，
Python只有一个核在运行
C++有两个核在运行

结论：
Python多线程是伪多线程，无法利用多核资源，并且某些场景还会因为上下文切换带来额外的资源损耗

## GIL的细节、作用、影响

### GIL的作用

容器方法：
append clear copy count index
其他容器：
tuple dict

Python解释器大部分逻辑都需要加锁

偷懒的实现（一把全局锁）

GIL锁在Python体系中无法剥离，尽管多核CPU已经是家常便饭

### GIL总结

GIL保证了Python解释器的正确运行
GIL简单粗暴的设计使得Python语言茁壮成长
GIL锁显然不是一个优雅的设计

### GIL与Python历代版本

2005年前，GIL锁影响并不大
但现在，GIL已经很难移除了

如果去掉GIL，就会对临界资源不停的加锁解锁，单线程能力受到影响

我的理解：全局解释器锁把对临界资源的加锁解锁给包起来了

## Python多线程切换的过程

线程运行要申请GIL锁

Tick——字节码片段
- Python虚拟机指令
- 100ticks将进行一次检查

线程执行100个ticks时就会强制它释放GIL锁，解决了一个线程独占GIL锁

因为ticks不是均匀时间的，所以Python3的解释器进行了改进，改成了5ms限制，同样有主动释放的过程

IO过程也会释放GIL锁

在网络/存储时都有IO的过程，在等待的时候，会释放GIL锁，等待IO完成，在去申请GIL锁

- 多线程运行需要竞争申请GIL
- Python虚拟机强制释放GIL
- IO过程自动释放GIL

## 解释型语言GIL概况一览

Ruby解释器拥有和Python类似的GIL锁
PHP默认不支持多线程，安装额外C扩展以支持多线程
Lua只支持单线程，可以使用多线程
Peal多线程在linux是通过多个Process实现
Shell没有线程的概念
JavaScript是单线程的，异步接口是很成熟的
