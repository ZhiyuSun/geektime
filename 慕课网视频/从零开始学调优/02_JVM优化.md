# JVM优化

## JVM优化的必要性

本地开发环境和生产环境大相径庭
- 运行的应用卡住了，日志不输出，程序没有反应
- 服务器的CPU负载突然升高
- 在多线程应用下，如何分配线程的数量

## JVM的运行参数

三种参数类型

### 标准参数
-help
-version
一般都很稳定，
java -help，看到所有的参数

-D 传入系统属性  java -Dname  System.getProperty("name")
意义：通过系统参数可以传入项目环境标记，根据不同环境选择不同配置

通过-server或-client设置jvm的运行参数
server vm的初始堆空间会大一些，默认使用并行垃圾回收器，启动慢运行快
client vm的初始堆空间好小一些，默认使用串行垃圾回收器，启动快运行慢
64位的操作系统只支持server模式

### 非标准参数

java -X
默认混合模式

### -XX参数（使用率较高）

主要用于jvm的调优和debug操作

boolean类型
-XX:+DisableExplicitGC表示禁用手动调用gc操作，也就是说调用System.gc()无效

非boolean类型
-XX:NewRatio=1 表示新生代和老年代的比值

-Xms与-Xmx参数
设置jvm的堆内存的初始大小和最大大小
-Xmx2048m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为2048m
-Xms512m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为512m

查看jvm的运行参数
- 运行java命令时打印参数，需要添加-XX:+PrintFlagsFinal参数即可
- 查看正在运行的jvm参数：需要借助于jinfo命令查看。
    - jinfo -flags pid
    - 不想查所有的 jinfo -flag concGCThreads pid

## JVM内存模型

jdk1.7的堆内存模型

jdk8毫无疑问是最快的Java版本
双8版本，jdk8,tomcat8.5

jvm heap
- Young Gen
    - eden 朝生夕死，很快回收——Minor GC
    - s0 和s1互相颠倒
    - s1
- Old Memory——Major GC
Perm
- class信息，字节码信息。-XX:PermSize,-XX:MaxPermSize

jdk1.8的堆内存模型

jvm heap
- Young Gen（1/3堆空间）
    - eden (8/10) 朝生夕死，很快回收——Minor GC
    - From (1/10) 和s1互相颠倒，survivor space
    - To  (1/10) survivor space
- Old Memory（2/3堆空间）
    - Tenured Space
MetaData(非堆，不在jvm内存里面了，而是在直接内存里面了)
- MetaData Space(直接内存)

为什么要废弃1.7中的永久区？
- 官方：为了融合HotSpot JVM和jRckit VM
- 没必要维护永久区。永久区发给字节码，每次有新文件基本都要重新启动服务器，垃圾回收器不需要管理永久区

### 通过jstat命令进行查看堆内存使用情况

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：
jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

jstat -class pid  查看类
jstat -compiler pid  查看编译信息
jstat -gc pid 查看垃圾统计信息
- S1C 本来多少
- S1U 使用了多少
- EC
- EU
- OC
- OU
- MC 方法区
- MU
- CCSC 压缩空间
- CCSU 已经使用的
- YGC 年轻代垃圾回收的次数
- YGCT 年轻代垃圾回收的时间
- FGC 老年代垃圾回收的次数
- FGCT
- GCT 总的垃圾回收的时间

单位一般是k


jstat -gc pid 1000 5
1000毫秒打印一次，打印5次

### jmap的使用以及内存溢出分析

查看内存使用情况
jmap -heap pid
Heap Configuration
Heap usage

查看内存中对象数量及大小
查看所有对象，包括活跃以及非活跃的
jmap -histo pid | more
查看活跃对象
jmap -histo:live pid | more
- B byte
- C char
- i int
- Z 布尔型

将内存使用情况dump到文件中
有些时候我们需要将jvm当前内存中的情况dump到文件中，然后对它进行分析，jmap也支持dump到文件中
jmap -dump:format=b, file=dumpFileName pid
jmap -dump:format=b, file=/tmp/dump.dat 12345
系统出问题时，就这么做的
-b是二进制的

通过jhat对dump文件进行分析
上节dump出来的文件时二进制文件，不方便查看，这时我们可以借助于jhat工具进行查看
jhat -port 8888 /tmp/dump.dat
内部是启动了一个服务器的
localhost:8888 浏览器中访问
执行类似sql的语句去查询

通过MAT工具对dump文件进行分析
eclipse提供的工具
打开dump文件

### 内存溢出的定位与分析

正常的，就加大内存。异常的，就修改代码
-XX:+HeapDumpOnOutOfMemory  oom时，把内存dump出来
MAT工具分析

## jstack的使用

线程的状态。这个状态图有问题

构造死锁
两个线程thread1,thread2,thread1拥有obj1锁，thread2拥有obj2锁。此时thread1想要获取obj2的锁，thread2也想要获取obj1锁，于是出现死锁

我的理解，对象就是锁，对象的头部几个字节存放是否被锁

拿到obj1后，不释放，所以obj2要放到obj1的里面来

死锁后，jstack pids
Thread-0和Thread-1都处于block状态
found one java-level deadlock

## jvisualvm的使用

使用jvisulvm能够监控线程，内存情况，查看方法的cpu时间和内存中的对象，已被GC的对象，反向查看分配的堆栈（如100个String对象分别由哪几个对象分配出来的）。jvisulvm使用简单，几乎0配置，功能比较丰富，几乎囊括了其他jdk自带命令的所有功能

jdk8/bin/jvisulvm.exe

线程dump

### JMX

JMX(java management extensions,即Java管理扩展)，是一个为应用程序、设备、系统等植入管理功能的框架。
JMX可以跨越一系列异构操作系统凭条、系统体系结构和网络传输协议，灵活的开发无缝集成的系统、网络和服务管理应用。

### 监控远程tomcat

想要监控远程的tomcat，就需要再远程的tomcat进行对JMX配置，配置完毕重启tomcat
















