关于flink的10个问题

1.阐述 Flink 如何处理反压，相比 Storm，Spark  Streaming 提供的反压机制，描述其实现有什么不同？

2.阐述流处理引擎提供的三种数据处理语义，解释 Flink  Checkpoint 机制如何保证 Flink 程序结果的 Exactly-Once 语义，描述如何通过两阶段提交协议提供端到端的 Exactly-Once 保证？结合 Kafka 如何构建端到端的 Exactly-Once 处理？

这一条差不多了解了

3.阐述 Flink 提供的容错机制，解释分布式快照 Chandy  Lamport 算法逻辑，剖析 Flink  Checkpoint 具体实现流程？

4.如何处理 Flink 作业频繁重启问题？

5.如何优化大状态的 Flink 作业？

6.如何排查 Flink Checkpoint 超时问题？

7.如何处理 Flink 作业中的数据倾斜问题？

8.Flink 反压机制，如何排查反压瓶颈在哪，及如何处理反压问题？

9.哪种 join 可以满足单个流断流的时候仍然能够保证正确的 join 到数据？

10.watermark 是怎么生成和传递的？

dispatcher用来接收用户上传的作业

jobmanager用来编译作业，然后向resourcemanager申请资源并执行作业


