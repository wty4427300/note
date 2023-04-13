# 事物的四大特性

1.原子性：是一个不可分割的整体，要么全都执行，要么全不执行。执行出错事务回滚。
2.隔离性：同一时间，只允许一个事务请求同一组数据。不同事物彼此之间没有干扰。
3。一致性：事务开始前和开始后。数据库的完整性约束没有被破坏。
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

读是事务开始时的read-view(视图)读.
写是,读后写,即已经提交的事务的更新是可以看到的,保证更新不丢失.

# row的版本

row可能会有多个版本,版本由trx_id(事务id)表示.

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

# 