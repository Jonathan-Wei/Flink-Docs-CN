# Google Cloud PubSub

此连接器提供了可以从[Google Cloud PubSub](https://cloud.google.com/pubsub)读取和写入的Source和Sink 。要使用此连接器，请将以下依赖项添加到您的项目中：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-gcp-pubsub_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

{% hint style="danger" %}
**注意**：此连接器最近已添加到Flink。它尚未接受广泛的测试。
{% endhint %}

请注意，流连接器目前不是二进制发行版的一部分。有关如何将程序与用于集群执行的库打包的信息，请参见 [此处](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/projectsetup/dependencies.html)。

## 消费或产生PubSubMessages

 该连接器提供了用于从Google PubSub接收消息和向Google PubSub发送消息的连接器。Google PubSub有`at-least-once`保证，因此连接器也提供同样的保证。

### PubSub SourceFunction

 类`PubSubSource`具有用于创建PubSubsources的构建器：`PubSubSource.newBuilder(...)`

有几种可选的方法可以改变创建`PubSubSource`的方式，最低限度是提供一个谷歌项目。Pubsub订阅以及反序列化`PubSubMessages`的方法。

例如：

```java
StreamExecutionEnvironment streamExecEnv = StreamExecutionEnvironment.getExecutionEnvironment();

DeserializationSchema<SomeObject> deserializer = (...);
SourceFunction<SomeObject> pubsubSource = PubSubSource.newBuilder()
                                                      .withDeserializationSchema(deserializer)
                                                      .withProjectName("project")
                                                      .withSubscriptionName("subscription")
                                                      .build();

streamExecEnv.addSource(source);
```

 当前，SourceFunction从PubSub中[提取](https://cloud.google.com/pubsub/docs/pull)消息，不支持[端点](https://cloud.google.com/pubsub/docs/push)[推送](https://cloud.google.com/pubsub/docs/push)。

### PubSub Sink

类PubSubSink具有用于创建`PubSubSink`的构建器：`PubSubSink.newBuilder (…)`

这个构建器的工作方式与`PubSubSource`类似。

```java
DataStream<SomeObject> dataStream = (...);

SerializationSchema<SomeObject> serializationSchema = (...);
SinkFunction<SomeObject> pubsubSink = PubSubSink.newBuilder()
                                                .withSerializationSchema(serializationSchema)
                                                .withProjectName("project")
                                                .withSubscriptionName("subscription")
                                                .build()

dataStream.addSink(pubsubSink);
```

### Google凭证

谷歌使用 [凭据](https://cloud.google.com/docs/authentication/production)对应用程序进行身份验证和授权，以便它们可以使用谷歌云平台资源\(例如PubSub\)。

这两个构建器都允许您提供这些凭证，但是在默认情况下，连接器将查找一个环境变量: [GOOGLE\_APPLICATION\_CREDENTIALS](https://cloud.google.com/docs/authentication/production#obtaining_and_providing_service_account_credentials_manually)，该变量应该指向包含凭证的文件。

如果您想手动提供凭证，例如，如果您自己从外部系统读取凭证，您可以使用`PubSubSource.newBuilder(…). withcredentials(…)`。

### 整合测试

运行集成测试时，您可能不想直接连接到PubSub，而是使用Docker容器进行读写。（请参阅：[本地PubSub测试](https://cloud.google.com/pubsub/docs/emulator)）

以下示例显示了如何创建源以从模拟器读取消息并将其发送回：

```java
String hostAndPort = "localhost:1234";
DeserializationSchema<SomeObject> deserializationSchema = (...);
SourceFunction<SomeObject> pubsubSource = PubSubSource.newBuilder()
                                                      .withDeserializationSchema(deserializationSchema)
                                                      .withProjectName("my-fake-project")
                                                      .withSubscriptionName("subscription")
                                                      .withPubSubSubscriberFactory(new PubSubSubscriberFactoryForEmulator(hostAndPort, "my-fake-project", "subscription", 10, Duration.ofSeconds(15), 100))
                                                      .build();
SerializationSchema<SomeObject> serializationSchema = (...);
SinkFunction<SomeObject> pubsubSink = PubSubSink.newBuilder()
                                                .withSerializationSchema(serializationSchema)
                                                .withProjectName("my-fake-project")
                                                .withSubscriptionName("subscription")
                                                .withHostAndPortForEmulator(hostAndPort)
                                                .build()

StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.addSource(pubsubSource)
   .addSink(pubsubSink);
```

### Atleast一次保证

#### **SourceFunction**

消息可能多次发送的原因有很多，例如谷歌PubSub端的失败场景。

另一个原因是确认截止日期已过。这是接收消息和确认消息之间的时间。PubSubSource将只在成功的检查点上确认消息，以保证至少一次。这意味着，如果成功的检查点之间的时间大于订阅消息的确认截止日期，则很可能会被多次处理。

由于这个原因，建议检查点间隔时间比确认截止时间短得多。

有关如何增加订阅的确认截止日期的详细信息，请参见PubSub。

注意:`PubSubMessagesProcessedNotAcked`指标显示有多少消息在等待下一个检查点之前被确认。

#### **SinkFunction**

由于性能原因，**SinkFunction**会对短时间内要发送到PubSub的消息进行缓冲。在每个检查点之前，这个缓冲区将被刷新，除非消息已经传递到PubSub，否则检查点将不会成功。





