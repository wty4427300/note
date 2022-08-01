# sql的语言共分为四大类:

1.数据查询语句DQL
2.数据操纵语句DML
3.数据定义语句DDL
4.数据控制语句DCL

# DQL

select
from
where

# DML主要有三种形式

1.插入:insert
2.更新:update
3.删除:delete

# DDL

CREATE TABLE/VIEW/INDEX/SYN/CLUSTER

# DCL

数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制
数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
1.GRANT：授权。
2.ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。
回滚---ROLLBACK
回滚命令使数据库状态回到上次最后提交的状态。其格式为：
SQL>ROLLBACK;
2.COMMIT [WORK]：提交。





