# Streaming File Sink

此连接器提供一个Sink，用于将分区文件写到支持Flink文件系统抽象的文件系统中。

流文件接收器将传入的数据写入存储桶。由于传入的流可以是无限制的，因此每个存储桶中的数据都被组织成有限大小的零件（Part）文件。使用默认的基于时间的存储，可以完全配置存储行为，在该存储中，我们每小时开始写入一个新存储桶。这意味着每个结果存储桶都将包含文件，这些文件包含从流中每隔1小时接收一次的记录。

存储桶目录中的数据被拆分为零件文件。对于已接收到该存储桶数据的接收器的每个子任务，每个存储桶将至少包含一个零件文件。其他零件文件将根据可配置的滚动策略创建。默认策略根据大小滚动零件文件，指定文件可以打开的最大持续时间的超时以及关闭文件后的最大不活动超时。

{% hint style="info" %}
重要说明：使用StreamingFileSink时需要启用检查点。零件文件只能在成功的检查点上完成。如果禁用了检查点，则零件文件将永远处于“进行中”或“待处理”状态，并且下游系统无法安全地读取它们。
{% endhint %}



![](../../.gitbook/assets/image%20%2837%29.png)



## 文件格式

同时`StreamingFileSink`支持按行和批量编码格式，例如[Apache Parquet](http://parquet.apache.org/)。这两个变体带有各自的构建器，可以使用以下静态方法来创建它们：

* 行编码接收器： `StreamingFileSink.forRowFormat(basePath, rowEncoder)`
* 批量编码接收器： `StreamingFileSink.forBulkFormat(basePath, bulkWriterFactory)`

创建行或大容量编码接收器时，我们必须指定存储桶的基本路径以及数据的编码逻辑。

请检出JavaDoc for [StreamingFileSink](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/StreamingFileSink.html)的所有配置选项，以及有关实现不同数据格式的更多文档。

### **行编码格式**

行编码格式需要指定一个编码器，用于将单个行序列化到正在处理的零件文件的OutputStream。

除了存储桶分配器，[RowFormatBuilder还](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/StreamingFileSink.RowFormatBuilder.html)允许用户指定：

* 自定义[RollingPolicy](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/RollingPolicy.html)：滚动策略以覆盖DefaultRollingPolicy
* bucketCheckInterval（默认= 1分钟）：用于检查基于时间的滚动策略的毫秒间隔

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

### Bulk**编码格式**

Bulk编码的接收器与行编码的接收器的创建方式类似，但是这里我们必须指定BulkWriter.Factory，而不是指定编码器。BulkWriter逻辑定义了如何添加、刷新新元素，以及如何为进一步的编码目的而最后确定大量记录。

Flink有三个内置的BulkWriter工厂:

* [ParquetWriterFactory](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/parquet/ParquetWriterFactory.html)
* [SequenceFileWriterFactory](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/sequencefile/SequenceFileWriterFactory.html)
* [CompressWriterFactory](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/compress/CompressWriterFactory.html)

{% hint style="info" %}
 **重要说明：**Bulk格式只能具有“ OnCheckpointRollingPolicy”，它会（仅）在每个检查点上滚动。
{% endhint %}

#### **Parquet格式**

Flink包含内置的便捷方法，用于为Avro数据创建Parquet编写器工厂。这些方法及其相关文档可在[ParquetAvroWriters](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/parquet/avro/ParquetAvroWriters.html)类中找到。

为了写入其他兼容Parquet的数据格式，用户需要使用[ParquetBuilder](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/parquet/ParquetBuilder.html)接口的自定义实现来创建[ParquetWriterFactory](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/formats/parquet/ParquetBuilder.html)。

要在应用程序中使用Parquet Bulk编码器，需要添加以下依赖项：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-parquet_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

可以这样创建一个将Avro数据写入Parquet格式的StreamingFileSink：

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.formats.parquet.avro.ParquetAvroWriters;
import org.apache.avro.Schema;


Schema schema = ...;
DataStream<GenericRecord> stream = ...;

final StreamingFileSink<GenericRecord> sink = StreamingFileSink
	.forBulkFormat(outputBasePath, ParquetAvroWriters.forGenericRecord(schema))
	.build();

input.addSink(sink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink
import org.apache.flink.formats.parquet.avro.ParquetAvroWriters
import org.apache.avro.Schema

val schema: Schema = ...
val input: DataStream[GenericRecord] = ...

val sink: StreamingFileSink[GenericRecord] = StreamingFileSink
    .forBulkFormat(outputBasePath, ParquetAvroWriters.forGenericRecord(schema))
    .build()

input.addSink(sink)
```
{% endtab %}
{% endtabs %}

#### **Hadoop SequenceFile格式**

要在应用程序中使用SequenceFile Bulk编码器，需要添加以下依赖项：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-sequence-file</artifactId>
  <version>1.10.0</version>
</dependency>
```

可以这样创建一个简单的SequenceFile编写器：

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.configuration.GlobalConfiguration;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.SequenceFile;
import org.apache.hadoop.io.Text;


DataStream<Tuple2<LongWritable, Text>> input = ...;
Configuration hadoopConf = HadoopUtils.getHadoopConfiguration(GlobalConfiguration.loadConfiguration());
final StreamingFileSink<Tuple2<LongWritable, Text>> sink = StreamingFileSink
  .forBulkFormat(
    outputBasePath,
    new SequenceFileWriterFactory<>(hadoopConf, LongWritable.class, Text.class))
	.build();

input.addSink(sink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink
import org.apache.flink.configuration.GlobalConfiguration
import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.io.LongWritable
import org.apache.hadoop.io.SequenceFile
import org.apache.hadoop.io.Text;

val input: DataStream[(LongWritable, Text)] = ...
val hadoopConf: Configuration = HadoopUtils.getHadoopConfiguration(GlobalConfiguration.loadConfiguration())
val sink: StreamingFileSink[(LongWritable, Text)] = StreamingFileSink
  .forBulkFormat(
    outputBasePath,
    new SequenceFileWriterFactory(hadoopConf, LongWritable.class, Text.class))
	.build()

input.addSink(sink)
```
{% endtab %}
{% endtabs %}

SequenceFileWriterFactory支持其他构造函数参数来指定压缩设置。

## 桶分配（Bucket）

Bucket逻辑定义了如何将数据组织成基本输出目录内的子目录。

行格式和Bulk格式\(请参阅文件格式\)都使用`DateTimeBucketAssigner`作为默认分配器。默认情况下，`DateTimeBucketAssigner`基于系统默认时区创建每小时一个桶，格式如下:yyyy-MM-dd——HH。日期格式\(即存储段大小\)和时区都可以手动配置。

我们可以通过在格式生成器上调用`BucketAssigner.withbucketassigner (assigner)`来自定义指定。

Flink带有两个内置的`BucketAssigners`：

* [DateTimeBucketAssigner](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/bucketassigners/DateTimeBucketAssigner.html)：默认的基于时间的分配器
* [BasePathBucketAssigner](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/bucketassigners/BasePathBucketAssigner.html)：将所有零件文件存储在基本路径中的分配器（单个全局存储区）

## 回滚策略

RollingPolicy定义了何时关闭给定的正在处理的零件文件，并将其移动到挂起状态，以及稍后移动到完成状态。处于“完成”状态的零件文件是准备好要查看的文件，并且保证包含有效的数据，在出现故障时不会恢复这些数据。滚动策略与检查点间隔\(下一个检查点将完成挂起文件\)相结合，控制部件文件对下游读者可用的速度以及这些部件的大小和数量。

Flink带有两个内置的RollingPolicies：

* [DefaultRollingPolicy](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/rollingpolicies/DefaultRollingPolicy.html)
* [OnCheckpointRollingPolicy](https://ci.apache.org/projects/flink/flink-docs-release-1.10/api/java/org/apache/flink/streaming/api/functions/sink/filesystem/rollingpolicies/OnCheckpointRollingPolicy.html)

## 零件文件生命周期

为了在下游系统中使用StreamingFileSink的输出，我们需要了解所生成的输出文件的命名和生命周期。

零件文件可以处于以下三种状态之一：

1. **正在进行中**：当前正在写入的零件文件正在进行中
2. **待处理**：已关闭（由于指定的滚动策略）正在等待提交的进行中文件
3. **已完成**：在成功的检查点上，待处理文件转换为“已完成”

下游系统只能安全读取完成的文件，因为这些文件以后不会被修改。

{% hint style="info" %}
 **重要信息：**对于任何给定的子任务，零件文件索引都严格增加（按创建顺序）。但是，这些索引并不总是顺序的。当作业重新启动时，所有子任务的下一个零件索引将是“最大零件索引+1”，其中“ max”是在所有子任务中计算的。
{% endhint %}

**零件文件示例**

为了更好地理解这些文件的生命周期，让我们看一个带有2个接收器子任务的简单示例：

```text
└── 2019-08-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    └── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
```

`part-1-0`滚动零件文件时（假设它变得太大），它将变为挂起状态，但不会重命名。然后，接收器将打开一个新的零件文件`part-1-1`：

```text
└── 2019-08-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

由于`part-1-0`现在是等待完成后，下一个成功的检查点后，最后定稿：

```text
└── 2019-08-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

按照存储区策略规定创建新的存储桶，这不会影响当前正在进行的文件：

```text
└── 2019-08-25--12
    ├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── part-1-0
    └── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
└── 2019-08-25--13
    └── part-0-2.inprogress.2b475fec-1482-4dea-9946-eb4353b475f1
```

旧的存储桶仍然可以接收新记录，因为存储存储策略是按每个记录进行评估的。

## 零件文件配置

完成的文件仅通过命名方案就可以与正在进行的文件区分开来。

默认情况下，文件命名策略如下:

* **进行中/待审核**：`part-<subtaskIndex>-<partFileIndex>.inprogress.uid`
* **已完成：** `part-<subtaskIndex>-<partFileIndex>`

 Flink允许用户为其零件文件指定前缀和/或后缀。可以使用`OutputFileConfig`来完成。例如，对于前缀“ prefix”和后缀“ .ext”，接收器将创建以下文件：

```text
└── 2019-08-25--12
    ├── prefix-0-0.ext
    ├── prefix-0-1.ext.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
    ├── prefix-1-0.ext
    └── prefix-1-1.ext.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

 用户可以通过以下方式指定`OutputFileConfig`：

{% tabs %}
{% tab title="Java" %}
```java
OutputFileConfig config = OutputFileConfig
 .builder()
 .withPartPrefix("prefix")
 .withPartSuffix(".ext")
 .build();
            
StreamingFileSink<Tuple2<Integer, Integer>> sink = StreamingFileSink
 .forRowFormat((new Path(outputPath), new SimpleStringEncoder<>("UTF-8"))
 .withBucketAssigner(new KeyBucketAssigner())
 .withRollingPolicy(OnCheckpointRollingPolicy.build())
 .withOutputFileConfig(config)
 .build();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val config = OutputFileConfig
 .builder()
 .withPartPrefix("prefix")
 .withPartSuffix(".ext")
 .build()
            
val sink = StreamingFileSink
 .forRowFormat(new Path(outputPath), new SimpleStringEncoder[String]("UTF-8"))
 .withBucketAssigner(new KeyBucketAssigner())
 .withRollingPolicy(OnCheckpointRollingPolicy.build())
 .withOutputFileConfig(config)
 .build()
```
{% endtab %}
{% endtabs %}

## 重要注意事项

### 一般

{% hint style="danger" %}
**重要提示1**:当使用Hadoop &lt; 2.7时，请使用OnCheckpointRollingPolicy在每个检查点上滚动零件文件。原因是，如果零件文件“遍历”检查点间隔，那么，在从故障中恢复时，StreamingFileSink可能会使用文件系统的truncate\(\)方法从正在处理的文件中丢弃未提交的数据。这个方法在2.7之前的Hadoop版本中是不支持的，Flink会抛出一个异常。
{% endhint %}

{% hint style="danger" %}
**重要提示2**:考虑到Flink sink和udf通常不区分正常的作业终止\(例如，有限的输入流\)和由于失败而终止的作业，在正常的作业终止之后，最后一个正在进行的文件将不会过渡到“完成”状态。
{% endhint %}

{% hint style="danger" %}
**重要说明3**：Flink和StreamingFileSink永远不会覆盖已提交的数据。 鉴于此，当尝试从旧的检查点/保存点还原时（假定该文件是由后续成功的检查点提交的正在处理的文件），Flink将拒绝继续进行操作，并且将由于无法定位正在处理的文件而引发异常。
{% endhint %}

### S3特有的

{% hint style="danger" %}
对于S3，`StreamingFileSink` 仅支持[基于Hadoop的](https://hadoop.apache.org/) FileSystem实现，而不支持基于[Presto](https://prestodb.io/)的实现。如果您的作业使用`StreamingFileSink`写入S3但您想使用基于Presto的作业进行检查点，建议明确使用_“s3a://”_（对于Hadoop）作为接收器目标路径的方案，_“s3p://”_用于检查点（对于Presto）。对于接收器和检查点使用_“s3://”_可能会导致不可预测的行为，因为两种实现都“监听”该方案。
{% endhint %}

{% hint style="danger" %}
 为了确保语义一次高效而有效，请`StreamingFileSink`使用[分段上传](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html) S3的功能（从现在开始为MPU）。此功能允许以独立的块（因此称为“多部分”）上载文件，当成功地将MPU的所有部分上载时，可以将其合并到原始文件中。对于不活动的MPU，S3支持存储桶生命周期规则，用户可以使用该规则来中止在启动后指定天数内未完成的分段上传。这意味着，如果您主动设置此规则并在一些零件文件未完全上传的情况下获取保存点，则与它们关联的MPU可能会在重新启动作业之前超时。这将导致您的作业无法从该保存点还原，因为未决的零件文件不再存在，并且Flink会尝试获取并失败而失败，并出现异常。
{% endhint %}

