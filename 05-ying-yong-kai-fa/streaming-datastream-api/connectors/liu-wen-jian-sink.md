# 流文件Sink

此连接器提供一个Sink，用于将分区文件写到支持Flink文件系统抽象的文件系统中。



![](../../../.gitbook/assets/image%20%2821%29.png)



## 文件格式

同时`StreamingFileSink`支持按行和批量编码格式，例如[Apache Parquet](http://parquet.apache.org/)。这两个变体带有各自的构建器，可以使用以下静态方法来创建它们：

* 行编码接收器： `StreamingFileSink.forRowFormat(basePath, rowEncoder)`
* 批量编码接收器： `StreamingFileSink.forBulkFormat(basePath, bulkWriterFactory)`

创建行或大容量编码接收器时，我们必须指定存储桶的基本路径以及数据的编码逻辑。

请检出JavaDoc for [StreamingFileSink](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/StreamingFileSink.html)的所有配置选项，以及有关实现不同数据格式的更多文档。

**行编码格式**

惟一需要的配置是要输出数据的基本路径和用于将记录序列化到每个文件的OutputStream的编码器。

基本用法如下:

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;

DataStream<String> input = ...;

final StreamingFileSink<String> sink = StreamingFileSink
    .forRowFormat(new Path(outputPath), new SimpleStringEncoder<String>("UTF-8"))
    .withRollingPolicy(
        DefaultRollingPolicy.builder()
            .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
            .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
            .withMaxPartSize(1024 * 1024 * 1024)
            .build())
	.build();

input.addSink(sink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy

val input: DataStream[String] = ...

val sink: StreamingFileSink[String] = StreamingFileSink
    .forRowFormat(new Path(outputPath), new SimpleStringEncoder[String]("UTF-8"))
    .withRollingPolicy(
        DefaultRollingPolicy.builder()
            .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
            .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
            .withMaxPartSize(1024 * 1024 * 1024)
            .build())
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

## 重要注意事项

### 一般

### S3特有的

{% hint style="danger" %}
对于S3，`StreamingFileSink` 仅支持[基于Hadoop的](https://hadoop.apache.org/) FileSystem实现，而不支持基于[Presto](https://prestodb.io/)的实现。如果您的作业使用`StreamingFileSink`写入S3但您想使用基于Presto的作业进行检查点，建议明确使用_“s3a://”_（对于Hadoop）作为接收器目标路径的方案，_“s3p://”_用于检查点（对于Presto）。对于接收器和检查点使用_“s3://”_可能会导致不可预测的行为，因为两种实现都“监听”该方案。
{% endhint %}

{% hint style="danger" %}
 为了确保语义一次高效而有效，请`StreamingFileSink`使用[分段上传](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html) S3的功能（从现在开始为MPU）。此功能允许以独立的块（因此称为“多部分”）上载文件，当成功地将MPU的所有部分上载时，可以将其合并到原始文件中。对于不活动的MPU，S3支持存储桶生命周期规则，用户可以使用该规则来中止在启动后指定天数内未完成的分段上传。这意味着，如果您主动设置此规则并在一些部分文件未完全上传的情况下获取保存点，则与它们关联的MPU可能会在重新启动作业之前超时。这将导致您的作业无法从该保存点还原，因为未决的零件文件不再存在，并且Flink会尝试获取并失败而失败，并出现异常。
{% endhint %}

