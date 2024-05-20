# 动机

设计为一个统一的平台，用于处理所有实时数据源。
高吞吐，例如实时日志聚合
优雅的处理大数据积压，以便能够支持来自离线系统的定期数据加载
低延迟交付，以便处理更传统的消息传递用例
分区，分布式，实时粗粒，创建新的feed，容错

# 持久性

## 使用文件系统

kafka严重依赖文件系统来存储和缓存消息。人们普遍认为“磁盘很慢”，这使人们怀疑持久性结构能否提供有竞争力的性能。
事实上，磁盘比人们预期的要慢得多，也快得多，这取决于它们的使用方式;正确设计的磁盘结构通常可以与网络一样快。

在具有六个 7200rpm SATA RAID-5 阵列的 JBOD 配置上，线性写入的性能约为 600MB/秒，但随机写入的性能仅为 100k/秒左右，相差超过
6000 倍。
在某些情况下顺序磁盘访问可能比随机内存访问更快。

当我们使用文件io时，自己在进程中维护进程内缓存，这些数据也会在PageCache中缓存，相当于缓存了两次。

* kafka实在jvm之上构建的，研究过java内存的人都知道

1. 对象内存开销非常高，通常会使存储的数据大小增加一倍（或更糟）。
2. 随着堆内存的增加，java垃圾回收变得越来越繁琐和更缓慢。

所以使用文件系统并依赖PageCache,优于维护内存中缓存和其他结构。存储紧凑的字节结构而不是单个对象，这样做
再 32GB 机器上缓存高达 28-30GB，而不会受到 GC 惩罚。即使重启缓存也是热数据，这也简化了代码因为所有用
于维护缓存和文件系统之间一致性的逻辑现在都在操作系统中。如果您的磁盘使用偏向于线性读取，那么预读会有效地用
每个缓存上的有用数据预先填充此缓存磁盘读取。

## 选择队列为数据结构

为什么使用队列而不适用btrees，因为btrees的操作是ologn，随着数据增加性能降低，而队列操（读取和追加到文件上）作是o1，因此性能和
数据大小完全解耦。

## 效率

### 当消除了磁盘的不良访问，那么此类系统效率低下有两个常见的原因:

1. 小型io操作过多
2. 字节复制过多


* 服务器接受和发送消息，都是多条合并发送，而不是一条一条的发送。
  持久化的时候也是一次将消息块附加到日志中
* 使用一种标准化的二进制消息格式
* 消息集合使用相同的格式写入磁盘，所以持久日志块的网络传输，可以使用内核函数
  sendfile完成，sendfile提供了高度优化的代码路径 out of pagecache到socket

### 一般情况下数据从文件传输到套接字的通用数据路径

1. 操作系统将数据从磁盘读取到内核空间中的页面缓存中
2. 应用程序将数据从内核空间读取到用户空间缓冲区中
3. 应用程序将数据写回内核空间的套接字缓冲区
4. 操作系统将数据从套接字缓冲区复制到网卡缓冲区，在那里通过网络发送数据

* 这一系列操作有四个缓存两个系统调用，显然是低效的。使用sendfile允许操作系统将数据
  从pagecache直接发送到网络，避免了过多复制。

* pagecache 和 sendfile 的这种组合意味着，在消费者连接的Kafka集群上，
  你将在磁盘上看不到任何读取活动，因为它们将完全从缓存中提供数据。

### 端到端批量压缩

瓶颈实际上不是 CPU 或磁盘，而是网络带宽。对于需要通过广域网在数据中心之间发送消息的数据管道来说尤其如此。
Kafka 通过高效的批处理格式支持这一点。一批消息可以聚集在一起，压缩并以这种形式发送到服务器。
这批消息将以压缩形式编写，并将在日志中保持压缩状态，并且只会由使用者解压缩。

# 生产者

## 负载均衡

* Kafka节点都可以响应有关元数据的请求，让生产者选择发送给那个分区域。
* 生产者客户端也可选择使用一些负载均衡策略来完成发送。对应的接口默认通过
  允许用户指定要分区的键并使用该键对分区进行哈希处理
* 例如，如果密钥 选择的是用户 ID，则给定用户的所有数据都将发送到同一分区。

## 异步发送

批处理是效率的重要驱动因素之一，为了实现批处理，kafka生产者将尝试在内存中
积累数据，并在单个请求中发送更大的批处理，批处理可以配置积累不超过固定数量
的消息，等待时间不超过某个固定的延迟限制。这允许积累更多的字节发哦是那个，
并且对服务器，这种缓冲是可配置的，并提供了一种机制，可以权衡少量的额外延迟
以获得更好的吞吐量。

# 消费者

向想要的topic发送请求，可以指定offset，来决定从哪个位置开始拉取，也可以
在需要的时候重放数据。

## 推与拉

我们的目标是为了让消费者尽可能的高速消费。

### 推拉的差异

* 在推模式下当消费者的消费速率低于生产速率时，消费者往往会不知所措（本质上是拒绝服务攻击，因为处理不过来）
* 在拉模式下消费速率慢的时候，消费者只是落后，可以在之后赶上。这可以通过某种回退协议来缓解，通过该协议，消费者可以指示它不堪重负

### 拉模式的优劣

* 它有助于对发送给消费者的数据进行积极的批处理。基于推送的系统必须选择立即发送请求或积累更多数据发送，
  然后在不知道下游消费者是否能够立即处理它的情况下稍后发送它。如果针对低延迟进行调整这将导致传输最终都会被缓冲，这是浪费。
  而拉去模式记录了日志的偏移量，因此总能获取需要的最佳一批数据。


* 缺陷在于当broker没有数据时，消费者会一直空轮询，知道有效数据到达，为了避免这种情况。
  我们在拉取消息的时候，允许消费者请求在长轮询中中止，等待数据到达(并可以选择等待给定数量的字节)

## 消费者的立场

* 跟踪已使用的内容是消息传递系统的关键性能点之一。

大部分系统都会保留broker上已经消费地消息的元数据。也就是说，当消息分发给消费者时。
broker要么立即在本地记录该数据，要么等待消费者确认(acknowledgement).这是一个务实
的选择broker知道消费了那些消息，他就可以删除这些消息，保持较小的数据大小。

## 消息传递的语义

* 最多一次：消息可能会丢失，但永远不会重新传递
* 至少一次：消息永远不会丢失，但可以重新传递。
* 只有一次：每条消息只传递一次，而且只有一次。

这里分为两个问题：发布消息的持久性保证和使用消息时的保证。

生产者和broker之间提供了至少一次的语义，当生产者把消息提交给broker时，
如果没有收到ack，则会重复发送，为了保证幂等，broker会给生产者分配一个id
生产者发送的每一个序列号，两者结合确定消息的唯一性，保证日志中消息不重复。

### 最多一次的实现

即消费者收到消息，在处理消息之前就commit。这样偏移量已经提交，假如已经commit
的消息还没有处理完消费者就崩溃了，那么消费者重启后也会在commit之后的消息位移开始
消费就做到了，消息可能会丢失，但永远不会重新传递。

### 至少一次实现

即消费者读取消息，处理消息，最后commit。这样在消息commit之前消费者崩溃了，重启之后
就冲重新发送消息，包括已处理未提交的消息，这样就做到了消息永远不会丢失，但可以重新传递。
所以这里处理消息，需要做幂等。

## 只有一次实现

* 需要两个机制

1. 事务支持：Kafka 提供了生产者事务的支持，这使得生产者能够在多个主题或分区上原子地发送一组消息。通过事务，生产者可以在确认消息发送之前将其持久化到Kafka，
   并且在发送失败时可以回滚事务，从而确保消息不会丢失。
2. Exactly-Once语义的消费：在消费端，Kafka 0.11 版本引入了幂等性和事务读（transactional
   reads）的概念。消费者可以通过幂等性来确保消息被消费一次且仅一次。此外，消费者还可以使用事务读来在消费消息的同时提交消费偏移量的事务，确保偏移量与消息的原子性一致性。

# 复制

kafka在配置数量的服务器之间复制每个topic分区的日志，每个topic可以设置自己的复制因子。
这允许在一些情况下，可以故障转移到这些副本，当集群中的服务器出现故障的时候，消息故障仍然
可用。

在 Kafka 中，每个主题（topic）的每个分区（partition）都有一个领导者（Leader）。在 KRaft 模式下，每个分区的领导者通过 Raft
协议选举产生，其他副本则成为跟随者（Followers）。因此，如果你有多个主题，每个主题中的每个分区都会有一个领导者。这意味着在 Kafka
集群中，可能会存在多个领导者，每个领导者负责管理各自分区的数据复制和数据读写操作。

复制的单位是topic的partition。在非故障条件下，Kafka中的每个分区都有一个leader和0个或多个follower。
所有的写入都转入分区的leader，读取可以转到分区的领导或跟随。通常，topic比broker多得多，并且leader在
broker之间均匀分配。follower的日志是与leader相同的，所有消息都具有相同的偏移量和相同的顺序（当然，在任何给定时间，领导者的日志末尾都可能有一些尚未复制的消息）。

follower像普通的kafka消费者一样使用来自领导的消息，并将其应用到他们自己的日志中。

* 在 Kafka 中，一个特殊的节点 称为“控制器”，负责管理集群中broker的注册。broker 活性有两个条件：

1. broker必须与控制器保持活动会话，以便接收定期元数据更新。
2. 作为follower的broker必须复制leader的写操作，而不是“太落后”。

# Exactly-Once 语义实现原理：幂等性与事务消息

事务消息最主要的动机是在流处理中实现 Exactly Once 的语义，这可以分为：

1. 仅发送一次： 单分区仅发送一次由生产者幂等保证，多分区仅发送一次由事务机制保证

2. 仅消费一次： Kafka 通过消费位点的提交来控制消费进度，而消费位点的提交被抽象成向系统 topic
   发送消息。这就使得发送和消费行为统一起来，只要解决了多分区发送消息的一致性就能实现 Exactly Once 语义

## 生产者幂等性

在创建 Kafka 生产者时设置了 enable.idempotence 参数，用于开启生产者幂等性。

Kafka 的发送幂等是通过序列号来实现的，每个消息都会被分配一个序列号，序列号是递增的，这样就可以保证消息的顺序性。当生产者发送消息时，会将消息的序列号和消息内容一起写入到日志文件中，下次收到非预期序列号的消息就会返回
OutOfOrderSequenceException 异常。

设置 enable.idempotence 参数后，生产者会检查以下三个参数的值是否合法（ProducerConfig#postProcessAndValidateIdempotenceConfigs）

* max.in.flight.requests.per.connection 必须小于 5

* retries 必须大于 0

* acks 必须设置为 all

Kafka 将消息的序列号信息保存在分区维度的 .snapshot 文件中，格式如下（ProducerStateManager#ProducerSnapshotEntrySchema）：

![img_7.png](img%2Fimg_7.png)

我们可以发现，该文件中保存了 ProducerId、ProducerEpoch 和 LastSequence。所以幂等的约束为：相同分区、相同 Producer（id 和
epoch） 发送的消息序列号需递增。即 Kafka 的生产者幂等性只在单连接、单分区生效，Producer 重启或消息发送到其他分区就失去了幂等性的约束。

.snapshot 文件在 log segment 滚动时更新，发生重启后通过读取 .snapshot 文件和最新的日志文件即可恢复 Producer 的状态。Broker
的重启或分区迁移并不会影响幂等性。

## 如何使用 Kafka 客户端完成一个事务：

```

// 事务初始化
val props = new Properties()
...
props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, transactionalId)
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true")

val producer = new KafkaProducer(props)
producer.initTransactions()
producer.beginTransaction()

// 消息发送
producer.send(RecordUtils.create(topic1, partition1, "message1"))
producer.send(RecordUtils.create(topic2, partition2, "message2"))

// 事务提交或回滚
producer.commitTransaction()
```

### 事务初始化

Kafka Producer 启动后我们使用两个 API 来初始化事务：initTransactions 和 beginTransaction。

回顾一下我们的 Demo，在发送消息时是发送到两个不同分区中，这两个分区可能在不同的 Broker 上，所以我们需要一个全局的协调者
TransactionCoordinator 来记录事务的状态。

所以，在 initTransactions 中，Producer 首先发送 ApiKeys.FIND_COORDINATOR 请求获取 TransactionCoordinator。

之后即可向其发送 ApiKeys.INIT_PRODUCER_ID 请求获取 ProducerId 及 ProducerEpoch（也是上文中用于幂等的字段）。此步骤生成的 id
和 epoch 会写入内部 Topic __transaction_state 中，并且将事务的状态置为 Empty。

__transaction_state 是 compaction Topic，其中消息的 key 为客户端设置的transactional.id（详见
TransactionStateManager#appendTransactionToLog）。

区别于 ProducerId 是服务端生成的内部属性；TransactionId 由用户设置，用于标识业务视角认为的“同一个应用”，启动具有相同
TransactionId 的新 Producer 会使得未完成的事务被回滚并且来自旧 Producer（具有较小 epoch）的请求被拒绝掉。

后续 beginTransaction 用于开始一个事务，该方法会创建一个 Producer 内部事务状态，标识这一个事务的开始，并不会有 RPC 产生。

### 消息发送

上一节说到 beginTransaction 只是更改 Producer 内部状态，那么在第一条消息发送时才隐式开启了事务：

首先，Producer 会发送 ApiKeys.ADD_PARTITIONS_TO_TXN 请求到 TransactionCoordinator。TransactionCoordinator
会将这个分区加入到事务中，并更改事务的状态为 Ongoing，这些信息被持久化到 __transaction_state 中。

然后 Producer 使用 ApiKeys.PRODUCE 请求正常发送消息到对应的分区中。这条消息的可见性控制在下文消息消费一节中会详细讨论。

### 事务提交与回滚

当所有消息发送完成后，Producer 可以选择提交或回滚事务，此时：

ꔷ TransactionCoordinator：具有当前事务所有相关分区的信息

ꔷ 其他 Broker：已经将消息持久化到日志文件中

接下来 Producer 调用 commitTransaction 会发送 ApiKeys.END_TXN 请求将事务状态更改为 PrepareCommit（回滚事务对应状态
PrepareAbort）并持久化到 __transaction_state 中，此时从 Producer 的视角来看整个事务已经结束了。

TransactionCoordinator 会异步向各个 Broker 发送 ApiKeys.WRITE_TXN_MARKERS 请求，当所有参加事务的 Broker
都返回成功后，TransactionCoordinator 会将事务状态更改为 CompleteCommit（回滚事务对应状态 CompleteAbort）并持久化到 __
transaction_state 中。

### 消息的消费

在 Broker 处理 ApiKeys.PRODUCE 请求时，完成消息持久化会更新 LSO 到第一条未提交的事务消息的 offset。这样在消费者消费消息时，可以通过
LSO 来判断消息是否可见：如果设置了 isolation.level 为 read_committed 则只会消费 LSO 之前的消息。

* LSO（log stable offset）: 它表示的是已经被成功复制到所有副本（replicas）并且可以被消费者安全消费的消息的最大偏移量。

## acid保证

ꔷ 原子性（Atomicity）

Kafka 通过对 __transaction_state Topic 的写入实现了事务状态的转移，保证了事务要么同时提交，要么同时回滚。

ꔷ 一致性（Consistency）

在事务进入 PrepareCommit 或 PrepareAbort 阶段时， TransactionCoordinator 异步向所有参与事务的 Broker 提交或回滚事务。这使得
Kafka 的事务做不到强一致性，只能通过不断重试保证最终一致性。

ꔷ 隔离性（Isolation）

Kafka 通过 LSO 机制和 .txnindex 文件来避免脏读，实现读已提交（Read Committed）的隔离级别。

ꔷ 持久性（Durability）

Kafka 通过将事务状态写入到 __transaction_state Topic 和消息写入到日志文件中来保证持久性。