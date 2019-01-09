# 概述

## 预定义的Source和Sink

Flink内置了一些基本的数据Source和Sink，并且总是可用的。预定义的数据源包括从文件、目录、Socket、集合\(Collection\)以及迭代器\(Iterators\)。预定义的数据接收器\(Sink\)支持写入文件、stdout和stderr以及Socket。

## 捆绑的连接器\(Connectors\)

连接器提供用于与各种第三方系统连接的代码。目前支持这些系统：

* [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/kafka.html)（Source/Sink）
* [Apache Cassandra](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/cassandra.html)（Sink）
* [亚马逊Kinesis Streams](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/kinesis.html)（Source/Sink）
* [Elasticsearch](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/elasticsearch.html)（Sink）
* [Hadoop文件系统](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/filesystem_sink.html)（Sink）
* [RabbitMQ](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/rabbitmq.html)（Source/Sink）
* [Apache NiFi](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/nifi.html)（Source/Sink）
* [Twitter Streaming API](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/twitter.html)（Source）

{% hint style="info" %}
请记住，要在应用程序中使用其中一个连接器，通常需要其他第三方组件，例如数据存储或消息队列的服务器。另请注意，虽然本节中列出的流连接器是Flink项目的一部分，并且包含在源版本中，但它们不包含在二进制分发版中。可以在相应的小节中找到进一步的说明。
{% endhint %}

{% hint style="info" %}
另请注意，虽然本节中列出的流连接器是Flink项目的一部分，并且包含在源版本中，但它们不包含在二进制分发版中。可以在相应的小节中找到进一步的说明
{% endhint %}

## Apache Bahir中的连接器\(Connectors\)

Flink的其他流媒体连接器正在通过[Apache Bahir](https://bahir.apache.org/)发布，包括：

* [Apache ActiveMQ](https://bahir.apache.org/docs/flink/current/flink-streaming-activemq/)（Source/汇）
* [Apache Flume](https://bahir.apache.org/docs/flink/current/flink-streaming-flume/)（Sink）
* [Redis](https://bahir.apache.org/docs/flink/current/flink-streaming-redis/)（Sink）
* [Akka](https://bahir.apache.org/docs/flink/current/flink-streaming-akka/)（Sink）
* [Netty](https://bahir.apache.org/docs/flink/current/flink-streaming-netty/)（Sink）

## 其他连接到Flink的方法

### 通过异步IO进行数据填充

使用连接器不是将数据输入和输出Flink的唯一方法。一种常见的模式是在一个`Map`或多个`FlatMap` 中查询外部数据库或Web服务以丰富主数据流。Flink提供了一个用于[异步I / O](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/asyncio.html)的API， 以便更有效，更稳健地进行这种丰富。

### 可查询状态\(Queryable State\)

当Flink应用程序将大量数据推送到外部数据存储时，这可能成为I/O瓶颈。如果所涉及的数据的读操作比写操作少得多，则外部应用程序可以更好地从Flink中提取所需的数据。[可查询状态](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/queryable_state.html)\(Queryable State\)接口通过允许按需查询Flink管理的状态来实现这一点。

