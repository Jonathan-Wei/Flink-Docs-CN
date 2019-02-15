# 流文件Sink

此连接器提供一个Sink，用于将分区文件写到支持Flink文件系统抽象的文件系统中。

{% hint style="danger" %}
对于S3，`StreamingFileSink` 仅支持[基于Hadoop的](https://hadoop.apache.org/) FileSystem实现，而不支持基于[Presto](https://prestodb.io/)的实现。如果您的作业使用`StreamingFileSink`写入S3但您想使用基于Presto的作业进行检查点，建议明确使用_“s3a://”_（对于Hadoop）作为接收器目标路径的方案，_“s3p://”_用于检查点（对于Presto）。对于接收器和检查点使用_“s3://”_可能会导致不可预测的行为，因为两种实现都“监听”该方案。
{% endhint %}

由于在流处理中输入可能是无限的，所以流文件接收器将数据写入Bucket中。Bucket行为是可配置的，但一个有用的默认设置是基于时间的Bucket，在这种情况下，我们每小时编写一个新Bucket，从而获得各自包含无限输出流的单个片段文件。

在Bucket中，我们进一步根据滚动策略将输出分割成更小的片段文件。这有助于防止单个bucket文件变得太大。这也是可配置的，但是默认策略根据文件大小和超时滚动文件。_即_如果没有新数据写入片段文件。

该`StreamingFileSink`同时支持逐行编码\(row-wise\)格式和块编码\(bulk-encoding\)格式，如 [Apache Parquet](http://parquet.apache.org/)。

**使用行编码输出格式**

惟一需要的配置是要输出数据的基本路径和用于将记录序列化到每个文件的OutputStream的编码器。

基本用法如下:

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;

DataStream<String> input = ...;

final StreamingFileSink<String> sink = StreamingFileSink
	.forRowFormat(new Path(outputPath), new SimpleStringEncoder<>("UTF-8"))
	.build();

input.addSink(sink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink

val input: DataStream[String] = ...

val sink: StreamingFileSink[String] = StreamingFileSink
    .forRowFormat(new Path(outputPath), new SimpleStringEncoder[String]("UTF-8"))
    .build()
    
input.addSink(sink)
```
{% endtab %}
{% endtabs %}

以上代码将创建一个每小时创建一个Bucket并使用默认滚动策略的流接收器。默认Bucket分配器是 [DateTimeBucketAssigner](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/bucketassigners/DateTimeBucketAssigner.html) ，默认滚动策略是[DefaultRollingPolicy](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/rollingpolicies/DefaultRollingPolicy.html)。您可以 在接收器构建器上指定自定义 [BucketAssigner](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/BucketAssigner.html) 和 [RollingPolicy](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/RollingPolicy.html)。请查看JavaDoc for [StreamingFileSink](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/StreamingFileSink.html) 以获取更多配置选项以及有关Bucket分配器和滚动策略的工作和交互的更多文档。

**使用批量编码的输出格式**

在上面的例子中，我们使用了一个`Encoder`可以单独编码或序列化每个记录的。流式文件接收器还支持批量编码的输出格式，如[Apache Parquet](http://parquet.apache.org/)。要使用它们，你将使用`StreamingFileSink.forBulkFormat()`而不是`StreamingFileSink.forBulkFormat()`并指定`BulkWriter.Factory`。

[ParquetAvroWriters](https://ci.apache.org/projects/flink/flink-docs-master/api/java/org/apache/flink/formats/parquet/avro/ParquetAvroWriters.html) 具有用于创建`BulkWriter.Factory`各种类型的静态方法。

{% hint style="info" %}
重要信息：批量编码格式只能与`OnCheckpointRollingPolicy`结合使用，后者会在每个检查点上滚动正在进行的部分文件。
{% endhint %}

