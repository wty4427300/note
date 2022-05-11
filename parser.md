parser的功能是吧sql语句按照sql语法规则进行解析，将文本转换成抽象语法树（ast）
１．首先build goyacc工具,然后使用goyacc根据parser.y生成解析器parser.go。
２．lex&yacc介绍是用来生成词法分析器和语法分析器的工具，他们的出现简化了编译器的编写。
３．lex根据输入的patterns生成词法分析器,词法分析器读取源代码，根据patterns将源代码转换成
source code->tokens->语法树。
source code源代码->词法分析<-lex<-patterns
生成tokens->语法分析<-yacc-<语法规则
生成语法树，语法树以运算符作为根节点和子节点，数据作为叶子节点
4.

// Lex返回一个token并将token值存储在v中。

//Scanner满足yyLexer接口。

// 0和invalid是这个函数返回的特殊令牌id:

//返回0告诉解析器扫描器满足EOF，

//返回invalid告诉解析器扫描器遇到非法字符。

... definitions ...
%%
... rules ...
%%
... subroutines ...

…定义……

%%

…规则……

%%

…子程序……

产生式冒号左边的项（例如 statement）被称为非终结符， INTEGER 和 VARIABLE 被称为终结符,它们是由 Lex 返回的 token.

预处理

tokenize

generate随机：每条规则都随机选择一个分支

如何终止递归:每个节点都存父节点的信息，loopbakcDetection向上寻找同名规则的个数，超过某个值填充terminator,随后在buildtree时裁剪。相当于引用计数器，当等于某个数时裁剪掉后面的节点。


枚举：遍历所有规则分支

出现次数限制防止死递归　

由于tidb和parser互相依赖所以需要两个go mod我们需要在编译的时候cp go.mod1 go.mod
cp go.sum1 go.sum这样就可以在goland中运行parser了。


直接执行parser_test.go

测试用例，预期能不能过　，把ast转换成sql

添加测试用例

查看mysql文档

然后去找parser.y对应的语法规则，添加规则和常量（ast.xxx）

添加string()里面的case分支

添加优先级

运行make fmt 

运行make test

提交代码

make parser重新生成parser.go



我要做的1.删除测试用例。


