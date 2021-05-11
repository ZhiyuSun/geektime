python里面只有字符串，没有字符，屏蔽了很多细节

python把所有数字类型都归为int类型
对于python来说，int占用字节是动态，python的int我们不用担心超过上限

Python3.8新特性：
- 海象运算符
- 类型注解。动态语言不需要声明变量类型，这种做法在很多人眼里是不好的的，不好维护。
类型的申明，不会实际影响使用。hints提示作用。
- 函数参数和返回值类型声明

为什么Python没有switch语句
查看Python官方:PEP 3103-A Switch/Case Statement
发现其实实现Switch Case需要被判断的变量是可哈希的和可比较的，这与Python倡导的灵活性有冲突。在实现上，优化不好做，可能到最后最差的情况汇编出来跟If Else组是一样的。所以Python没有支持。

python和go的路线不一样，go走的是高性能

python的list和go的切片

Python的try里面的return之前会把结果暂存到一个变量里，再去执行finally，如果finally里改了这个变量，值传递没有影响，引用传递有影响，finnaly里可以加return，里面return了，就不看try的return了

python哪怕没有某个方法，也可以通过get_attr之类的魔法方法找到