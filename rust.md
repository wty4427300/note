# 一.rust编程指北。
1.缩进风格使用４个空格
2.！都是宏
3.;结尾
4.rustc main.rs编译
5.预编译静态类型语言
6.cargo --version查看cargo版本或者cargo -V
7.使用cargo创建项目new cargo hellow
8.cargo生成了两个文件和一个目录（cargo.toml和main.rs文件夹是src）
9.cargo.toml有四行属性分别是项目名，版本，作者以及edition值
10.构建和运行cargo项目
cargo build
linux可执行文件: /target/debug/项目名
首次运行cargo build也会生成一个cargo.lock这个文件记录以来的实际版本
11.我们也可以使用cargo run运行可执行文件。如果我们改变了程序代码cargo会先编译在执行。
12.cargo check可快速检查目标代码是否可以编译。（因为他省略了生成可执行文件的过程）



# 开始编码
## 一.编写猜猜看游戏
1.let:绑定
2.match:表达式可以分别处理result每一种结果。
3.方法：
4.关联函数：静态方法
5.外部crate：可以理解为引外部的包。方式 修改cargo.toml的[dependencies] 片段告诉 Cargo 本项目依赖了哪些外部 crate 及其版本
6.mut:（加了就是可变）（没加就是不可变）

## 二.通用编程概念
1.关键字
2.标识符
3.原始标识符：有时候需要把关键字作为名称所以可以使用  r#关键字  来使用

## 三.变量与可变性
变量默认不可变
可以使用mut使得变量可变
常量关键字为const
当多次使用let时，实际上是创建了一个新变量，我们可以改变值的类型，但复用这个名字

##　四.数据类型
标量类型代表一个单独的值。
rust有四种基本的标量类型：整形，浮点，布尔，字符
let x=2.0;//f64
let y:f32=3.0;//f32
数值运算 （加，减，乘，除，取余）

复合类型
复合类型可以将多个值组合成一个类型
rust有两种（元组，数组）

rust中数组是固定长度的一旦声明他们的长度就不能增长或缩小。

## 五.函数
也是很简单滴由关键字fn来声明
加了分号就是语句
而语句不会返回值

具有返回值的函数

我们可以不对返回值命名，但是必须->声明返回值的类型。

## 六.注释
//大家一样

## 七.控制流
其他和别的语言没什么区别

但是可以这样写let 变量=if true{
 值
}else{
 值
};

值的类型要相同

循环有三种loop,while,for

for 变量 in  数组.iter()   (正序)
相当于go里面的for range
for 变量 in  数组.rev()   （倒序）

## 七.所有权
栈堆
所有权就是为了管理堆数据

所有权规则
1.rust中的每一个值都有一个被其称为所有者的变量。
2.值有且只有一个所有者。
3.当所有者（变量）离开作用域，这个值将被丢弃。

变量作用域

string是分配在堆上的。
而string::form()这中语发可以将函数置于string的命名空间namespace下。

string可变
而字符串字面量不可变

内存与分配

## 1.作用域
内存会在拥有它的变量离开作用域后就被自动释放。

## 2.移动
rust的内存拷贝被称为移动。因为他只是拷贝了指针和一些栈上的数据，然后将堆上的数据绑定到栈上。又因为当栈上有第二个引用的了，rust为了不出现二次释放内存的错误，所以将第一个无效化，且以后也不能再使用。

## 3.克隆
如果需要使用深拷贝，可以使用clone函数

## 4.拷贝
编译期就已知大小的数据会被存储在栈上。

所有权与函数
把值传递给函数在语义上与给变量赋值相似，向函数传递值可能会移动或者复制，就像赋值语句一样。在堆上的变量传入函数没什么不同，出函数时会被drop函数释放。所以在外面的作用域不会对它有其他的操作。

返回值与作用域

变量的所有权总是遵循相同的模式：将值赋给另一个变量时移动它。当持有堆中数据值的变量离开作用域时，其值将通过drop被清理掉，除非数据被移动为另一个变量所有。

## 八.引用与借用
&允许你使用值但不获得所有权。

也就是说你给函数传了个引用，离开的时候函数作用域的时候并不会drop释放，因为你只有引用没有所有权。

我们将获取引用作为函数参数称为借用。不允许修改引用的值。

可变引用

在引用的时候加上（mut）& mut 变量。不过可变引用有一个很大的限制：在特定作用域中的特定数据有且只有一个可变引用.

这样做的好处
1.两个或更多指针同时访问统一数据
2.至少有一个指针被用来写入数据
3.没有同步数据访问的机制

不能在同一个作用域内同时拥有可变与不可变引用

悬垂引用

就是引用的指针还在,但是对应的内存被释放了.

引用的规则
1.在任意给定的时间.要么只能有一个可变引用,要么只能有多个不可变引用.
2.引用必须总是有效的.

## 九.slice类型
1.首先slice没有所有权
2.变量.iter()

let a=&s[0..4] 前开后闭区间
let a=&s[0..=4]前开后开区间

## 十.使用结构体住址相关联的数据

定义并实例化结构体
1.结构体比元组灵活的地方在于:不需要依赖顺序来指定或访问实例中的值.

2.rust不允许只将某个字段标记为可变的.必须整个实例都是可变的.

3.我们可以在函数体的最后一个表达式中构造一个结构体的新实例,来隐式返回整个实例.(这么有点类似构造函数噢噢噢噢)

4.参数名与字段名完全相同叫做字段初始化语法.

5.使用结构体更新语法从旧实例创建新实例.

注意:变量绑定结构体的写法,别忘后面加;号.

6.可以定义元组结构体

7.没有任何字段的结构体被叫做 类单元结构体.

## 十一.一个使用结构体的示例程序

使用{:?}打印结构体,或者实例.

## 十二.方法语法

1.说白了就是把实例的函数全部放在impl块里.就叫做方法了.

2.自动引用和解引用

带有更多参数的方法. 一个方法传入两个实例一个self一个other使用&代表借用.希望main函数继续持有其所有权,不影响继续使用.

关联函数
1.impl允许在块中定义不以self作为参数的函数.这被称为关联函数,因为他们与结构体相关联.

2.关联函数经常被用作返回一个结构体新实例的构造函数.

3.每个结构体允许拥有多个impl块,但是一般这么写没啥意义,在trait中会用到这个(意思就是先不聊了,溜了溜了).

## 十三.枚举和匹配模式

1.无值枚举和结构体绑定
  
2.有值枚举直接申请内存

3.可以使用impl来为枚举定义方法.

option枚举和其相对于空值的优势

rust没有空值但是可以使用option<T>这个枚举来编码存在或者不存在.

使用option的none需要声明类型.

4.如何从option<T>中的some中取出T来使用?

## 十四.控制流运算符

1.match的匹配是顺序的,表达式的结果值就是match表达式的返回值

2.如果分支代码较短则不使用大括号.如果想在分支运行多行代码可以使用大括号将代码包裹起来

3.绑定值的模式

4.匹配option<T>

将match和枚举相结合是一种常见的写法.

5.通配符_

## 十五.简单的控制流

if let(match的一个语法糖)

## 十六.包.crate.模块

一个包可以带有零个或者一个库crate和多个二进制crate.

## 十七.模块系统用来控制作用域和私有性

模块.一个组织代码和控制路劲私有性的方式
路径.一个命名item(项)的方式
use.关键字用来将路径引入作用域
pub.关键字使项变为公有.
as.关键字用于将项引入作用域时进行重命名
使用外部包
嵌套路径来消除大量的use语句
使用glob运行符将模块的所有内容引入作用域
如何将不同模块分割到单独的文件中

路径就是调用模块的方式
1.绝对路径:crate开头逐层向下
2.相对路径:从相对路径,由当前模块层开始(以self和super或者当前模块的表示符开始).

## 十八.常见集合

非常快的过了一边,看样子似乎不用留什么笔记了

painc这个东西和go那边意义上没有太大的区别

## 十九.trait和生命周期
泛型在编译期会被填充具体的类型，所以运行时并不会有性能的损失．（泛型代码的单态化）

不允许一个大的生命周期的变量引用一个小生命周期的变量。

如果我们不标明生命周期，就会根据作用域来判断。

生命周期的注解只出现在函数签名中，不出现在函数体中。

生命周期的三条规则

1.多个引用参数，每个都有不同的生命周期

2.一个引用参数，参数和返回值生命周期一致

3.如果参数有一个self则认为这是一个方法，返回值的生命周期和self实例一致

string是贯穿整个程序运行过程的生命周期。

生命周期也是泛型

## 二十.测试



































 
































































































