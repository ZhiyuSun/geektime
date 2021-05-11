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



## 面向对象

