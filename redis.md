# Redis 线程模型演进史

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


# 主从结构


