# 事物的四大特性

1.原子性：是一个不可分割的整体，要么全都执行，要么全不执行。执行出错事务回滚。
2.隔离性：同一时间，只允许一个事务请求同一组数据。不同事物彼此之间没有干扰。
3.一致性：事务开始前和开始后。数据库的完整性约束没有被破坏。
4.事务完成后落盘，不能回滚。

# 事务的并发问题

脏读：事务A读取了事务B的数据，事务B回滚，A读到了脏数据。
幻读:事务A修改表A，事务B向表A插入了一条数据，事务A修改完发现表A有一条记录还没有被修改。
不可重读：事务Ａ不断的读表Ａ，事务Ｂ不断的修改表Ａ。导致事务Ａ读取到的数据不一致。

# 四大隔离级别

１．读未提交。就是读到脏数据
２．读已提交。就是要等另一个事务提交完了，才能够读取。解决了脏读问题
３．可重读。保证了每次的读到的数据一致，不管其他事务是否已经提交。解决了不可重读问题
４．串行化。开启一个序列化事务，其他事务的对数据表的写操作都会挂起。

# 全局锁

1.备份的时候需要全局锁,为了备份的逻辑视图完整性.
但是由mvcc实现可重复读是可以保证获取一致性视图的,所以可以不用全局锁.

# 表级锁

分两种:表锁,元数据锁(mdl).
表锁lock table需要显示的执行

mdl在访问一个表的时候会自动添加,以确保读写的正确性.
当对一个表做增删改查操作的时候，加 MDL 读锁
当要对表做结构变更操作的时候，加 MDL 写锁。

读锁不互斥,所以可以有多个线程对表进行增删改查
读写,写写 互斥,所以改变表结构同时间只能有一个线程执行

# 联合索引

实际生效是联合索引+主键索引
比如(a,b)+(c,b)=(a,b,c)

# 行锁

行锁是由引擎层各自实现的.

# rr可重复读隔离级别下

读是事务开始时的read-view(视图)读.即看不到为提交的事务更新.
写是,读后写,即已经提交的事务的更新是可以看到的,保证更新不丢失.

## 视图

mysql中有两个视图的概念:

1. view,执行查询语句生成结果,创建视图的语法是create view.
2. innodb在实现mvcc的时用到的一致性视图,consistent read view
   用于支持 RC（Read Committed，读提交）和 RR（Repeatable Read，可重复读）隔离级别的实现。

视图没有物理结构,作用是事务执行期间用来定义"我能看到什么数据"

## “快照”在 MVCC 里是怎么工作的?

InnoDB 里面每个事务有一个唯一的事务 ID，叫作 transaction id。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的。
数据表中的一行记录，其实可能有多个版本 (row)，每个版本有自己的 row trx_id。

* 按照可重复读的定义，一个事务启动的时候，能够看到所有已经提交的事务结果。但是之后，这个事务执行期间，其他事务的更新对它不可见。
  ，如果是这个事务自己更新的数据，它自己还是要认的。

在实现上， InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”的所有事务 ID。“活跃”指的就是，启动了但还没提交。
数组里面事务 ID 的最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。
这个视图数组和高水位，就组成了当前事务的一致性视图（read-view）。

这个视图数组把所有的 row trx_id 分成了几种不同的情况。

![img_5.png](data%2Fimg_5.png)

这样，对于当前事务的启动瞬间来说，一个数据版本的 row trx_id，有以下几种可能：

1. 如果落在绿色部分，表示这个版本是已提交的事务或者是当前事务自己生成的，这个数据是可见的；
2. 如果落在红色部分，表示这个版本是由将来启动的事务生成的，是肯定不可见的；
3. 如果落在黄色部分，那就包括两种情况a. 若 row trx_id 在数组中，表示这个版本是由还没提交的事务生成的，不可见；b. 若 row
   trx_id 不在数组中，表示这个版本是已经提交了的事务生成的，可见。

* 更新操作为什么是读后写(当前读)和,这是因为必须在最新版本的数据上更新,否则更新就丢失了.
  对应的,select如果加锁也是当前读(加上 lock in share mode 或 for update).

可重复读的核心就是一致性读（consistent read）；而事务更新数据的时候，只能用当前读。如果当前的记录的行锁被其他事务占用的话，就需要进入锁等待。

而读提交的逻辑和可重复读的逻辑类似，它们最主要的区别是：在可重复读隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；在读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图。

* 对于可重复读，查询只承认在事务启动前就已经提交完成的数据；
* 对于读提交，查询只承认在语句启动前就已经提交完成的数据；

# 索引的遍历过程

层级遍历找叶子节点,叶内数据二分查找.

# innodb按页从磁盘获取数据,默认一页16k

# change buffer

数据页在内存里的时候就直接更新,如果数据页不在内存里,就缓存在change buffer里面.在下次
查询需要访问这个数据页的时候,将数据页读取到内存,然后执行change buffer中和这个页相关的
操作.除了查询,后台线程也会定期merge change buffer.

唯一索引需要执行唯一性判断,所以需要一定需要读取数据页,直接修改即可,不需要change buffer.
chang buffer使用的是buffer pool的内存,不能无限增加,通过配置可以修改其占据的百分比.

所以普通索引的更新比唯一索引快,因为可以直接使用change buffer.

# 关于记录复用和page复用

记录复用只要符合二分法得到的查询范围,就可以复用
例如 200 400 600 三个记录删除了400,在插入一条500,那么原来400的记录空间就是可以复用的

# 删除导致的空洞怎么处理

innodb是标记删除,所可以通过重建标,来排除空洞,建立一个新表,把数据copy过去,删除旧标.

# online ddl

建立一个临时文件，扫描表 A 主键的所有数据页；
用数据页中表 A 的记录生成 B+ 树，存储到临时文件中；
生成临时文件的过程中，将所有对 A 的操作记录在一个日志文件（row log）中，对应的是图中 state2 的状态；
临时文件生成后，将日志文件中的操作应用到临时文件，得到一个逻辑数据上与表 A 相同的数据文件，对应的就是图中 state3 的状态；
用临时文件替换表 A 的数据文件。

# 两阶段提交

先写数据,在写redo log,最后写binlog.这是完整的过程.

# redo log到底存储什么

1. 事务ID： 每个Redo Log记录都关联着一个事务，记录了该事务执行的一系列修改操作。
2. 操作类型： Redo Log标记了每次修改操作的类型，如插入、更新、删除等。
3. 受影响数据页： 对于每个修改操作，Redo Log记录了被修改的数据页的标识。
4. 修改前的数据： Redo Log记录了修改操作前的数据，这样在恢复时可以了解修改前的状态。
5. 修改后的数据： Redo Log记录了修改操作后的数据，以及如何修改的信息。
6. 时间戳： 记录了Redo Log记录的生成时间戳，用于恢复时的排序和应用。

# 回答：一个事务的 binlog 是有完整格式的：

statement 格式的 binlog，最后会有 COMMIT；
row 格式的 binlog，最后会有一个 XID event。

# 为什么需要两阶段提交

redo log是作用于innodb的一旦提交就无法回滚,如果这时候binlog写入失败的话,
redo log又无法回滚,数据就和binlog不一致了.两阶段就是为了给所有人一个犯错的
机会,当所有人都准备好再一起提交.

# 数据是从那落盘的

buffer pool,redo log只存page级别的数据变更.innodb如果判断一个数据页,
在崩溃重启的时候丢失了更新,就会先去磁盘里加载页,然后从redo log里更新页

先写redo log buffer,在commit的时候再去写具体的buffer.

# 一条查询sql执行很慢的原因

1.cpu占用满了,新的sql就无法执行了.
2.mdl锁.
3.等待flush table
4.等待行锁
可以在 sys.innodb_lock_waits表里查询

# 幻读

for update的锁,会在两阶段提交的commit阶段释放.
在可重复读的隔离级别下,普通查询是快照读,加锁的是当前读(即值如果发生了改变会读取到最新的结果).
修改读到修改的值不算幻读,幻读仅仅指的是,新插入的行.

幻读导致的问题
本来某一句update只能修改一行,但是在这个update的事务提交之前,又有新的事务更新或者插入了数据.
那么在按照时序执行binlog的时候就会到导致,update可更新的行多了一些,破坏了数据的一致性.
给所有的扫描行加锁可以解决更新的问题,但是无法解决插入的问题.(即行锁无法解决幻读的问题).
所以需要加间隙锁.这样插入新的记录就会导致间隙增加,就可以防止插入新的数据导致的幻读问题.
间隙锁不会冲突所以两次加锁都会成功.

读锁可以只锁索引,但是写锁会顺便锁上主键索引.

间隙锁可以随便加,但行锁会互斥.

设置wait_timeout即空闲的链接在这个秒数后会断开.

# mysql如何保证数据不丢失

只要写了redo log和binlog就可以保证异常重启数据不丢失.

## binlog写入

事务执行的时候先写binlog cache,事务提交的时候写binlog.
一个事务的binlog是不可拆分的,因此不管事务多大,也要保证单次写入.

binlog cache的大小由,binlog cache size控制,每个线程一个binlog cache
如果单个线程的binlog cache大小超过了限制,就要暂存到磁盘里.(存储常见的套路).

sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync;
sync_binlog=1 的时候，表示每次提交事务都会执行 fsync;
sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync;

## redo log写入

redo log buffer,在提交的时候写到redo log(disk).

设置为 0 的时候,表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
设置为 1 的时候,表示每次事务提交时都将 redo log 直接持久化到磁盘;
设置为 2 的时候,表示每次事务提交时都只是把 redo log 写到 page cache.

read only对super用户是无效的.

xid是用来联系bin log和redo log的,比如redo log里面有一个事务
是prepare状态,但不知道是commit状态,那就可以用xid去bin log查询该事务
是否提交,有提交则是commit状态.若没有提交则回滚该事务.

# 主备一致

bin log的三种类型

limit 1语句的风险:多where条件下可能走不同的索引,不同索引获取的第一条信息不同
如果是写语句,就会修改不同而row.

binlog的三种模式
statement:记录 语句原文.
row:记录的是event即对row做了什么操作.记录了真实操作航的主键id.
mixed=statement+row:因为row会记录这条记录本身,如果写10w行,那binlog
里面也会保存10w行数据,耗费很多io资源,所以则这种格式就由mysql自己判断,
记录那种格式的binlog.

# 高可用

主备延迟最主要的原因是,备库消费日志比主库生产日志要慢.
主备延迟的一些情况:
1.备库cpu满了,读请求太多,影响了数据同步
2.大事务
...

# 多线程复制

1.coordinator负责读取中转日志和分发事务,更新日志由worker线程来执行
数量由slave_parallel_workers来决定

分配事务到worker的两个限制
1.不能造成更新覆盖.更新同一行的两个事务,必须被分发到同一个worker中.
2.同一个事务不能备拆分,必须放到同一个worker中.

## 按照表分发的方案

因为一个事务可能涉及多个worker
所以每次分配的时候应该遍历所以普的worker的等待队列
如果只有一个冲突在放在对应worker的队列后面,如果有一个以上的冲突
则需要等待变成一个冲突再加入队列,保证数据同步的一致性.
如果不冲突那就分配给最闲的worker.

## 按行分发的方案

核心思路就是如果两个事务不更新相同的行,就能在备库上并行执行所以这个模式需要binlog为row格式.

## 按照redo log group分配

能够在同一组里提交的事务,一定不会修改同一行.
主库上可以并行执行的事务,备库上也一定是可以并行执行的。

缺点:一组执行完之后才能执行另一组.

因此，MySQL 5.7 并行复制策略的思想是：
同时处于 prepare 状态的事务，在备库执行时是可以并行的;
处于 prepare 状态的事务，与处于 commit 状态的事务之间，在备库执行时也是可以并行的。

binlog_group_commit_sync_delay 参数，表示延迟多少微秒后才调用 fsync;
binlog_group_commit_sync_no_delay_count 参数，表示累积多少次以后才调用 fsync。

## 参数binlog-transaction-dependency-tracking

hash(值是通过 库名 + 表名 + 索引名 + 值 计算出来的)

COMMIT_ORDER，表示的就是前面介绍的，根据同时进入 prepare 和 commit 来判断是否可以并行的策略。
WRITESET，表示的是对于事务涉及更新的每一行，计算出这一行的 hash 值，组成集合 writeset。
如果两个事务没有操作相同的行，也就是说它们的 writeset 没有交集，就可以并行。WRITESET_SESSION，
是在 WRITESET 的基础上多了一个约束，即在主库上同一个线程先后执行的两个事务，在备库执行的时候，要保证相同的先后顺序。

# 一主多从

基于点位的主备切换:

CHANGE MASTER TO
MASTER_HOST=$host_name
MASTER_PORT=$port
MASTER_USER=$user_name
MASTER_PASSWORD=$password
MASTER_LOG_FILE=$master_log_name
MASTER_LOG_POS=$master_log_pos

这六个参数重要的是最后两个:使用那个日志文件的,那个位置开始.也就是主库对应的文件名和日志偏移量
如何获取 master_log_pos
备库同步完毕,执行show master status,获取备库上最新的file和pos
取主库的的故障时间T
用mysql binlog工具解析备库的file获取T时刻的pos.

同步的过程中可能会有sql重复执行导致的错误,比如主键冲突之类的,只要跳过就好.
set global sql_slave_skip_counter=1;
start slave;

# 读写分离

读写分离策略由于主从延迟,会导致读库上读到旧数据.

## 解决方案:

1. 强制走主库.
2. Sleep 方案
   即主库更新后,读库的读请求sleep 1秒之后在执行,大概率能获取新值
3. 判断主备无延迟方案
   看show slave status的seconds_behind_master.
   Master_Log_File 和 Read_Master_Log_Pos，表示的是读到的主库的最新位点；
   Relay_Master_Log_File 和 Exec_Master_Log_Pos，表示的是备库执行的最新位点。
   当他们相同的时候,则说明同步完成.
   Auto_Position=1 ，表示这对主备关系使用了 GTID 协议。
   Retrieved_Gtid_Set，是备库收到的所有日志的 GTID 集合；
   Executed_Gtid_Set，是备库所有已经执行完成的 GTID 集合。
   如果这两个集合相同，也表示备库接收到的日志都已经同步完成。
   这种方案的问题是:binlog发给备库,但还没执行完,所以还是会查到旧的数据.

4. 配合semi-sync
   也就是半同步复制.
   事务提交的时候主库把binlog发给从库.
   从库收到binlog以后,发回给主库一个ack.
   主库收到ack以后,才给客户端返回 事务完成.
   但是一主多从,还是会有问题.因为只要一个从库ack,就会回应,所以
   其他从库可能还是旧数据.

5. 等主库位点方案
   主动去主库执行 show master status.获取 file 和 pos
   select master_pos_wait(file, pos[, timeout]);
   它是在从库执行的；
   参数 file 和 pos 指的是主库上的文件名和位置；
   timeout 可选，设置为正整数 N 表示这个函数最多等待 N 秒。

6. gtid
   select wait_for_executed_gtid_set(gtid_set, 1);
   等待，直到这个库执行的事务中包含传入的 gtid_set，返回 0；
   超时返回 1。 这样就可以确定事务同步完了没有,防止获取过期数据.

## 实践

* 配置
  在主库上启用二进制日志（Binary Log），这是进行数据复制的基础。通常在/etc/my.cnf或my.ini配置文件中添加或修改以下参数：

1.      [mysqld]
   server-id=1 # 设置主库的唯一ID
   log-bin=mysql-bin # 开启二进制日志功能，并设置日志文件前缀
   binlog_format=row # 建议使用行格式记录事务
2.     [mysqld]
   server-id=2 # 设置从库的唯一ID
3. 在从库上执行 CHANGE MASTER TO 命令，指定主库的信息和复制位置：
   CHANGE MASTER TO
   MASTER_HOST='主库IP',
   MASTER_USER='replication_user', # 复制账号
   MASTER_PASSWORD='replication_password', # 复制密码
   MASTER_LOG_FILE='mysql-bin.000001', # 主库的二进制日志文件名
   MASTER_LOG_POS=123; # 主库的日志起始位置

# 如何判断一个数据库是不是出问题了？

1. select 1
   无法判断并发线程阻塞的问题.
2. select * from mysql.health_check;
   创建一个只有一条数据的表.
   无法检测binlog写满磁盘的情况,更新和事务无法commit的情况,但读是没问题的
3. 更新判断
   update mysql.health_check set t_modified=now();
4. 内部统计
   打开统计配置相关,全开性能降低10%.

# 如何查看死锁

使用show engine innodb status
查看LATESTDETECTED DEADLOCK

由于锁是一个个加的，要避免死锁，对同一组资源，要按照尽量相同的顺序访问；
在发生死锁的时刻，for update 这条语句占有的资源更多，回滚成本更大，所以 InnoDB 选择了回滚成本更小的 lock in share mode
语句，来回滚。

insert,update,delete都会改变间隙锁的区间范围导致死锁.

# 误删数据怎么办

## 误删除行

binlog至少是row格式
对于 insert 语句，对应的 binlog event 类型是 Write_rows event，把它改成 Delete_rows event 即可；
同理，对于 delete 语句，也是将 Delete_rows event 改为 Write_rows event；
而如果是 Update_rows 的话，binlog 里面记录了数据行修改前和修改后的值，对调这两行的位置即可。

如果有多个事务,需要按照顺序来还原

## 误删库/表

这种情况下，要想恢复数据，就需要使用全量备份，加增量日志的方式了。
这个方案要求线上有定期的全量备份，并且实时备份 binlog。

跳过两种
如果原实例没有使用 GTID 模式，只能在应用到包含 12 点的 binlog 文件的时候，先用–stop-position
参数执行到误操作之前的日志，然后再用–start-position 从误操作之后的日志继续执行；
如果实例使用了 GTID 模式，就方便多了。假设误操作命令的 GTID 是 gtid1，那么只需要执行 set gtid_next=gtid1;begin;commit; 先把这个
GTID 加到临时实例的 GTID 集合，之后按顺序执行 binlog 的时候，就会自动跳过误操作的语句。

# kill相关

1. kill query + 线程id
2. kill connection + 线程id
3. kill不是立即生效而是.把对应的线程标记状态.进去innodb之后,检查线程状态,在执行具体逻辑.

# 查询会打爆内存吗?

## server层

1. 获取一行写到net_buffer中.这块内存的大小是由参数net_buffer_length定义的.默认
   大小16k.
2. 重复获取行,知道net_buffer写满,调用网络接口发出去.
3. 如果发送成功,就清空net_buffer,然后继续取下一行,并写入net_buffer.
4. 如果发送函数返回eagain 或 wsaewouldblock,就表示本地网络站写满了,进入等待.
   直到网络栈可以重新写,再继续发送.
5. 结论:分段发送,所以不会打爆内存

## innodb层

buffer pool 使用lru
整个链表5:3分为young区和old区

young区满了,新数据放在old区.

old区的数据页每次被访问:

1. 如果这个数据页在lru链表里面存在超过1秒就把他移动到链表头部.
2. 小于1秒保持位置不变.

# join

Index Nested-Loop Join(被驱动标走索引)
循环嵌套join,驱动表需要全表扫描.所以小表作为驱动表比较好.

如果是Block Nested-Loop Join
还是小表当驱动表好,因为小表分块(block)少.

在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，
过滤完成之后，计算参与 join 的各个字段的总数据量，
数据量小的那个表，就是“小表”，应该作为驱动表。

# join优化

## mrr

1. 从被驱动表的索引里面获取满足条件的记录,将id值放入read_rnd_buffer.
2. 排序read_rnd_buffer
3. 按照排序完的顺序依次到主键索引去获取.排序后大概率在磁盘上是顺序读取.

## bka

1. 不再一行一行的join,现在驱动表获取一批,放入join_buffer

# 临时表为什么可以重名

因为是按照session+thread_id做了前缀.

# 什么时候会使用内部临时表

union的时候
union会去重,但是union all不会.

tmp_table_size用来控制临时表内存大小,默认是16m.

如果内存临时表不够会转为磁盘临时表,默认使用innodb

## group by 优化方法(索引)

generated always 关联更新建立有序索引
alter table t1 add column z int generated always as(id % 100), add index(z);

## group by 优化方法(直接排序)

SQL_BIG_RESULT 告诉优化器不走内存临时表,直接使用文件临时表
select SQL_BIG_RESULT id%100 as m, count(*) as c from t1 group by m;

## 是否需要使用Memory引擎

内存表是hash索引,主键id索引里存放的是数据的位置,而数据部分
则是单独存放在一个内存数组里面.

1. innodb存放数据总是有序的,但是内存表的数据是顺序写入的.
2. 数据文件有空洞的时候,innodb为了保证有序性,只能在固定的位置写入新值
   而内存表找到空位就可以插入新值.
3. 数据位置发生变化innodb只需要修改主键索引,
   而内存表需要修改所有的索引
4. innodb支持变长数据类型,不同记录的长度可能不同;内存表
   不支持blob和text,并且即使定义了varchar(N),
   实际也当作char(N),也就是固定长度字符串来存储,因此
   内存表的每行数据长度相同.

## 指定内存表使用b+树索引,不指定默认使用hash索引

alter table t1 add index a_btree_index using btree (id);

## 内存表不支持行锁,只支持表锁.因此,一张表只要有更新就会堵住其他所有在这个表上的读写操作

# 主键id为什么不连续

AUTO_INCREMENT 是一张表的自增主键计数器

mysql 8.0版本存在了redo log,所以重启可以恢复.

id为0,null自增+1,id有值且插入成功则更新为当前id.

插入失败sql回退,但是AUTO_INCREMENT没有回退

事务回滚也是如此.

批量插入例如insert...select 这种sql无法提前计算出需要的id数目,所以需要锁语句

但是insert...values...是可以从sql计算需要增加多少id的.
所以申请id完成后就可以释放锁了.

但是批量插入如果一个一个申请锁会很慢,所以同一个sql申请获取的锁,是上一次的两倍

# insert语句的锁

insert 唯一键冲突会在索引上加next-ket lock(读锁),阻塞其他写入.

# 怎么最快速的复制一张表

## 导出insert...values 语句并执行

## 导出csv文件

1. 主库执行完csv文件之后,把csv内容写入binlog
2. 备库从binlog中读取csv内容,生成本地csv文件
3. 执行load data.

## 物理拷贝

# grant

给用户赋权的

# 分区表

是mysql自己的分表策略,可以根据一个列的范围自己分成多个表.
这些表在server层看来是一个表,在innodb层看起来是多个表,也就是说中一个表加锁不会影响另一个表.

# 一些额外的问题

使用left join,左表不一定是驱动表.如果需要left join的语义,就不能把被驱动表的字段放在
where条件里面做等值判断或不等判断.必须写在on里.

## 复习bnl

1. 把驱动表内容放入join buffer
2. 顺序读取 被驱动表,先判断on的条件,再判断where条件,符合条件则放入结果集
3. 被驱动表扫面完成后,对于没有被匹配的驱动表行,剩余字段补上null.

如果一条 join 语句的 Extra 字段什么都没写的话，就表示使用的是 Index Nested-Loop Join（简称 NLJ）算法

## Simple Nested Loop Join

1. 取出驱动表每一行数据,到被动表去做全表扫描匹配
2. 匹配成功则做为结果集返回.

## 为什么bnl快

nlj访问下一条记录算是访问一个随机地址的指针操作
bnl访问下一条这是访问数组的下一个元素,属于访问连续内存
所以bnl比njl快.

## distinct 和 group by 的性能

不需要聚合函数的时候distinct 和 group by 的性能一致,语义一致
执行流程

1. 创建一个临时表,临时表有需要去重的字段,给字段加上唯一索引.
2. 遍历表,依次将数据插入临时表,发现唯一键冲突就跳过
3. 遍历完成后将临时表作为结果集返回客户端.

# 自增id用完了怎么办

## 表定义自增值 id

达到上限只够,在申请下一个,得到的值保持不变.

## InnoDB 系统自增 row_id

从0 开始到 2^48-1,之后再次从0开始

## xid

xid是server层维护的

在mysql中对应事务的

来自于内存全局变量global_query_id,到达上限之后从0开始重新计数

## Innodb trx_id

Xid 是由 server 层维护的。InnoDB 内部使用 Xid，就是为了能够在 InnoDB 事务和 server 之间做关联。
但是，InnoDB 自己的 trx_id，是另外维护的。

对于只读事务，InnoDB并不会分配trx_id。
只有update / insert / for update 等修改类语句才会分配trx_id。

只读事务不分配trx_id的好处

1. 减小事务视图里面活跃事务数组的大小.
2. 减少trx_id的申请次数

max_trx_id 会持久化存储，重启也不会重置为 0，那么从理论上讲，
只要一个 MySQL 服务跑得足够久，
就可能出现 max_trx_id 达到 2^48-1 的上限，然后从 0 开始的情况。

当trx_id为2^48-1的时候下一个trx_id就会为0,根据innodb可见性原则
trx_id 2^48-1之前的事务就会读取到trx_id为0的事务,从而脏读.

## thread_id

递增且做了唯一性判断

# 事务可见性

## 读已提交（Read Committed）

在数据库的隔离级别中，读已提交（Read Committed）是比较常用的一个隔离级别。在读已提交隔离级别下，事务提交后，其他事务可以立即看到这个事务所做的修改。
换句话说，在读已提交隔离级别下，事务的可见性是动态的，即事务所做的修改在提交后就可以立即被其他事务看到。
这种动态可见性有一个显著的好处，即能够提高系统的并发性。因为其他事务可以立即看到提交事务所做的修改，所以其他事务不需要等待提交事务的结束，就可以立即开始自己的操作，从而减少了等待时间。
然而，读已提交隔离级别也存在一些问题。例如，由于事务的可见性是动态的，因此其他事务可以看到提交事务所做的一部分修改，而这部分修改可能导致数据不一致或者错误。此外，由于事务的可见性是动态的，因此在同一个事务中，相同的查询可能会返回不同的结果，这也可能导致一些问题。
因此，在使用读已提交隔离级别时，需要仔细考虑事务的可见性问题，并确保应用程序正确处理这些问题。

# MySQL EXPLAIN(执行计划)

1. id:查询的执行顺序,可以帮助查看那个子查询或者连接首先被执行.
2. select_type:查询的类型例如SIMPLE（简单查询）、SUBQUERY（子查询）或 JOIN（连接）等。
3. table:涉及到的表
4. type:表示访问表的方式，通常包括 ALL（全表扫描）、INDEX（索引扫描）等。
5. key:如果使用了索引，表示被使用的索引。
6. key_len:表示索引的长度。
7. row:估计需要扫描的行数。
8. extra:包含其他信息，如是否使用了临时表、文件排序等。
9. possible_keys如果该列为NULL，则说明没有可用的索引。
   如果列中列出了一些索引名，那么这些索引都有可能在执行SQL查询时提高效率，优化器会根据查询的具体条件、表数据分布以及其他因素（如统计信息）来决定最终选择哪个索引来执行查询，这个选定的索引会在key列中体现出来。
