# Kinesis

 Kinesis连接器提供对[Amazon AWS Kinesis Streams的](http://aws.amazon.com/kinesis/streams/)访问。

要使用连接器，请将以下Maven依赖项添加到您的项目中:

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kinesis_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

{% hint style="danger" %}
在Flink 1.10.0版本之前，Flink -connector-kinesis\_2.11依赖于在Amazon软件许可下授权的代码。链接到先前版本的“flink-connector-kinesis”将把这段代码包含到您的应用程序中。
{% endhint %}

## 使用Amazon Kinesis Streams服务

 按照[Amazon Kinesis Streams开发人员指南](https://docs.aws.amazon.com/streams/latest/dev/learning-kinesis-module-one-create-stream.html) 中的说明设置Kinesis流。确保创建适当的IAM策略和用户以读取/写入Kinesis流。

## Kinesis 消费者

FlinkKinesisConsumer是一个完全并行的流数据源，它订阅同一个AWS服务区域内的多个AWS Kinesis流，并且可以在作业运行时透明地处理流的重新分片。使用者的每个子任务负责从多个运动碎片中获取数据记录。每个子任务获取的碎片数量会随着碎片的关闭和由Kinesis创建而改变。

在使用Kinesis流中的数据之前，请确保在AWS仪表板中创建的所有流的状态均为“ ACTIVE”。

{% tabs %}
{% tab title="Java" %}
```java
Properties consumerConfig = new Properties();
consumerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1");
consumerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id");
consumerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key");
consumerConfig.put(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();

DataStream<String> kinesis = env.addSource(new FlinkKinesisConsumer<>(
    "kinesis_stream_name", new SimpleStringSchema(), consumerConfig));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val consumerConfig = new Properties()
consumerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1")
consumerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id")
consumerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key")
consumerConfig.put(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST")

val env = StreamExecutionEnvironment.getEnvironment

val kinesis = env.addSource(new FlinkKinesisConsumer[String](
    "kinesis_stream_name", new SimpleStringSchema, consumerConfig))
```
{% endtab %}
{% endtabs %}

## Kinesis生产者

### 线程模型

### 背压

## 使用自定义Kinesis Endpoint

