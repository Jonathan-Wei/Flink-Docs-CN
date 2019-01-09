# RabbitMQ

## RabbitMQ连接器许可证 <a id="rabbitmq-connector"></a>

Flink的RabbitMQ连接器定义了对“RabbitMQ AMQP Java客户端”的Maven依赖，在Mozilla Public License 1.1（“MPL”），GNU通用公共许可证版本2（“GPL”）和Apache许可证版本2下获得三重许可。 （“ASL”）。

Flink本身既不重用“RabbitMQ AMQP Java Client”的源代码，也不从“RabbitMQ AMQP Java Client”打包二进制文件。

基于Flink的RabbitMQ连接器创建和发布衍生作品的用户（从而重新分发“RabbitMQ AMQP Java客户端”）必须意识到这可能受到Mozilla公共许可证1.1（“MPL”），GNU中声明的条件的约束。通用公共许可证版本2（“GPL”）和Apache许可证版本2（“ASL”）。

## RabbitMQ连接器 <a id="rabbitmq-connector"></a>

此连接器提供对[RabbitMQ](http://www.rabbitmq.com/)数据流的访问。要使用此连接器，请将以下依赖项添加到项目中：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-rabbitmq_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

请注意，流连接器当前不是二进制分发的一部分。请参阅[此处的](https://ci.apache.org/projects/flink/flink-docs-master/dev/linking.html)链接以进行群集执行。

### 安装RabbitMQ

按照[RabbitMQ下载页面](http://www.rabbitmq.com/download.html)的说明进行操作。安装后，服务器自动启动，并且可以启动连接到RabbitMQ的应用程序。

**RabbitMQ Source**

此连接器提供了一个`RMQSource`使用RabbitMQ队列消息的类。此源提供三种不同级别的保证，具体取决于它如何配置使用：

1. **完全一次\(Exactly-once\)**：为了实现RabbitMQ源的一次性保证，需要以下内容 :
   * _启用检查点_：_启用检查_点后，只有在检查点完成时才会确认消息（因此，从RabbitMQ队列中删除）。
   * _使用相关ID_：Correlation ID是RabbitMQ应用程序功能。在将消息注入RabbitMQ时，必须在消息属性中设置它。源使用相关标识对从检查点还原时已重新处理的任何消息进行重复数据删除
   * _非并行源_：源必须是非并行的（并行度设置为1）才能实现精确一次。这种限制主要是由于RabbitMQ将消息从单个队列分派给多个消费者的方法
2. **至少一次\(At-least-once\)**：当启用了检查点，但未使用相关性ID或源是并行时，源仅提供至少一次保证。
3. **不保证\(No guarantee\)**：如果未启用检查点，则源不具有任何强有力的传递保证。在此设置下，一旦源接收并处理消息，消息将自动被确认，而不是与Flink的检查点协作

下面是一个完全一次的RabbitMQ源代码的代码示例。内部注释解释了可以忽略配置的哪些部分以获得更轻松的保证。

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// checkpointing is required for exactly-once or at-least-once guarantees
env.enableCheckpointing(...);

final RMQConnectionConfig connectionConfig = new RMQConnectionConfig.Builder()
    .setHost("localhost")
    .setPort(5000)
    ...
    .build();
    
final DataStream<String> stream = env
    .addSource(new RMQSource<String>(
        connectionConfig,            // config for the RabbitMQ connection
        "queueName",                 // name of the RabbitMQ queue to consume
        true,                        // use correlation ids; can be false if only at-least-once is required
        new SimpleStringSchema()))   // deserialization schema to turn messages into Java objects
    .setParallelism(1);              // non-parallel source is only required for exactly-once
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
// checkpointing is required for exactly-once or at-least-once guarantees
env.enableCheckpointing(...)

val connectionConfig = new RMQConnectionConfig.Builder()
    .setHost("localhost")
    .setPort(5000)
    ...
    .build
    
val stream = env
    .addSource(new RMQSource[String](
        connectionConfig,            // config for the RabbitMQ connection
        "queueName",                 // name of the RabbitMQ queue to consume
        true,                        // use correlation ids; can be false if only at-least-once is required
        new SimpleStringSchema))     // deserialization schema to turn messages into Java objects
    .setParallelism(1)               // non-parallel source is only required for exactly-once
```
{% endtab %}
{% endtabs %}

### **RabbitMQ Sink**

此连接器提供了一个`RMQSink`用于将消息发送到RabbitMQ队列的类。下面是设置RabbitMQ接收器的代码示例。

{% tabs %}
{% tab title="Java" %}
```java
final DataStream<String> stream = ...

final RMQConnectionConfig connectionConfig = new RMQConnectionConfig.Builder()
    .setHost("localhost")
    .setPort(5000)
    ...
    .build();
    
stream.addSink(new RMQSink<String>(
    connectionConfig,            // config for the RabbitMQ connection
    "queueName",                 // name of the RabbitMQ queue to send messages to
    new SimpleStringSchema()));  // serialization schema to turn Java objects to messages
```
{% endtab %}

{% tab title="Scala" %}
```scala
val stream: DataStream[String] = ...

val connectionConfig = new RMQConnectionConfig.Builder()
    .setHost("localhost")
    .setPort(5000)
    ...
    .build
    
stream.addSink(new RMQSink[String](
    connectionConfig,         // config for the RabbitMQ connection
    "queueName",              // name of the RabbitMQ queue to send messages to
    new SimpleStringSchema))  // serialization schema to turn Java objects to messages
```
{% endtab %}
{% endtabs %}

有关RabbitMQ的更多信息，请[点击此处](http://www.rabbitmq.com/)。

