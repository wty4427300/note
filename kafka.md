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
public List<ConsumerRecord<K, V>>records(TopicPartition partition){};
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

从上次的消费位置开始消费
endOffsets
对应的就有beginningOffsets,但它不会一直是0,
因为日志清理会清除老的数据
public void seekToBeginning(Collection<TopicPartition> partitions)
public void seekToEnd(Collection<TopicPartition> partitions)
也可以通过kafka提供的这两个方法从开头或者结尾消费,更加简洁
public Map<TopicPartition, OffsetAndTimestamp> offsetsForTimes(
Map<TopicPartition, Long> timestampsToSearch)
public Map<TopicPartition, OffsetAndTimestamp> offsetsForTimes(
Map<TopicPartition, Long> timestampsToSearch,
Duration timeout)
通过这些方法可以追溯到具体的时间开始消费,map key为分区,value为时间戳

## 再均衡

是指分区所属权从一个消费者转移到另一个消费者的行为

* 作用是为消费组高可用和伸缩性提供保障，但是再均衡发生期间，消费组内的消费者无法读取消息。
* 如果一个消费者消费完毕但是没有提交消费偏移量，此时发生再均衡，分区分配给另一个消费者，就会发生重复消费

再均衡监听器用来设定发生再均衡动作前后的一些准备或收尾的动作。
ConsumerRebalanceListener 是一个接口

1. void onPartitionsRevoked(Collection partitions)
   这个方法会在再均衡开始之前和消费者停止读取消息之后被调用。可以通过这个回调方法来处理消费位移的提交，以此来避免一些不必要的重复消费现象的发生。参数
   partitions 表示再均衡前所分配到的分区。
2. void onPartitionsAssigned(Collection partitions) 这个方法会在重新分配分区之后和消费者开始读取消费之前被调用。参数
   partitions 表示再均衡后所分配到的分区。

## 消费者拦截器

ConsumerInterceptor

* public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records)；
* public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets)；
* public void close()。

KafkaConsumer 会在 poll() 方法返回之前调用拦截器的 onConsume() 方法来对消息进行相应的定制化操作，比如修改返回的消息内容、按照某种规则过滤消息（可能会减少
poll() 方法返回的消息的个数）。如果 onConsume() 方法中抛出异常，那么会被捕获并记录到日志中，但是异常不会再向上传递。

KafkaConsumer 会在提交完消费位移之后调用拦截器的 onCommit() 方法，可以使用这个方法来记录跟踪所提交的位移信息，比如当消费者使用
commitSync 的无参方法时，我们不知道提交的消费位移的具体细节，而使用拦截器的 onCommit() 方法却可以做到这一点。

close() 方法和 ConsumerInterceptor 的父接口中的 configure() 方法与生产者的 ProducerInterceptor 接口中的用途一样，这里就不赘述了。

## 重要的消费者参数

### fetch.min.bytes

该参数用来配置consumer再一次拉去请求(调用poll()方法)中能从kafka中拉取的最小数，默认只为1b。
如果返回给consumer的数据量小于这个配置的值，那么他就需要等待，直到数据量满足这个参数的配置大小。
适当调大这个参数的值可以提高吞吐量，不过也会造成额外的延迟。

### fetch.max.bytes

一次拉去的最大值，默认为52428800（B），也就是50MB。但该大小，不是绝对的。
当一个非空分区中拉去的第一条消息是大于该值的，那么该消息仍然返回，以确保消费
者继续工作。

### fetch.max.wait.ms

如果 Kafka 仅仅参考 fetch.min.bytes 参数的要求，那么有可能会一直阻塞等待而无法发送响应给 Consumer，显然这是不合理的。
fetch.max.wait.ms 参数用于指定 Kafka 的等待时间。

### max.partition.fetch.bytes

这个参数用来配置从每个分区里返回给 Consumer 的最大数据量，默认值为1048576（B），即1MB。这个参数与 fetch.max.bytes 参数相似
前者用来限制一次拉取中每个分区的消息大小，后者用来限制一次拉取中整体消息的大小，同样如果参数设置比消息的大小要小，那么也不会造成无法消费
，kafka为了保证消费逻辑的正常运转不会对此做强硬的限制。

## max.poll.records

这个参数用来配置 Consumer 在一次拉取请求中拉取的最大消息数，默认值为500（条）。如果消息的大小都比较小，则可以适当调大这个参数值来提升一定的消费速度。

## connections.max.idle.ms

这个参数用来指定在多久之后关闭闲置的连接，默认值是540000（ms），即9分钟

## exclude.internal.topics

__consumer_offsets 和 __transaction_state。exclude.internal.topics 用来指定 Kafka 中的内部主题是否可以向消费者公开，默认值为
true。如果设置为 true，那么只能使用 subscribe(Collection)的方式而不能使用 subscribe(Pattern)的方式来订阅内部主题，设置为
false 则没有这个限制。

## receive.buffer.bytes

这个参数用来设置 Socket 接收消息缓冲区（SO_RECBUF）的大小，默认值为65536（B），即64KB。如果设置为-1，则使用操作系统的默认值。如果
Consumer 与 Kafka 处于不同的机房，则可以适当调大这个参数值。

## send.buffer.bytes

这个参数用来设置Socket发送消息缓冲区（SO_SNDBUF）的大小，默认值为131072（B），即128KB。与receive.buffer.bytes参数一样，如果设置为-1，则使用操作系统的默认值。

## request.timeout.ms

这个参数用来配置 Consumer 等待请求响应的最长时间，默认值为30000（ms）。

## metadata.max.age.ms

这个参数用来配置元数据的过期时间，默认值为300000（ms），即5分钟。如果元数据在此参数所限定的时间范围内没有进行更新，则会被强制更新，即使没有任何分区变化或有新的
broker 加入。

## reconnect.backoff.ms

这个参数用来配置尝试重新发送失败的请求到指定的主题分区之前的等待（退避）时间，避免在某些故障情况下频繁地重复发送，默认值为100（ms）。

## isolation.level

这个参数用来配置消费者的事务隔离级别。字符串类型，有效值为“read_uncommitted”和“read_committed”，表示消费者所消费到的位置，如果设置为“read_committed”，那么消费者就会忽略事务未提交的消息，即只能消费到LSO（LastStableOffset）的位置，默认情况下为“read_uncommitted”，即可以消费到
HW（High Watermark）处的位置。
