1词法分析
词法分析的作用就是对一个文本或称为一串字符串（包括空格，换行，特殊符号等）进行内容提取。通过分析给每一个字段打上标记（TOKEN）。那么如何分析呢，以sql的查询语句为例

select * from userinfo where name = 'wty' order by age

这个字符串有多少信息呢，首先，select，from， where， order， by 这些单词在从左到右扫描时与userinfo，age这些并无不同，但很明显这些单词是数据库的保留单词，也称为关键字。所以一般的我们需要一个关键字表，在扫描到单词时判断其是可变的字段还是关键字。其次对于 * ,=,’wty’，我们单独将其标记为特殊符号，操作符，字符串，就这样，我们对所有的可能情况进行识别给不同类型的字段以唯一的标识(TOKEN)一般为int。所以此法分析的作用就是将一系列的文本转换成TOKEN序列。

2文法分析
对于词法分析后的TOKEN序列，我们需要知道他是否符合我们的文法规范，例如

1 select from a=1

像这样的句子不符合sql规范，我们需要明确的指出。文法分析其实就是一个有限状态自动机，当扫描到一个TOKEN时，我们根据不同的情况跳转到另一状态，直到终结状态。

2.3语义分析
语义阶段与文法阶段往往可以同步进行，在sql文法分析的阶段我们便可以标明该语义树的各个部分，比如在扫描到select时，我们便可以确定这是一个select语句或者可能是错误语句，然后根据后续信息补全select语法树。

工具 goyacc
go git github.com/golang/tools/cmd/goyacc
go build
go intall



1.lex与yacc工作流程
go提供goyacc用于语法分析。在我们写好一个yacc规则后，使用goyacc sql.y指令生成对用的go文件。其中包含两个重要的对象

type yyLexer interface {
    Lex(lval *yySymType) int
    Error(s string)
} type yyParser interface {
    Parse(yyLexer) int
    Lookahead() int
}


yyparser是yacc自动实现的，不需要我们操作，parser是入口函数，它会不停的调用Lex函数来获取TOKEN进行文法分析。我们需要自己实现一个Lex即实现yyLexer接口。

Lex实现

type Tokenizer struct {
    query Query
    scanner *Scanner
}

func (tkn *Tokenizer) Lex(lval *yySymType) int{
    var typ int
    var val string

    for {
        typ, _, val  = tkn.scanner.Scan()
        if typ == EOF{
            return 0
        }
        if typ !=WS{
            break
        }
    }
    lval.str = val
    return typ
}
func (tkn *Tokenizer) Error(err string){
    log.Fatal(err)
}

2.yacc写法



