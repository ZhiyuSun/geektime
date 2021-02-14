# Java内存模型——底层原理

## 什么叫“底层原理”？本章研究的内容？

Java面试的必考点，只有学会了这一章的内容，才能说你真正懂了并发
（并发的内容是衡量一个程序员能力水平的重要部分）

## 从Java代码到CPU指令

源程序——字节码（.class）——运行——不同操作系统执行
- 最开始，我们编写的Java代码是.java文件
- 在编译（javac命令）后，从刚才的*.java文件会变出一个新的Java字节码文件（.class）
- JVM会执行刚才生成的字节码文件（.class），并把字节码文件转化为机器指令
- 机器指令可以直接在CPU上执行，也就是最终的程序执行

JVM实现会带来不同的翻译，不同的CPU平台的机器指令又千差万别，无法保证并发安全的效果一致。（早期根本无法保证）
（为了解决这个问题），重点开始向下转移：转化过程的规范、原则

## JVM内存结构 VS Java内存模型 VS Java对象模型

容易混淆：三个截然不同的概念，但是很多人容易弄混

JVM内存结构，和Java虚拟机的运行时区域有关
Java内存模型，和Java的并发编程有关
Java对象模型，和Java对象在虚拟机中的表现形式有关

### JVM内存结构

class文件——类加载器（class loader）
运行时数据区（Runtime Data Area）
- 所有线程共享
    - 方法区（Method Area）：static静态变量，类信息，常量信息，永久引用
    - 堆（Heap）: 最大的一块，new的实例对象，大小动态分配
- 线程私有
    - Java栈（Java Stack）：基本数据类型，以及对象的引用。编译的时候确定了大小，并且大小不会改变
    - 本地方法栈（Native Method Stack）：native相关的栈
    - 程序计数器（Program Counter Register）：当前现场所执行到的字节码的行号数，上下文切换时会被保存
执行引擎（Execution）
本地接口（Native interface）
本地库（Native Libraries）

### Java对象模型

栈（保存对象引用）
方法区（instanceKlass）
堆（mark word 元数据指针）

- Java对象自身的存储模型
- JVM会给这个类创建一个instanceKlass，保存在方法区，用来在JVM层表示该Java类
- 当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了对象头以及实例数据

## JMM是什么

Java Memory Model

- C语言不存在内存模型的概念
- 依赖处理器，不同处理器结果不一样
- 无法保证并发安全
- 需要一个标准，让多线程运行的结果可预期

JMM是规范

- 是一组规范，需要各个JVM的实现来遵守JMM规范，以便于开发者可以利用这些规范，更方便地开发多线程程序
- 如果没有这样一个JMM内存模型来规范，那么很可能经过了不同JVM的不同规则的排序之后，导致不同的虚拟机上运行的结果不一样，那是很大的问题。

JMM是工具类和关键字的原理

- volatile syschronized lock等的原理都是JMM
- 如果没有JMM，那就需要自己指定什么时候用内存栅栏等，那是相当麻烦的，幸好有了JMM，让我们只需要用同步工具和关键字就可以开发并发程序

## 重排序

什么是重排序：在线程1内部的两行代码的实际执行顺序和代码在Java文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，它们的顺序被改变了，这就是重排序，这里被颠倒的是y=a和b=1这两行语句。

重排序的好处：提高处理速度

对比重排序前后的指令优化。重排序明显提高了处理速度

重排序的3种情况
- 编译器优化：包括JVM，JIT编译器等
- CPU指令重排：就算编译器不发生重排，CPU也可能对指令进行重排
- 内存的重排序：线程A的修改线程B却看不到，引出可见性问题

## 可见性

用volatile解决问题
强制把数据更改flush到主内存当中

为什么会有可见性问题
- CPU有多级缓存，导致读的数据过期
    - 高速缓存的容量比主内存小，但是速度仅次于寄存器，所以在CPU和主内存之间就多了Cache层
    - 线程间的对于共享变量的可见性问题不是直接由多核引起的，而是由多缓存引起的
    - 如果所有个核心都只用一个缓存，那么也就不存在内存可见性问题了
    - 每个核心都会将自己需要的数据读到独占缓存中，数据修改后也是写入到缓存中，然后等待刷入到主存中。所以会导致有些核心读取的值是一个过期的值

JMM的抽象：主内存和本地内存
- Java作为高级语言，屏蔽了这些底层细节，用JMM定义了一套读写内存数据的规范，虽然我们不再需要关心一级缓存和二级缓存的问题，但是，JMM抽象了主内存和本地内存的概念
- 这里说的本地内存并不是真的是一块给每个线程分配的内存，而是JMM的一个抽象，是对于寄存器、一级缓存、二级缓存等的抽象。

主内存和本地内存的关系
JMM有以下规定：
- 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存的拷贝
- 线程不能直接读写主内存中的变量，而是只能操作自己工作内存中的变量，然后再同步到主内存中
- 主内存是多个线程共享的，但线程不共享工作内存，如果线程间需要通信，必须借助主内存中转来完成

所有的共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是通过本地内存交换的，所以才导致了可见性问题

### Happens-Before原则

什么是Happens-Before
- Happens-Before规则是用来解决可见性问题的：在时间上，动作A发生在动作B之前，B保证能看见A，这就是Happens-Before
- 两个操作可以用Happens-Before来确定它们的执行顺序：如果一个操作Happens-Before于另一个操作，那么我们说第一个操作对于第二个操作是可见的

什么不是Happens-Before
- 两个线程没有相互配合的机制，所以代码X和Y的执行结构并不能保证总被对法看到的，这就不具备Happens-Before

Happens-Before规则有哪些？
- 单线程规则（一个线程之内，后面的代码一定能看到前面的语句发生了什么）（重排序还是可能的）（Happens-Before不影响重排序）
- **锁操作（Synchronized和Lock）**
- **volatile变量**
- 线程启动（子线程能看到主线程的结果）
- 线程join()
- 传递性：如果hb(A,B)而且hb(B，C),那么可以推出hb(A,C)
- 中断：一个线程被其他线程interrupt时，那么检查中断（isInterrputed）或者抛出InterruptedException一定能看到
- 构造方法：对象构造方法的最后一行指令Happens-Before于finalize()方法的第一行指令
- 工具类的Happens-Before原则
    - 线程安全的容器get一定能看到在此之前的put等存入动作
    - CountDownLatch
    - Semaphore
    - Future
    - 线程池
    - CyclicBarrier

演示
Happens-Before有一个原则是：如果A是对volatile变量的写操作，B是对同一个变量的读操作，那么hb(A,B)
近朱者赤：给b加了volatile，不仅b被影响，也可以实现轻量级同步
b之前的写入（对应代码b=a）对读取b后的代码（print b）都可见，所以在writerThread里对a的赋值，一定会对readerThread里的读取可见，所以这里的a即使不加volatile，只要b读到的是3，就可以由Happens-Before原则保证了读取到的都是3而不可能读取到1

### volatile关键字

如果只是沉迷于业务开发，不在业务开发中总结，业余时间提高，遇到复杂问题永远没办法解决
不去系统梳理多线程的知识点，知识是难以运用的
不推荐去996公司，没时间提高

是什么？
- volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为。
- 如果一个变量被修饰成volatile，那么JVM就知道了这个变量可能会被并发修改
- 但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的原子保护，volatile仅在很有限的场景下才能发挥作用。

不适用场景：a++
适用场合：
- boolean flag，如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。（不一定是布尔变量）
- 作为刷新之前变量的触发器（FieldVisibility）

作用：
- 可见性：读一个volatile变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个volatile属性会立即刷入到主内存。
- 禁止指令重排序优化：解决单例双重锁乱序问题

volatile和Synchronized的关系
- volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全

用volatile修正重排序
OutOfOrderExecution

小结：
- volatile修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag;或者作为触发器，实现轻量级同步
- volatile属性的读写操作都是无锁的，它不能代替synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的
- volatile只能作用于属性，我们用volatile修饰属性，这样compilers就不会对这个属性做指令重排序
- volatile提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile属性不会被线程缓存，始终从主存中读取
- volatile提供了happens-before保证，对volatile变量v的写入happens-before所有其他线程后续对v的读操作
- volatile可以使得long和double的赋值是原子的，后面马上会讲long和double的原子性

能保证可见性的措施：
- 除了volatile可以让变量保证可见性外，synchronized、Lock、并发集合，Thread.join()和Thread.start()等都可以保证可见性
- 具体看happens-before原则的规定

升华：对synchronized可见性的正确理解
- synchronized不仅保证了原子性，还保证了可见性
- synchronized不仅让被保护的代码安全，还近朱者赤（synchronized之前的代码都能被看到）

## 原子性

什么是原子性
- 一系列的操作，要么全部执行成功，要么全部不执行，不会出现执行一般的情况，是不可分割的
- ATM里取钱
- i++不是原子性的（i=1,i+1,i=2）
- 用synchronized实现原子性

Java中的原子操作有哪些？
- 除了long和double之外的基本类型（int,byte,boolean,short,char,float）的赋值操作
- 所有引用reference的赋值操作，不管是32为机器还是64位机器
- Java.concurrent.Atomic.*包中所有类的原子操作

double和long的原子性
可以看官方文档
- 问题描述：官方文档、对于64位的值的写入，可以分为两个32位的操作进行写入、读取错误、使用volatile解决
- 结论：在32位上的JVM上，long和double的操作不是原子的，但是在64位的JVM上是原子的
- 实际开发中：商用Java虚拟机中不会出现

原子操作+原子操作！=原子操作
- 简单地把原子操作组合在一起，并不能保证整体依然具有原子性
- 全同步的HashMap也不完全安全

## Java内存模型——底层原理

底层原理
自顶向下
三兄弟，jvm内存结构，java内存模型，java对象模型（对象头，示例对象）
JMM，是规范
重排序
可见性
happens-before
volatile a++是组合操作，不是原子操作。原子操作+volatile保证了可见性=线程安全
原子性
synchronized不但保证了原子性，还保证了可见性