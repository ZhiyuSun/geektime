# JVM

## 谈谈你对Java的理解

- 平台无关性
- GC
- 语言特性
- 面向对象
- 类库
- 异常处理

## 平台无关性如何实现,compile once run anywhere

javac xxx.java
javac编译，生成字节码

java xxx.class
jvm解析，转换成特定平台的执行命令

javac -p xxx.class
反编译

java源码首先被编译成字节码，再由不同平台的JVM进行解析，java语言在不同的平台上运行时不需要进行重新编译，java虚拟机在执行字节码的时候，把字节码转换成具体平台上的机器指令

## JVM如何加载.class文件

java虚拟机

class files ——> class loader
- class loader: 依据特定格式，加载class文件到内存
- execution engine: 对命令进行解析
- native interface：融合不同开发语言的原生库为Java所用
- runtime data area：JVM内存空间结构模型

## 谈谈反射

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法
对于任意一个对象，都能够调用它的任意方法和属性，这种动态获取信息以及动态调用对象方法的功能称为Java语言的反射机制

写一个反射的例子

classforName
newInstance
getDeclaredMethod
setAccessible
xxx.invoke(r, args)


## 类从编译到执行的过程

- 编译器将Robot.java文件编译为Robot.class字节码文件
- ClassLoader将字节码转换为JVM中的Class<Robot>对象
- JVM利用Class<Robot>对象实例化为Robot对象

## 谈谈ClassLoader

ClassLoader在java中有着非常重要的作用，它主要工作在Class装载的加载阶段，其主要作用是从系统外部获得Class二进制数据流。
它是Java的核心组件，所有的Class都是由ClassLoader进行加载的，ClassLoader负责通过将Class文件里的二进制数据流装载进系统，然后交给Java虚拟机进行连接，初始化等操作

## ClassLoader的种类

BootStrapClassLoader，C++编写，加载核心库java
ExtClassLoader:java编写，加载扩展库javax.*
AppClassLoader:java编写，加载程序所在目录
自定义CLassLoader：Java编写，定制化加载

MyClassLoader自定义类的编写（秦金卫老师也讲到了）
MyClassLoader extends ClassLoader

## 谈谈类加载器的双亲委派机制

自底向上检查类是否已经加载
自顶向下尝试加载类

## 为什么要使用双亲委派机制去加载类

- 避免多份同样字节码的加载

## 类的加载方式

隐式加载：new
显式加载：loadClass,forName等

## laodClass和forName的区别

类的装载过程

加载
- 通过CLassLoader加载class文件字节码，生成Class对象
链接
- 校验：检查加载的class的正确性和安全性
- 准备：为类变量分配存储空间并设置类变量初始值
- 解析：JVM将常量池内的符号引用转换为直接引用(可选)
初始化
- 执行类变量赋值和静态代码块

## java的内存模型

地址空间的划分
- 内核空间
- 用户空间

jvm内存模型-jdk8
线程私有：程序计数器，虚拟机栈，本地方法栈
所有线程共享：metaspace,java堆

程序计数器
- 当前线程所执行的字节码行号指示器（逻辑）
- 改变计数器的值来选取下一条需要执行的字节码指令
- 和线程是一对一的关系即“线程私有”
- 对Java方法技术，如果是native方法则计数器为undefined
- 不会发生内存泄露

java虚拟机栈（Stack）
- Java方法执行的内存模型
- 包含多个栈帧

局部变量表和操作数栈
- 局部变量表：包含方法执行过程中的所有变量
- 操作数栈：入栈、出栈、复制、交换、产生消费变量

递归会引发java.lang.StackOverflowError异常
递归过深，栈帧数超出虚拟栈深度

虚拟机栈过多会引发java.lang.OutOfMemoryError异常

本地方法栈
- 与虚拟机栈相似，主要作用于标注了native的方法

元空间（metaSpace）与永久代（PermGen）的区别
- 元空间使用本地内存，而永久代使用的是jvm的内存
- 字符串常量池存在永久代中，容易出现性能问题和内存溢出
- 类和方法的信息大小难以确定，给永久代的大小指定带来困难
- 永久代会为GC带来不必要的复杂性
- 方便HotSpot与其他JVM如Jrockit的集成

Java堆
- 对象实例的分配区域
- GC管理的主要区域

性能调优参数-Xms，-Xmx,-Xss的含义
- Xss：规定了每个线程虚拟机栈的大小（通常256k）
- Xms:堆的初始值
- Xmx:堆能达到的最大值

## java内存模型中堆和栈的区别

联系：引用对象、数组时，栈里定义变量保存堆中目标的首地址
区别
- 管理方式：栈自动释放，堆需要GC
- 空间大小：栈比堆小
- 碎片相关：栈产生的碎片远小于堆
- 分配方式：栈支持静态和动态分配，而堆仅支持动态分配
- 效率：栈的效率比堆高

元数据：类
堆：对象

## 为什么会出现金三银四的现象

- 年终奖已发放，调薪情况已经确定
- 卧薪尝胆迎来阶段性成果
- 公司增加员工名额，弥补劳动力缺口

## 金三银四是找工作的最佳时期吗

那倒未必

优势：供选择的公司多，机会多
劣势：人才供应量旺盛。成为备胎的概率大增，获取offer时间较慢。若无明显竞争力，薪资涨幅相对不会很高

## 相对容易找到工作的时期

临近年末的时候

大多数人不愿意在这个时候跳槽，导致远低于求
门槛变低，通过率变高
拿到offer的时间会相对变短

## 年末跳槽的好与坏

真的会失去年终奖吗？那倒未必

通过薪资等形式补偿
谈薪方式一：直接和猎头表达
谈薪方式二：说出顾虑

优势一：薪水涨幅空间可能会大
优势二：能去心仪的公司的概率相对较大
劣势一：充当救火英雄的面大

只要底子过硬，任何时候都属于稀缺





