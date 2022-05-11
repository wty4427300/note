# 装饰器模式
允许向一个新对象添加新功能,同时又不改变其结构.

创建一个装饰类,用来包装原有的类,并在保持类方法签名完整性的前提下,提供了额外的功能

主要解决:一般的,我们为了扩展一个类经常使用继承方式实现.由于继承为引入静态特征,并且随着扩展功能的增多,子类会很膨胀

何时使用:在不想增加很多子类的情况下扩展类

如何解决:将具体功能职责划分,同时继承装饰者模式

关键代码:
1.component类充当抽象角色,不应该具体实现
2.修饰类引用和继承component类,具体扩展功能类重写父类方法

# springboot

多环境配置在主配置文件里面配置
spring.profiles.active=test
等号后面写那个名就调用那个配置文件

@RestController是注解返回的都是对象而不是视图

@SpringBootApplication是一个方便注解

	{@Configuration：将类标记为应用程序上下文的Bean定义的源。

	@EnableAutoConfiguration：告诉Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。例如，如果spring-webmvc在类路径上，则此注释将应用程序标记为Web应用程序并激活关键行为，例如设置DispatcherServlet。

	@ComponentScan：告诉Spring在包中寻找其他组件，配置和服务hello，让它找到控制器。}

大概的组织方式就是dao层数据到service层,controller层数据到service层,两个数据作比较得出结果.controller层向前端发送结果或者数据.

# mybatis二级缓存
1.session级别
2.mapper级别xml的对象用hash存

mysql时间自动更新

1.timestamp default current_timestamp on update current_timestamp
在创建记录和修改现有记录的时候都对这个数据列刷新

2.timestamp default current_timestamp
字段设置为当前时间,但以后修改时,不再刷新它

3.timestamp on update current_timestamp
在创建新记录的时候把这个字段设置为0

# MyBatis工作原理
1.读取mybatis-config.xml配置文件.主要内容是获取数据库连接
2.加载映射文件Mapper.xml.
3.创建会话工厂sqlsessionfactory
4.创建sqlsession对象.由会话工厂创建sqlsession对象,该对象包含了执行sql的所有方法
5.mybatis底层定义了一个executor接口来操作数据库,它会根据sqlsession传递的参数动态生成需要执行的sql语句.同时负责查询缓存的维护
6.在executor接口的执行方法中,包含一个mappedstatement类型的参数.该参数 对应映射的信息的封装,用于存储要映射的sql语句的id,参数等也就是说mapper文件中的一句sql要对应一个mappedstatement的对象.sql的id就是mappedstatement的id.
7.输入参数映射
8.输出结果映射

使用mysql中的concat()函数拼接sql防止sql注入
(select * from table where username like concat('%',#{value},'%')

# jwt的三重结构
header头部(标题包含了令牌的元数据(从哪来以及授权信息之类的东西),并包含签名或加密算法的类型)
payload(用户id以及一些需要存储的东西)
signature(签名/签证)

jwt的header分为两部分
alg:hs256
typ:jwt


# maven打包需要编写plugins(在pom.xml里面)

# 标准的基于角色权限的五张表
权限
角色
角色-权限
用户-角色

角色和用户的关系通过数据库配置控制.角色可以访问的权限通过硬编码控制.

抽象类是模板设计
而接口是契约式设计

# java计算时间比较方便的方式.
Date date=new Date();
        Calendar calendar=Calendar.getInstance();
        calendar.setTime(date);
        calendar.set(Calendar.DAY_OF_MONTH,calendar.get(Calendar.DAY_OF_MONTH)+1);//让日期加1
        date=calendar.getTime();



 
# springboot
配置有两种方式一种
是通过application.yml 配置文件配置
另一种就是用@springbootconfiguration注解


# java8

## lambda表达式
lambda表达式主要用来定义执行的方法类型接口,例如一个简单方法接口.
比如说我有一个借口,在以前的话我需要继承接口实现这个接口的实际执行方法.
现在我只需要在栈上初始化这个接口并在等号后面写上lambda表达式即可因为
lambda表达式本身就相当于闭包,是函数的实际实现
这样就不需要我们再去写匿名方法.

表达式写法
1.不写参数类型
编译器可统一识别
(a,b)->a+b
2.写参数类型
(int a,int b)->a+b
3.接受两个参数需要加(),单参数不需要
4.如果计算过程只有一个语句就不需要{}
5.如果使用了{}就必须写return


lambda表达式不可修改局部变量
而且lambda表达是的参数是不允许和局部变量同名的.

## 方法引用
类::方法
对象::方法


## 函数式接口
@FunctionalInterface
interface GreetingService
{
    void sayMessage(String message);
}
public class funcIMP {
    public static void main(String[] args) {
        GreetingService greetingService=message -> System.out.println(message);
        greetingService.sayMessage("shabi");
    }
}


## 默认方法
就是说接口可以有实现方法,而且不需要实现类去实现其方法
但是有时候一个类实现了多个接口,这些接口具有相同的默认方法
解决方案有两种
1.创建自己的默认方法来覆盖重写
2.用super来调用指定接口的默认方法

# Stream
对集合的聚合处理
        List<String> strings= Arrays.asList("aaefwefwefewfwefwefwef","","aaaaa","dfdsfs","sdsdsdsdsd");
        List<String> filtered=strings.stream().filter(string->!string.isEmpty()).collect(Collectors.toList());
比如说这两行代码
第二行就是一个标准的stream
filter就是一个stream函数,他的参数是一个lambda,他过滤出不为空的元素.collect将过滤出的元素归类到一个新的list里面.
## forEach
Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
## map
map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());

## limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

Random random = new Random();
random.ints().limit(10).forEach(System.out::println);

## sorted
sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);

## parallel
parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
int count = strings.parallelStream().filter(string -> string.isEmpty()).count();

## Collectors
Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);

## 统计
另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());












