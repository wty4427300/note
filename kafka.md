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
A
手动提交分为同步提交和异步提交

```java
public class ConsumerRecord<K, V> {
    private final String topic;
    private final int partition;
    private final long offset;
    private final long timestamp;
    //timestampType 有两种类型：CreateTime和LogAppendTime，分别代表消息创建的时间戳和消息追加到日志的时间戳。
    private final TimestampType timestampType;
    private final int serializedKeySize;
    private final int serializedValueSize;
    //headers 表示消息的头部内容。
    private final Headers headers;
    private final K key;
    private final V value;
    private volatile Long checksum;
    //省略若干方法
}
```

ConsumerRecords返回若干消息集合,提供Iterator方法来遍历

```java
public List<ConsumerRecord<K, V>> records(TopicPartition partition);
```

也可以这样根据partition来消费

# 5.消费投递模式

1. 点对点:点对点模式是基于队列的，消息生产者发送消息到队列，消息消费者从队列中接收消息
2. 订阅:发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题（Topic）

* 如果所有的消费者都隶属于同一个消费组，那么所有的消息都会被均衡地投递给每一个消费者，即每条消息只会被一个消费者处理，这就相当于点对点模式的应用。
* 如果所有的消费者都隶属于不同的消费组，那么所有的消息都会被广播给所有的消费者，即每条消息会被所有的消费者处理，这就相当于发布/订阅模式的应用。

# 6.消息消费

## lastConsumedOffset

当前消费到的位置

## position

下一次拉取的消息位置

## committed offset

已经提交过的消费位移

## 自动提交

kafka默认设置是自动提交的(定期提交)
由配置enable.auto.commit决定是否自动提交
由配置auto.commit.interval.ms决定多少秒提交一次

## 手动提交

手动提交可以细分为同步提交和异步提交，对应于 KafkaConsumer 中的 commitSync() 和 commitAsync() 两种类型的方法。

区别就是提交的时候是否会阻塞消费者线程.

## 重新消费

Kafka 允许你提交任何你想要的 offset，包括之前已经提交过的 offset。如果你提交了 offset=4，然后下次提交 offset=2，消费者将会从
offset=2 的位置重新开始消费消息。
这种情况通常用于重新消费之前的消息，可能是因为某些处理失败或者需要重新处理之前的数据。

## 消费的暂停和恢复

public void pause(Collection<TopicPartition> partitions)
public void resume(Collection<TopicPartition> partitions)
还是以partition为粒度

## 退出消费

使用wakeup(),并且他是线程安全的.会抛出一个WakeupException.
使用这种方式需要手动关闭资源
public void close()
public void close(Duration timeout)
@Deprecated
public void close(long timeout, TimeUnit timeUnit)

## auto.offset.reset

![img_4.png](data%2Fimg_4.png)
当一个新的消费者被创建的时候,它没有可查询的消费位移.
这时候kafka就会根据auto.offset.reset的配置来决定从何处消费

* 默认值为latest,从分区末尾开始消费
* earliest,从0开始消费
* none,找不到消费位移抛出异常(NoOffsetForPartitionException)
* 如果配置错误则报 ConfigException

## 从特定位置消费
