# HDFS

此连接器提供一个Sink，用于将分区文件写入[Hadoop FileSystem](http://hadoop.apache.org/)支持的任何文件系统 。要使用此连接器，请将以下依赖项添加到项目中：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-filesystem_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

{% hint style="info" %}
请注意，Streaming Connectors当前不是二进制分发包的一部分。 有关如何使用库打包程序以执行集群的信息，请参考 [此处](https://ci.apache.org/projects/flink/flink-docs-master/dev/linking.html)。
{% endhint %}

**Bucketing File Sink**

可以配置bucketing行为和写入操作，稍后我们将讨论这一点。以下是创建Bucketing接收器的方法，默认情况下，该接收器将接收到按时间拆分的滚动文件：

{% tabs %}
{% tab title="Java" %}
```java
DataStream<String> input = ...;

input.addSink(new BucketingSink<String>("/base/path"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[String] = ...

input.addSink(new BucketingSink[String]("/base/path"))
```
{% endtab %}
{% endtabs %}

有两个配置选项指定何时应关闭片段文件并启动新片段文件：

* 通过设置批量大小（默认部件文件大小为384 MB）
* 通过设置批次滚动时间间隔（默认滚动间隔为`Long.MAX_VALUE`）

当满足这两个条件中的任何一个时，将启动新的片段文件。

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Tuple2<IntWritable,Text>> input = ...;

BucketingSink<String> sink = new BucketingSink<String>("/base/path");
sink.setBucketer(new DateTimeBucketer<String>("yyyy-MM-dd--HHmm", ZoneId.of("America/Los_Angeles")));
sink.setWriter(new SequenceFileWriter<IntWritable, Text>());
sink.setBatchSize(1024 * 1024 * 400); // this is 400 MB,
sink.setBatchRolloverInterval(20 * 60 * 1000); // this is 20 mins

input.addSink(sink);

```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[Tuple2[IntWritable, Text]] = ...

val sink = new BucketingSink[String]("/base/path")
sink.setBucketer(new DateTimeBucketer[String]("yyyy-MM-dd--HHmm", ZoneId.of("America/Los_Angeles")))
sink.setWriter(new SequenceFileWriter[IntWritable, Text]())
sink.setBatchSize(1024 * 1024 * 400) // this is 400 MB,
sink.setBatchRolloverInterval(20 * 60 * 1000); // this is 20 mins

input.addSink(sink)
```
{% endtab %}
{% endtabs %}

这将创建一个接收器，该接收器将写入遵循此模式的存储Bucket文件：

```text
/base/path/{date-time}/part-{parallel-task}-{count}
```

其中`date-time`是我们从日期/时间格式中得到的字符串，`parallel-task`是`ParallelSink`实例的索引，`count`是由于批处理大小或批处理滚动间隔而创建的部件文件的运行数量。

有关详细信息，请参阅JavaDoc for [BucketingSink](http://flink.apache.org/docs/latest/api/java/org/apache/flink/streaming/connectors/fs/bucketing/BucketingSink.html)。

