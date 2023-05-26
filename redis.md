_# Redis 线程模型演进史

1. redis是一个单线程应用
   redis在处理客户端请求的时候,都是由唯一的主线程进行处理的,其中包括了请求的读取和解析,
   命令的执行和响应的回复,都是由唯一的主线程进行处理的.
   4.0版开始redis使用一些后台线程处理比较耗时的操作.例如,清理脏数据,释放超时链接,删除大key
   等等.但是网络读写和执行命令还是使用单线程来处理.
2. redis使用单线程的原因
   redis执行的是纯内存操作,瓶颈不在cpu,而是在网络和内存,单线程避免了不必要的上下文切换
   和竞争条件,也无需考虑锁的问题.单线程模型也使得代码会相对的简单.
3. redis多线程的改进
   当redis server收到一个请求的时候,会先进入io(网络)队列.多个io线程会从io队列
   里面读取请求,并进行解析,解析之后的命令就会交给redis主线程进行执行.执行后的结果
   会重新进入io队列,等待io线程将响应返回给客户端.

# redis多线程模型

## redis 6.0之前的线程模型

是一个主线程和三个后台线程的模型

## 三个后台线程分别是

1. close_file:关闭aof,rdb等过程中产生的大临时文件
2. aof_fsync:将追加至aof文件的数据刷盘
3. lazy_free:惰性释放大对象

## 网络模块引入多线程

1. 使用pthread_create创建线程指定IOThreadMain为处理函数
2. 看IOThreadMain的代码得知,IO 线程是从 io_threads_list 队列
   (或者说列表)获取待处理的客户端，并根据操作类型选择具体的执行逻辑
   也就是一个经典的生产者消费者模式.

## 多线程处理读请求的逻辑

处理客户端建连请求的逻辑 acceptTcpHandler()
客户端连接上可读事件的处理函数是 CT_Sokcet->ae_handler
它实际指向了 connSocketEventHandler() 函数
connSocketEventHandler() 函数里可以同时处理可读事件和可写事件，
默认会先处理可读事件，然后再处理可写事件。
调用方可以在 connection->flags 字段中设置 CONN_FLAG_WRITE_BARRIER 标志位
connection->read_handler 指向的实际就是
readQueryFromClient() 函数。
readQueryFromClient() 中的第一步就是调用
postponeClientRead() 函数即延迟读取,如果开启了多线程模式
postponeClientRead() 就会将读时间的client添加到
redisServer.clients_pending_read队列中,
多线程模式下这种，主线程通过队列可读事件分配给 IO 线程进行处理的行为，
就是前面说的“延迟读取”.
每次处理网络事件之前，都会调用 aeEventLoop->beforesleep 指向的函数
beforeSleep()->handleClientsWithPendingReadsUsingThreads()
io_threads_list[i]会通过取余的方式均匀的分配给每个io thread对应的client链表
io线程做读取和解析两步工作,之后全部交给主线程执行,执行完了以后
由putClientInPendingWriteQueue()写入clients_pending_write队列
然后和上面的一样由
beforeSleep()->handleClientsWithPendingWritesUsingThreads()
分配给io线程.具体写回的逻辑在writeToClient()里面

# string

# list

# set

# hash

hsetnx 可以判断一个key是否写入了,key以存在就无法写入.
hexists 判断key是否存在,存在返回1,不存在返回.
HINCRBY 给value做递增可以加一个正数或者一个负数,从而实现加减法.
HSCAN 获取一个hash表所有的内容,但是一次只返回一小部分,分多次返回,不会阻塞redis

# sortSet

# sds

## 为什么需要sds

1. 为了存储 \o 这种特殊字符,如果用c本身的字符串 \o 就是字符串的结尾了
2. c的字符串只是一个char数组,没有length的属性.需要使用的话,得遍历计算.
3. 扩容问题,sds会多申请预留空间,这样字符串增长也不需要扩容.

redis sds有5个不同的结构体,分别使用uint-uint64表示字符串最大长度.
不同大小的字符串使用合适的结构体存储,最大程度的内存节省空间.

sds禁止了内存对齐,因为需要使用指针前后移动获取字段值,为了对齐插入额外的字段
会导致取到空白值.

unichar flags 大小为5bit,使用低3位,0-4,来表示sds的五种类型.

# 内存淘汰策略

1. no-enviction:当内存满的时候,该策略不会触发淘汰,但是当客户端
   再次发送新建key的请求或是修改已有key的请求时,redis会直接返回错误
   但是del和get是正常响应的.

2. volatile-lru:从设置了过期时间的key中选取一些进行淘汰,先采样一部分key
   ,然后淘汰采样后的最近没有被访问的key.

3. allkeys-lru:从所有的key从,淘汰最近最少使用的key.

4. volatile-random: 从设置了过期时间的key里随机淘汰.

5. allkeys-random:从所有key里随机淘汰.

6. volatile-ttl:从设置了过期时间的key中筛选将要过期的key进行淘汰.

7. volatile-lfu:过期时间的 Key 中，筛选出最近访问频率最低的 Key 进行淘汰。

8. allkeys-lfu 策略：使用近似 LFU 算法从全部 Key 中，筛选出最近访问频率最低的 Key 进行淘汰。

注:lru最近最少使用,lfu最近访问频率最低.

# redis的key失效是如何实现的

1. 定期扫描部分key判断key的时间戳是否是已经失效
2. 惰性失效,即每次请求到对应key的时候判断是否过期

# rdb

给redis的整个内存做了快照，然后存储到.rdb文件里面

## 触发方式

1. 手动
   执行save 或者 bgsave
   save会使用主线做持久化，会导致整个redis服务不可用
   bgsave则是fork出一个子进程来进行rdb持久化，不会阻塞主线程

2. 当redis正常关停的时候会触发一次rdb

3. 通过config set save x y
   将rdb设置成距上次rdb持久化超过x秒，并且有y个key被修改过

4. 落盘过程

先打开一个临时文件temp-进程号。rdb
然后使用rioFile写入
通过fflush() 和 fsync()刷盘
调用rname()函数系应该文件名
如果不配置的话默认是dump.rdb

# aof

Append Only File,即只做增量持久化.核心思路就是将redis执行过的每条命令
都保存到aof文件当中.

在实际生产环境中,一般会使用rdb+aof的方案:

1. rdb定时全量持久化,而rdb和故障之间的这个时间段则是用aof进行持久化.

## aof落盘的三种策略

1. no
   不会主动调用fsync()刷盘,完全依赖操作系统.

2. always
   每次写入aof的时候
   都会调用fsync进行刷盘
   在主线程完成

3. everysec
   每秒执行一次fsync()进行刷盘.该策略是上述两种策略的折中.最多丢失1秒的
   aof日志.在后台线程完成

## aof的rewrite机制

类似于lsm tree的键值合并,只留下最后的结果,减小日志的大小.

# 高并发,高可用

## 主从架构

一主多从,主写从读.

## Redis replication 的核心机制

redis采取的是异步复制方式.

1. 初次连接,全量复制,master启动后台线程生成rdb文件
   同时将客户端新收到的命令缓存在内存中,rdb生成后发给slave.
   收到后先落盘再加载到内存里,然后master将内存中缓存的新数据
   发给slave.

2. slave不会主动过期key,当master过期key时会给slave模拟一条
   del命令

## sentinel哨兵模式

1. 启动单独的进程对redis集群进行管理
   集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
   消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
   故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
   配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

2. 哨兵本身也可以集群运行
   配置quorum=1
   意味着只要有一个哨兵认为master宕机了就可以进行切换
   此时必须有一半以上的哨兵是可以运行的

3. 配置 quorum=2 ，如果 M1 所在机器宕机了，
   那么三个哨兵还剩下 2 个，S2 和 S3 可以一致认为
   master 宕机了，然后选举出一个来执行故障转移，
   同时 3 个哨兵的 majority 是 2，
   所以还剩下的 2 个哨兵运行着，
   就可以允许执行故障转移.

## 哨兵主备切换数据丢失的问题

1. 异步复制导致的数据丢失,当master->slave的复制还未结束
   此时,master宕机,这部分数据就丢失了.
2. 脑裂导致的数据丢失,即master因为网络脱离了集群,集群选出了另一个master
   ,此时虽然slave切换到了新的master,但是某个client可能还连接着老的
   master,他会向老的master写入新数据,那么对于新的master而言
   这部分数据就丢失了.

3. 解决方案
   min-slaves-to-write 1
   min-slaves-max-lag 10
   意思就是至少有一个slave,数据复制和同步延迟不能超过10秒
   一旦超过这个时间,master就不会接受任何请求了.

   这个方案最丢失10秒的数据.

## sdown 和 odown 转换机制

sdown 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机
odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机
sdown 达成的条件很简单，如果一个哨兵 ping 一个 master，超过了 is-master-down-after-milliseconds 指定的毫秒数之后，
就主观认为 master 宕机了；如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就认为是
odown 了。

## 哨兵集群的自动发现机制

使用pub/sub实现
哨兵会往__sentinel__:hello channel中发送消息
其他哨兵可以消费这条消息.
消息内容就是自己的:host,ip和runid还有master.

## slave配置的自动纠错

如果slave成为master候选人,那么哨兵会确保,slave复制现有的master的数据.
如果slave连接到一个错误的master上,比如故障转移之后,哨兵确保,slave连接到
正确的master.

## slave->master 选举算法

