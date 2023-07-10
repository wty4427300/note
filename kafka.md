# 1.一个topic可以有多个partition

每一个partition是一个文件，每一次添加新消息，相当于对一个log日志做追加操作

因为每个partition是一个文件，所以单个partition内可以保持消息的有序性

一个broker是一个kafka实例

一个topic的多个partition可以跨多个broker

# 2.生产者是线程安全的可以多线程共享一个

生产send分为同步和异步两种方式：差距就是异步有callback（）函数

# 3.生产者由两根线程协调
主线程 和 发送线程
主线程：主要是做拦截，序列化，分区器最后缓存到消息累加器
发送线程：从消息累加器获取消息并发送到kafka中

消息的生产实际上是一个写log的方式，如果定义出消息累加器做缓存，希望可一次多发一些，减少网络io，那么就需要在消息累加器里面记录写入partition的索引
消息累加器的默认大小是32m。

消息累加器为每个分区都维护一个双端队列，写入时追加到队列尾部，发送时从头部取出。
也就是说消息累加器维护多个partition

发送的时候因为是按照字节发送的，所以需要频繁申请bytebuffer,所以消息累加器内部维护了一个bufferpool,用来buffer复用，他只能复用固定大小的
默认16k可以调节

会有一个map缓存发送但未响应的消息k,v(nodeid,request)

# 4.消费者相关

如果连续订阅两次topic以第二次为主

消费者的assign方法可以订阅某些topic的特定partition

partitionsFor(String topic)可以查看topic里面有多少分区

//当topic的集合为空时这三行代码的效果是一样的
consumer.unsubscribe();
consumer.subscribe(new ArrayList<String>());
consumer.assign(new ArrayList<TopicPartition>());

位移提交是可配置的

enable.auto.commit配置为true,则为自动提交

手动提交分为同步提交和异步提交







