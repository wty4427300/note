数据库系统概念
第三章：
1.基本类型
char 定长
varchar  变长
int 整数
smallint 小整数类型
numeric(p,d):定点数
real,double precision:浮点数与双精度浮点数，精度与机器相关。
表
create table department
(dept_name varchar(20)
 building varchar(15)
 buildget numeric(12,2)
 primary key(dept_name)
);
删除表 drop table 表名
插入insert into 表名(filed1,filed2,...)
           values
           (value1,value2,.....)

修改数据/更新数据
update 表名 set 属性名=new属性值
修改指定的数据
update 表名 set 属性名=new属性值 where 属性=值

删除
delete from 表名 where 属性
不用where全部删除

where....like的使用
select * from 表名 where 属性（字段） like  "%属性值" 百分号代表任意字段，相当于做了一个字符串匹配

union连接两个以上的select语句
select 属性 from 表名
union（重复值只会列出一次，用union all会列出所有值）
select 属性 from 表名
order by 属性（order by排序）
设置字段属性为not null，这样字段为空的时候会报错
auto_increment定义属性为自增，一般用于主键，数值会自动加1.
primary key定义主键。
engine设置存储引擎，charset设置编码。

SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name
select 字段 ，统计 from 表名 group by 字段。按照字段分组，并统计每个人有多少条记录。
with rollup
SELECT name, SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
在实现分组统计的基础上再进行相同的统计。
我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce 语法：
select coalesce(a,b,c)

连接
大致分为三类
inner join内连：获取两个表中字段匹配关系的记录。
left join左连接：获取左表所有记录，即使右表没有对应匹配的记录。
right join右连接：与left join相反。
select a.id,b.id,a.name from stu a inner join employee_tbl b on a.id=b.id;


事物
begin或者start transaction显示开启一个事物
commit提交事物，并使已对数据库进行的所有修改成为永久性的。
rollback回滚事物，并撤销正在进行的所有未提交的修改。
savepoint identifier,savepoint允许在事物中创建一个保存点，一个事物可以有多个保存点
release savepoint identifier删除一个保存点，当没有指定保存点时，执行该语句会抛出一个异常
rollback to identifier把事物回滚到标记点；
set transaction 用来设置事物隔离级别。 innodb有四大隔离级别。

1、用 BEGIN, ROLLBACK, COMMIT来实现

BEGIN 开始一个事务
ROLLBACK 事务回滚
COMMIT 事务确认
2、直接用 SET 来改变 MySQL 的自动提交模式:

SET AUTOCOMMIT=0 禁止自动提交
SET AUTOCOMMIT=1 开启自动提交

show columns from testalter_tbl;

alter修改表名或者字段

alter table 表名 drop 字段名

只剩一个字段无法删除时，在最后添加一个字段
alter table 表名 add 字段名 数据类型

添加到第一列
alter table 表名 add 字段名 数据类型 frist

添加到某字段之后

ALTER TABLE testalter_tbl ADD i INT AFTER c;

修改字段类型以及名称
alter table 表名  modify 字段名 新的字段类型

alter table 表名 change 被修改字段名 新的字段名 新的字段类型

mysql> ALTER TABLE testalter_tbl 
    -> MODIFY j BIGINT NOT NULL DEFAULT 100;
修改字段类型并且默认值为100

修改表名
mysql> ALTER TABLE testalter_tbl RENAME TO alter_tbl;

索引
创建索引
create index 索引名 on 表名（字段(字段长度)）

alter添加索引

alter table 表名 add index 索引名（列名column）


创建表的时候直接指定
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
INDEX [indexName] (username(length))  
 
);  

删除索引 

drop index [索引名] on 表名

唯一索引

创建索引
CREATE UNIQUE INDEX indexName ON mytable(username(length)) 
修改表结构
ALTER table mytable ADD UNIQUE [indexName] (username(length))
创建表的时候直接指定
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
UNIQUE [indexName] (username(length))  
 
);  


ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
ALTER TABLE tbl_name ADD INDEX index_name (column_list): 添加普通索引，索引值可出现多次。
ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):该语句指定了索引为 FULLTEXT ，用于全文索引。


mysql> ALTER TABLE testalter_tbl MODIFY i INT NOT NULL;
mysql> ALTER TABLE testalter_tbl ADD PRIMARY KEY (i);
主键只能作用于一个列上，添加主键索引时，你需要确定主键默认不为空


显示索引信息
SHOW INDEX FROM table_name; \G

临时表的创建与删除
create tempopary table 表名
drop table 表名

表复制
SHOW CREATE TABLE 表名 \G;
得到完整结构，然后执行就像完事了。

 INSERT INTO clone_tbl (runoob_id,
    ->                        runoob_title,
    ->                        runoob_author,
    ->                        submission_date)
    -> SELECT runoob_id,runoob_title,
    ->        runoob_author,submission_date
    -> FROM runoob_tbl;


来给大家区分下mysql复制表的两种方式。

第一、只复制表结构到新表

create table 新表 select * from 旧表 where 1=2

或者

create table 新表 like 旧表 

第二、复制表结构及数据到新表

create table新表 select * from 旧表 



