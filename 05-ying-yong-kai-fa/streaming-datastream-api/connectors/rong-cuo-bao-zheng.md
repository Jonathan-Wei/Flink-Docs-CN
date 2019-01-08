# 容错保证

## Sink和Source的容错保证

Flink的容错机制在出现故障时恢复程序并继续执行。此类故障包括机器硬件故障，网络故障，瞬态程序故障等。

只有当源加入快照机制时，Flink才能保证一次状态更新为用户定义的状态。下表列出了Flink与捆绑连接器的状态更新保证。

请阅读每个连接器的文档以了解容错保证的详细信息。

| Source | 保障 | 说明 |
| :--- | :--- | :--- |
| Apache Kafka | 一次 | 根据版本使用适当的Kafka连接器 |
| AWS Kinesis Streams | 一次 |  |
| RabbitMQ | 最多一次（v 0.10）/一次（v 1.0） |  |
| Twitter Streaming API | 最多一次 |  |
| Collections | 一次 |  |
| Files | 一次 |  |
| Sockets | 最多一次 |  |

为了保证端到端精确一次的记录传递（除了精确一次的状态语义之外），数据接收器需要加入检查点机制。下表列出了Flink与捆绑接收器的传送保证（假设一次状态更新）：

| Sink | 保障 | 说明 |
| :--- | :--- | :--- |
| HDFS rolling sink | 一次 | 实现取决于Hadoop版本 |
| Elasticsearch | 至少一次 |  |
| Kafka producer | 至少一次 |  |
| Cassandra sink | 至少一次/恰好一次 | 只有一次幂等更新 |
| AWS Kinesis Streams | 至少一次 |  |
| File sinks | 至少一次 |  |
| Socket sinks | 至少一次 |  |
| Standard output | 至少一次 |  |
| Redis sink | 至少一次 |  |

