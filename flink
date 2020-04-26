flink
一.datastreamapi

1.1.单条记录
filter,map

1.2.windows
时间窗口

1.3.合并流
union把多个数据类型相同的合并
join多条数据合成一条
connect类型不同的流连起来，这条流就有了多种类型

1.4拆分流
split

2.1.DataStream基本转换


3.1.物理分组


二.flink的客户端操作

1.1 5种任务提交方式

1.scala shell
2.sql client
3.commandline
4.restful
5.web


1.2 命令行

bin/flink -h 查看所有命令行
bin/flink run -h 查看某一个命令的参数

1.3 standalone集群启动

bin/start-cluster.sh（本地启动一个flink服务）

可以通过 http://127.0.0.1:8081 访问

run

命令行执行
bin/flink run -d examples/streaming/TopSpeedWindowing.jar

查看任务列表
bin/flink list -m 127.0.0.1:8081

stop停止任务
bin/flink stop -m 127.0.0.1:8081 d67420e52bd051fae2fddbaa79e046bb(job id)

取消任务
bin/flink cancel -m 127.0.0.1:8081 5e20cb6b0f357591171dfcca2eea09de
取消任务。如果在 conf/flink-conf.yaml 里面配置了 state.savepoints.dir，会保存 Savepoint，否则不会保存 Savepoint。
也可以在停止的时候显示指定 Savepoint 目录。
flink-1.7.2 bin/flink cancel -m 127.0.0.1:8081 -s /tmp/savepoint 29da945b99dea6547c3fbafd57ed8759

取消和停止（流作业）的区别如下：
cancel()调用，立即调用作业算子的cancel()方法，以尽快取消他们。如果算子在接到cancel()调用后没有停止，flink将开始定期中断算子线程的执行，直到所有算子停止为止。

stop()是更加优雅的方式，仅适用于source实现了stoppableFunction接口的作业。当用户请求停止作业时，作业的所有source都将被stop()方法调用，指导所有source正常关闭时，作业才会正常结束，这种方式，使作业正常处理完所有作业。

Savepoint
bin/flink savepoint -m 127.0.0.1:8081 ec53edcfaeb96b2a5dadbfbe5ff62bbb /tmp/savepoint
Checkpoint 是增量做的，每次的时间较短，数据量较小，只要在程序里面启用后会自动触发，用户无须感知；Checkpoint 是作业 failover 的时候自动使用，不需要用户指定。

Savepoint 是全量做的，每次的时间较长，数据量较大，需要用户主动去触发。Savepoint 一般用于程序的版本更新（详见文档），Bug 修复，A/B Test 等场景，需要用户指定。

通过 -s 参数从指定的 Savepoint 启动：
bin/flink run -d -s /tmp/savepoint/savepoint-f049ff-24ec0d3e0dc7 ./examples/streaming/TopSpeedWindowing.jar

Modify
修改任务并行度
bin/flink modify -p 4 7752ea7b0e7303c780de9d86a5ded3fa
bin/flink modify -p 3 7752ea7b0e7303c780de9d86a5ded3fa

conf/flink-conf.yaml

taskmanager.numberOfTaskSlots: 4
state.savepoints.dir: file:///tmp/savepoint

修改后需要重启集群再重启任务
bin/stop-cluster.sh && bin/start-cluster.sh
bin/flink run -d examples/streaming/TopSpeedWindowing.jar

info
用来查看flink任务的执行计划的
bin/flink info examples/streaming/TopSpeedWindowing.jar

http://flink.apache.org/visualizer/
将命令输出的json拷贝到这个网站
可以和实际运行的物理计划作对比


web相关
在 Flink Dashboard 页面左侧可以看到有个「Submit new Job」的地方，用户可以上传 Jar 包和显示执行计划和提交任务。Web 提交功能主要用于新手入门和演示用。

三.window和time

1.1 window将有限流切成有限流
window可以是时间驱动，也可以是事件驱动的

window()的参数是一个分发器，负责将数组分发到正确的window中（一条数据可能分发到多个window中）

2.1常见的几种assigner
1.tumbling window(窗口间无重复元素类似set)
2.sliding window(窗口间的元素可能重复)
3.session window（只对一个用户的数据集做时间上的分割）以及global window（对一个用户的所有数据做统计不做时间分割）
4.如果需要自定义的话需要实现class，继承WindowAssigner

evictor主要用于做一些数据的自定义操作
evicBefore 和 evicAfter 两个方法
分别是执行用户代码之前和执行用户代码之后

flink提供了三种evictor
CountEvictor保留指定元素数
DeltaEvictor通过执行用户给定DeltaFunction以及预设的threshold，判断是否删除一个元素
TimeEvictor设定一个阀值interval,删除所有不再max_ts-interval范围内的元素，其中max_ts是窗口内时间戳的最大值

trigger
每个windowassigner都自带一个默认的trigger，如果默认的不满足可以自定义，只要继承trigger即可
1.onElement() 每次往 window 增加一个元素的时候都会触发
2.onEventTime() 当 event-time timer 被触发的时候会调用
3.onProcessingTime() 当 processing-time timer 被触发的时候会调用
4.onMerge() 对两个 trigger 的 state 进行 merge 操作
5.clear()  window 销毁的时候被调用
上面的接口中前三个会返回一个 TriggerResult，TriggerResult 有如下几种可能的选择：
* CONTINUE 不做任何事情
* FIRE 触发 window
* PURGE 清空整个 window 的元素并销毁窗口
* FIRE_AND_PURGE 触发窗口，然后销毁窗口

3.1 Time & Watermark
在 Flink 中 Time  可以分为三种
Event-Time 表示事件发生的时间，Processing-Time 则表示处理消息的时间（墙上时间），Ingestion-Time 表示进入到系统的时间。
env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime); // 设置使用 ProcessingTime

watermark是为了在乱序的数据中保证某个时间段内的数据已经全部得到，就是在指定的window后再收集一段时间
但是真实世界中我们没法得到一个完美的 watermark 数值 — 要么没法获取到，要么耗费太大，因此实际工作中我们会使用近似 watermark  — 生成 watermark(t) 之后，还有较小的概率接受到时间戳 t 之前的数据，在 Flink 中将这些数据定义为 “late elements”, 同样我们可以在 window 中指定是允许延迟的最大时间（默认为 0）

4.1 window的内部实现
每条数据过来之后，会由 WindowAssigner 分配到对应的 Window，当 Window 被触发之后，会交给 Evictor（如果没有设置 Evictor 则跳过），然后处理 UserFunction。而 UserFunction 则是用户编写的代码。

四.状态管理
1.1 无状态计算的例子
单条输入包含所有信息可直接计算
1.1.2 有状态计算的例子(ps:基本可以的出我要做的访问统计实际是一种有状态的计算了。)
单条输入仅包含部分信息，相同输入可能得到不同的输出

1.2 flink的状态类型
1.2.1 managed state & raw state
Managed State 是 Flink 自动管理的 State，而 Raw State 是原生态 State，两者的区别如下：

从状态管理方式的方式来说，Managed State 由 Flink Runtime 管理，自动存储，自动恢复，在内存管理上有优化；而 Raw State 需要用户自己管理，需要自己序列化，Flink 不知道 State 中存入的数据是什么结构，只有用户自己知道，需要最终序列化为可存储的数据结构。

从状态数据结构来说，Managed State 支持已知的数据结构，如 Value、List、Map 等。而 Raw State只支持字节数组 ，所有状态都要转换为二进制字节数组才可以。

从推荐使用场景来说，Managed State 大多数情况下均可使用，而 Raw State 是当 Managed State 不够用时，比如需要自定义 Operator 时，推荐使用 Raw State。

1.2.3 Keyed State & Operator State
Managed State 分为两种，一种是 Keyed State；另外一种是 Operator State。在Flink Stream模型中，Datastream 经过 keyBy 的操作可以变为 KeyedStream 。

每个 Key 对应一个 State，即一个 Operator 实例处理多个 Key，访问相应的多个 State，并由此就衍生了 Keyed State。Keyed State 只能用在 KeyedStream 的算子中，即在整个程序中没有 keyBy 的过程就没有办法使用 KeyedStream。

相比较而言，Operator State 可以用于所有算子，相对于数据源有一个更好的匹配方式，常用于 Source，例如 FlinkKafkaConsumer。相比 Keyed State，一个 Operator 实例对应一个 State，随着并发的改变，Keyed State 中，State 随着 Key 在实例间迁移，比如原来有 1 个并发，对应的 API 请求过来，/api/a 和 /api/b 都存放在这个实例当中；如果请求量变大，需要扩容，就会把 /api/a 的状态和 /api/b 的状态分别放在不同的节点。由于 Operator State 没有 Key，并发改变时需要选择状态如何重新分配。其中内置了 2 种分配方式：一种是均匀分配，另外一种是将所有 State 合并为全量 State 再分发给每个实例。

在访问上，Keyed State 通过 RuntimeContext 访问，这需要 Operator 是一个Rich Function。Operator State 需要自己实现 CheckpointedFunction 或 ListCheckpointed 接口。在数据结构上，Keyed State 支持的数据结构，比如 ValueState、ListState、ReducingState、AggregatingState 和 MapState；而 Operator State 支持的数据结构相对较少，如 ListState。

五.容错机制与故障恢复
1. 状态如何恢复如何保存
肯定是checkpoint和savepoint啊，意料之内
这里有一个点就是如果数据源不支持重发的话任务和状态虽然回归了，但是数据丢失了，这样就会导致计算结果错误。
我暂时想到的方案就是题提高chechpoint的频率，减小结果的误差

1.2 Checkpoint 通过代码的实现方法如下：

首先从作业的运行环境 env.enableCheckpointing 传入 1000，意思是做 2 个 Checkpoint 的事件间隔为 1 秒。Checkpoint 做的越频繁，恢复时追数据就会相对减少，同时 Checkpoint 相应的也会有一些 IO 消耗。

接下来是设置 Checkpoint 的 model，即设置了 Exactly_Once 语义，表示需要 Barrier 对齐，这样可以保证消息不会丢失也不会重复。

setMinPauseBetweenCheckpoints 是 2 个 Checkpoint 之间最少是要等 500ms，也就是刚做完一个 Checkpoint。比如某个 Checkpoint 做了700ms，按照原则过 300ms 应该是做下一个 Checkpoint，因为设置了 1000ms 做一次 Checkpoint 的，但是中间的等待时间比较短，不足 500ms 了，需要多等 200ms，因此以这样的方式防止 Checkpoint 太过于频繁而导致业务处理的速度下降。

setCheckpointTimeout 表示做 Checkpoint 多久超时，如果 Checkpoint 在 1min 之内尚未完成，说明 Checkpoint 超时失败。

setMaxConcurrentCheckpoints 表示同时有多少个 Checkpoint 在做快照，这个可以根据具体需求去做设置。

enableExternalizedCheckpoints 表示下 Cancel 时是否需要保留当前的 Checkpoint，默认 Checkpoint 会在整个作业 Cancel 时被删除。Checkpoint 是作业级别的保存点。

1.3聊聊可选的状态存储方式

Checkpoint 的存储，第一种是内存存储，即 MemoryStateBackend，构造方法是设置最大的StateSize，选择是否做异步快照，这种存储状态本身存储在 TaskManager 节点也就是执行节点内存中的，因为内存有容量限制，所以单个 State maxStateSize 默认 5 M，且需要注意 maxStateSize <= akka.framesize 默认 10 M。Checkpoint 存储在 JobManager 内存中，因此总大小不超过 JobManager 的内存。推荐使用的场景为：本地测试、几乎无状态的作业，比如 ETL、JobManager 不容易挂，或挂掉影响不大的情况。不推荐在生产场景使用。

另一种就是在文件系统上的 FsStateBackend ，构建方法是需要传一个文件路径和是否异步快照。State 依然在 TaskManager 内存中，但不会像 MemoryStateBackend 有 5 M 的设置上限，Checkpoint 存储在外部文件系统（本地或 HDFS），打破了总大小 Jobmanager 内存的限制。容量限制上，单 TaskManager 上 State 总量不超过它的内存，总大小不超过配置的文件系统容量。推荐使用的场景、常规使用状态的作业、例如分钟级窗口聚合或 join、需要开启HA的作业。


还有一种存储为 RocksDBStateBackend ，RocksDB 是一个 key/value 的内存存储系统，和其他的 key/value 一样，先将状态放到内存中，如果内存快满时，则写入到磁盘中，但需要注意 RocksDB 不支持同步的 Checkpoint，构造方法中没有同步快照这个选项。不过 RocksDB 支持增量的 Checkpoint，也是目前唯一增量 Checkpoint 的 Backend，意味着并不需要把所有 sst 文件上传到 Checkpoint 目录，仅需要上传新生成的 sst 文件即可。它的 Checkpoint 存储在外部文件系统（本地或HDFS），其容量限制只要单个 TaskManager 上 State 总量不超过它的内存+磁盘，单 Key最大 2G，总大小不超过配置的文件系统容量即可。推荐使用的场景为：超大状态的作业，例如天级窗口聚合、需要开启 HA 的作业、最好是对状态读写性能要求不高的作业。

六. table api

1.1 
第一，Table API & SQL 是一种声明式的 API。用户只需关心做什么，不用关心怎么做，比如图中的 WordCount 例子，只需要关心按什么维度聚合，做哪种类型的聚合，不需要关心底层的实现。

第二，高性能。Table API & SQL 底层会有优化器对 query 进行优化。举个例子，假如 WordCount 的例子里写了两个 count 操作，优化器会识别并避免重复的计算，计算的时候只保留一个 count 操作，输出的时候再把相同的值输出两遍即可，以达到更好的性能。

第三，流批统一。上图例子可以发现，API 并没有区分流和批，同一套 query 可以流批复用，对业务开发来说，避免开发两套代码。

第四，标准稳定。Table API & SQL 遵循 SQL 标准，不易变动。API 比较稳定的好处是不用考虑 API 兼容性问题。

第五，易理解。语义明确，所见即所得。

table api的编写步骤
1.生成table(输入数据生成table)
2.写sql进行聚合和统计     

生成table的步骤
1.注册对应的tablesource
2.调用table environement的scan方法获取table对象。

注册source的三种方式
1.descriptor
2.自定义source
3.datastream

输出table
Table descriptor, 自定义 Table sink 以及输出成一个 DataStream
                                  
当我们拿到一个 Table 后，调用 groupBy 会返回一个 GroupedTable。GroupedTable 里只有 select 方法，对 GroupedTable 调用 select 方法会返回一个 Table。拿到这个 Table 后，我们可以再调用 Table 上的方法。图中其他 Table，如 OverWindowedTable 也是类似的流程。值得注意的是，引入各个类型的 Table 是为了保证 API 的合法性和便利性，比如 groupBy 之后只有 select 操作是有意义的，在编辑器上可以直接点出来。

一些对列的增强操作
clumns operation
对列的增加，替换，删除和重命名

columns function
选取多列或者反选

七.sql api
1.window aggregation
根据时间窗口对数据的进行聚合计算，并且在到达结束时间时进行结果输出(只输出一次)

2.group aggregation
无限制，数据来一次计算一次，有多次输出

为什么使用kafka和es:因为有相应的组件，配置好相应的yaml后会自动连接，并将结果写入。








