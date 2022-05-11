对于go语言来说编译的最小单位是包

每个包下有很多的go文件

这些个go文件又一个fileSet结构保存
底层的数据结构就是数组

base代表一个文件的开始
base+size代表一个文件的结束

offset代表这个文件读取到哪儿的偏移量

pos是fileSet的全局偏移量

什么是面值啊？就是在代码里面直接表示的值

整数型面值不支持科学计数法

type basicList struct{
    ValuePos token.Pos   // 偏移量
	Kind     token.Token // token类型
	Value    string  //值
}
通过这个结构体可以直接构造面值

基础面值在语法树中是属于叶子节点的存在

type Ident struct {
	NamePos token.Pos // identifier position
	Name    string    // identifier name
	Obj     *Object   // denoted object; or nil
}

go语言的代码结构
分为三层：目录结构，目录内部的包结构，文件内部的代码结构

map[String] *ast.Package 包含多个包的信息

parser.PasrseFile用于解析单个文件返回ast.ParseFile

单个ast.Package由多个ast.File组成

# 关复合类型

## 类型语法

# 数组类型
从语法树中我们能获得该数组的类型，元素类型，元素个数，以及一个元素的长度






