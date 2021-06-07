# google工程师

## 基础语法

1. 变量
2. 常量
3. 条件
switch会自动break，除非使用fallthrough
switch后可以没有表达式
4. 循环
for条件里不需要括号
5. 函数
可以返回两个值，第二个返回值是error,fmt.Errorf("string")
函数可以当成参数
p := reflect.Vlaueof(f).Pointer()
fName := runtime.FuncForPC(p).Name()
利用reflect，可以获得函数的指针
匿名函数
func sum(values ...int) int {} 可变参数
返回值类型写在最后面
可返回多个值
函数作为参数
没有默认参数，可选参数
6. 指针
go语言的指针不能运算
go语言只有值传递一种方式

## 内建容器

1. 数组
var arr1 [5]int
arr2 := [3]int{1,2,3}
arr3 := [...]int{1,2,3}
var grid [4][5]int
数量写在类型的前面
遍历，用for，或range
_省略变量
数组是值类型，
func f(arr [10]int)会拷贝数组
[10]int和[20]int是不同的类型
func f(arr *[10]int)，这种做法才会拷贝指针
go语言的数组的指针方法一般不会用
使用切片
2. slice
arr:=[...]int{0,1,2,3,4}
s := arr[2:4]
arr[:]，全部
arr[2:]
arr[:6]
func f(arr []int) 变成slice，就可以改变数组的值了
slice本身没有数据，是对底层array的一个view
reslice，对slice的结果也可以继续取slice，这些slice都是对应同一个array
slice的扩展
arr:=[...]int{0,1,2,3,4,5,6}
s1 = arr[2:6] // 2,3,4,5
s2 = s1[3:5] // 5,6
slice的结构
- ptr:指向开头的元素
- len长度
- cap整个长度
slice可以向后扩展，不可以向前扩展
s[i]不可以超越len(s)，向后扩展不可以超越底层数组cap(s)
len(s1),cap(s1) // 4,6
len(s2),cap(s2) // 2,3
向slice添加元素
添加元素时如果超越cap，系统会重新分配更大的底层数组
由于值传递的关系，必须接收append的返回值
s = append(s, val)
var s []int // len=0,cap=0
s1 := []int{2,4,6,8} // 4,4
s2 := make([]int,16) // 16,16
s2 := make([]int,10,32) // 10,32
删除元素
s2 = append(s2[:3], s2[4:]...) // 把第三个元素删掉
删除头尾
s2 = s2[1:]
s2 = s2[:len(s2)-1]

3. map
``` go
m:=map[string][string]{
    "s":"sun",
}
m2 := make[map[string]int] // empty
var m3 map[string]int // nil
for k,v := range m {

} // 无序的，如需顺序，需手动key排序，
res, ok := m["notexist"]
// res是空串，key不存在，获得value类型的初始值
delete(m, "s")
len(m)
// map的key使用哈希表，必须可以比较相等
// 除了slice,map,function的内建类型都可以作为key
// struct类型不包含上述字段，也可以作为key

// 示例：3. 寻找最长不含有重复字符的子串
func lengthOfNonRepeatingSubStr(s string) int {
    // 这种方式是int32,ch是rune
    // for i, ch:=range s {}
    lastOccurred := make(map[byte]int)
    // lastOccurred := make(map[rune]int)
    start := 0
    maxLength := 0
    // for i, ch := range []byte(s) {
    for i, ch := range []rune(s) { // 解决中文问题
        if lastI, ok := lastOccurred[ch]; ok && lastI >= start {
            start = lastI + 1
        }
        if i - start + 1 > maxLength {
            maxLength = i - start + 1
        }
        lastOccurred[ch] = i
    }
    return maxLength
}
// 但是中文就不对了
// rune相当于golang的char
s := "Yes我爱慕课网！"
// len(s) 19
// 每个中文都是3字节
for _, b := range []byte(s){
    fmt.Printf("%X", b)
}
for _, b := range s{ // ch is rune
    fmt.Printf("%X", b)
}
for i, ch := []rune(s) { // 这样可以一个一个字节的输出，每个字符占4个字节，重新开了一个rune数组去存
    fmt.Printf("%c", ch)
}
// 使用utf8.RuneCountInString获得字符

Fields,Split,Join
Contains,Index
ToLower,ToUpper
Trim,TrimRight,TrimLeft

```


## 面向对象

go语言仅支持封装，不支持继承和多态
（继承和多态的事情，面向接口来做）
go语言没有class,只有struct

``` go
type treeNode struct {
    value int
    left, right *treeNode
}

func main(){
    var root treeNode
    root = treeNode{value:3}
    root.left = &treeNode{}
    root.right = &treeNode{5,nil,nil}
    root.right.left = new(treeNode) // 不管是指针还是实例，go语言都可以用点语法
    nodes := []treeNode {
        {value:3}
        {},
        {6, nil, &root},
    }
    fmt.Println(nodes)
}

// 工厂函数
func createNode(value int) *treeNode {
    return &treeNode{value:value} // 返回了局部变量的地址
}

root.left.right = create(2)

// 结构创建在堆上还是栈上？不需要知道
// 编译器如果看到返回出去，就分配到堆上，如果不返回，就放在栈上

func (node treeNode) print() { // 值传递不能改变里面内容
    fmt.Print(node.value)
}
func (node *treeNode) setValue(value int) {
    if node == nil {
        fmt.Println("error")
        return
    }
    node.value = value
}
root.print()
root.setValue(100)

// 显示定义和命名方法接收者
// 使用指针作为方法接收者，只有使用指针才可以改变结构内容
// nil指针也可以调用方法！
var pRoot *treeNode
pRoot.setValue(200)
pRoot = &root
pRoot.setValue(300)
pRoot.print()

// 树的遍历
// 略

// 值接受者vs指针接受者
// 要改变内容必须使用指针接受者
// 结构过大也考虑使用指针接收者
// 一致性：如有指针接收者，最好都是指针接收者
// 值接收者是go语言特有
// 值/指针接收者均可接收值/指针


```

封装
- 驼峰
- 首字母大写public，小写private

包 
- 每个目录一个包
- main包包含可执行入口
- 为结构定义的方法必须放在同一个包内
- 可以是不同文件

如何扩充系统类型或者别人的类型
- 定义别名：最简单，
- 使用组合：最常用
- 使用内嵌：需要省下许多代码

``` go
// 组合
type myTreeNode struct {
    node *tree.Node
}
// 定义别名
type Queue []int

// 使用内嵌来扩展类型
type myTreeNode struct {
    *tree.Node
}
myTreeNode可以点Right,Left,Node,方法，平铺出来

```
## 依赖管理

依赖管理的三个阶段
gopath,govendor,go mod

gopath
所有的依赖都去gopath里面找
结果gopath越来越大

vendor解决依赖的多版本问题
包寻找顺序 vender,goroot,gopath

govender
大量的第三方依赖管理工具

go1.11版本后，加入了go module
建项目，生成了go.mod文件
go get -u 后，go.mod 和go.sum里面多了东西

代码存放在gopath/go/pkg/uber/1.12下面

go mod tidy

go mod init moduletest2
go build ./... 构建全部的包

go mod
- 由go命令统一管理，用户不必关心目录结构
- 初始化：go mod init
- 增加依赖：go get
- 更新依赖 go get [@v] , go mod tidy
- 讲旧项目迁移到go mod.go mod init, go build ./...

go包管理难的问题在go mod后得到了解决

一个package下面只能有一个main函数
在目录下建立多个文件，每个main函数文件放单个文件里
官方就是这么做的，每个工具有自己独立的目录
package main
一般大的项目没有这么多的main入口
go build ./... 当前文件和目录下所有子文件都去构建
go install ./... 才能看到结果

## 接口

接口的概念
- 强类型语言：熟悉接口的概念
- 弱类型语言：没（少）有接口的概念

python是强类型还是弱类型？

两个包都有Retriver结构，引入时只能按infra.Retriever，不能按test.Retriever

定义一个retriever接口

infra.Retriever只要实现了retriver里面的方法

duck typing
- 描述事物的外部行为而非内部结构
- 严格说go属于结构化类型系统，类似duck typing

不同语言的鸭子实现

go语言的鸭子类型

接口的定义
- 接口由使用者定义

接口的实现
- 接口的实现是隐式的
- 只要实现接口里的方法

接口变量里面有什么
接口变量
- 实现者的类型
- 实现者的指针——指向实现者

接口变量自带指针
接口变量同样采用值传递，几乎不需要使用接口的指针
指针接收者实现只能以指针方式使用，值接收者都可

查看接口变量
- 表示任何类型：interface{}
- v.(int)

指针接收者，值接收者

go语言重要接口：Reader,Writer

## 函数式编程

函数与闭包

函数式一等公民：参数，变量，返回值都可以是函数
高阶函数
闭包

正统函数式编程
- 不可变性：不能有状态，只有常量和函数
- 函数只能有一个参数

go语言闭包的应用

斐波那契数列
为函数实现接口
使用函数来遍历二叉树

go语言闭包
- 更为自然，不需要修饰如何访问自由变量
- 没有lamda表达式，但有匿名函数

## 异常处理

defer file.Close()
defer writer.Flush()

先进后出

defer
- 确保调用在函数结束时发生
- 参数在defer语句时计算
- defer列表为后进先出

何时使用defer
- open/close
- lock/unlock
- printHeader/PrintFooter

统一错误处理逻辑

panic
- 停止当前函数执行
- 一直向上返回，执行每一层的defer
- 如果没有遇见recover，程序退出

recover
- 仅在defer调用中使用
- 获取panic的值
- 如果无法处理，可重新panic

出错处理在实际项目中的应用

error vs panic
- 意料之中的：使用error。如：文件打不开
- 意料之外的：使用panic。如：数组越界

错误处理综合示例
- defer+panic+recover
- type asseartion
- 函数式编程

## 测试

传统测试vs表格驱动测试

表格驱动测试
- 分离的测试数据和测试逻辑
- 明确的出错信息
- 可以部分失败
- go的语法更容易实现表格驱动测试


Benchmark
看程序的运行速度

go tool pproof

go test -bench cpuprofile

性能调优
- cpuprofile获取性能数据
- go tool pprof 查看性能数据
- 分析慢在哪里
- 优化代码
- 重复上面的步骤

## goroutine

对并发编程天然支持

协程coroutine
- 轻量级线程
- 非抢占式多任务处理，由协程主动交出控制权
- 编译器/解释器/虚拟机层面的多任务
- 多个协程可能在一个或多个线程上运行

（线程会争取cpu控制权）

runtime.Gosched()
交出控制权

子程序是协程的一个特例

其他语言中的协程
- C++：Boost.Coroutine
- Java：不支持

python中的协程
- 使用yield关键字实现协程
- 3.5加入了async def对协程原生支持

goroutine
- 调度器。协程放在一个或多个线程里
- 任何函数只需加上go就能送给调度器运行
- 不需要在定义时区分是否是异步函数
- 调度器在合适的点进行切换
- 使用-race来检测数据访问冲突

goroutine可能的切换点
- IO,select
- channel
- 等待锁
- 函数调用（有时）
- runtime.Gosched()
- 只是参考，不能保证切换，不能保证在其他地方不切换

使用top命令查看情况
1000个goroutine，映射到了电脑上的4个核进行

## channel

c := make(chan int)

channel是goroutine和goroutine之间的交互

Printf是io操作，channel之间进行调度

channel箭头

bufferChannel,缓冲区

channelClose
- c := make(chan int 3)

channel
bufferd channel
range
理论基础：CSP模型

不要通过共享内存来通信，通过通信来共享内存

使用channel来等待goroutine结束
waitGroup实现等待

使用channel来实现树的遍历

select+default 非阻塞式获取

使用select来进行调度
- select的使用
- 定时器的使用
- 在select中使用nil channel

传统同步机制
- waitGroup
- mutex
- cond

## 广度优先搜索

用循环创建二维slice
使用slice来实现队列
用point来实现抽象

## http库

net/http/pprof 分析性能

localhost:8888/debug/pprof

go tool pprof 
web可视化

其他标准库
bufio
log
encoding/json
regexp
time
strings/math/rand

godoc -http :8888
看标准库的文档

studygolang.com/pkgdoc

## gin

zap 

middleware的使用，类似java里的aop

利用middleware记录path,log latency, response code

context 接口，

*gin Context

可以在中间件里，给context设置requestId

## 分布式网络爬虫

爬虫项目，有一定复杂性，可以灵活调整项目的复杂性，平衡语言/爬虫之间的比重

网络爬虫分类
- 通用爬虫，如baidu,google
- 聚焦爬虫，从互联网获取结构化数据

爬虫的法律风险

robots协议

https://www.baidu.com/robots.txt

技术上：没有约束力
法律上：作为参考

法律风险
- 结果导向
- 使用常识进行判断
- 若不确定，请求相关人士，或直接联系对方
- QPS

es
server:9200/index/type/id
index-database
type-table

## 分布式

对外：REST
模块内部：RPC
模块之间：中间件，REST

分布式：指导节点之间如何通信
微服务：鼓励按业务划分模块
微服务架构通过分布式架构来实现

多层架构VS微服务架构
- 微服务架构具有更多的服务
- 微服务通常需要配合自动化测试，部署，服务发现等
- 目前我们倾向于微服务架构

目前的问题
- 限流问题
- 去重问题
- 数据存储问题

限流问题
- 单节点能够承受的流量有限
- 将worker放到不同的节点

去重问题
- 单节点能承受的去重数据量有限
- 无法保存之前去重结果
- 基于key-value（如redis）进行分布式去重

存储问题
- 存储部分的结构，技术栈和爬虫部分区别很大
- 进一步优化需要特殊的es技术背景
- 固有分布式

解决限流问题
解决存储问题
分布式去重：回家作业

从channel到分布式

rpc
- jsonrpc
- grpc(使用protobuf)


