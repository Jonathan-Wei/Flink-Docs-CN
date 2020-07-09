# NiFi

此连接器提供可以读取和写入[Apache NiFi](https://nifi.apache.org/)的Source和Sink 。要使用此连接器，请将以下依赖项添加到项目中：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-nifi_2.11</artifactId>
  <version>1.7.0</version>
</dependency>
```

**安装Apache NiFi**

有关设置Apache NiFi群集的说明，请 [访问此处](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#how-to-install-and-start-nifi)。

**Apache NiFi Source**

连接器提供了一个Source，用于从Apache NiFi到Apache Flink读取数据。

该类`NiFiSource(…)`提供了2个用于从NiFi读取数据的构造函数。

* `NiFiSource(SiteToSiteConfig config)`- 构造`NiFiSource(…)`给定客户端的SiteToSiteConfig，默认等待时间为1000毫秒。
* `NiFiSource(SiteToSiteConfig config, long waitTimeMs)`- 构造`NiFiSource(…)`给定客户端的SiteToSiteConfig和指定的等待时间（以毫秒为单位）。

例：

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment streamExecEnv = StreamExecutionEnvironment.getExecutionEnvironment();

SiteToSiteClientConfig clientConfig = new SiteToSiteClient.Builder()
        .url("http://localhost:8080/nifi")
        .portName("Data for Flink")
        .requestBatchCount(5)
        .buildConfig();

SourceFunction<NiFiDataPacket> nifiSource = new NiFiSource(clientConfig);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val streamExecEnv = StreamExecutionEnvironment.getExecutionEnvironment()

val clientConfig: SiteToSiteClientConfig = new SiteToSiteClient.Builder()
       .url("http://localhost:8080/nifi")
       .portName("Data for Flink")
       .requestBatchCount(5)
       .buildConfig()

val nifiSource = new NiFiSource(clientConfig)       
```
{% endtab %}
{% endtabs %}

这里的数据是从名为“Data for Flink”的Apache NiFi输出端口读取的，这是Apache NiFi端到端协议配置的一部分。

**Apache NiFi Sink**

连接器提供了一个Sink，用于将数据从Apache Flink写入Apache NiFi。

该类`NiFiSink(…)`提供了一个用于实例化a的构造函数`NiFiSink`。

* `NiFiSink(SiteToSiteClientConfig, NiFiDataPacketBuilder<T>)`根据客户端的SiteToSiteConfig构造一个`NiFiSink(…)`和一个`NiFiDataPacketBuilder`**，**后者将数据从Flink转换为NiFiDataPacket，供NiFi接收

例：

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment streamExecEnv = StreamExecutionEnvironment.getExecutionEnvironment();

SiteToSiteClientConfig clientConfig = new SiteToSiteClient.Builder()
        .url("http://localhost:8080/nifi")
        .portName("Data from Flink")
        .requestBatchCount(5)
        .buildConfig();

SinkFunction<NiFiDataPacket> nifiSink = new NiFiSink<>(clientConfig, new NiFiDataPacketBuilder<T>() {...});

streamExecEnv.addSink(nifiSink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val streamExecEnv = StreamExecutionEnvironment.getExecutionEnvironment()

val clientConfig: SiteToSiteClientConfig = new SiteToSiteClient.Builder()
       .url("http://localhost:8080/nifi")
       .portName("Data from Flink")
       .requestBatchCount(5)
       .buildConfig()

val nifiSink: NiFiSink[NiFiDataPacket] = new NiFiSink[NiFiDataPacket](clientConfig, new NiFiDataPacketBuilder<T>() {...})

streamExecEnv.addSink(nifiSink)
```
{% endtab %}
{% endtabs %}

有关[Apache NiFi](https://nifi.apache.org/)端到端协议的更多信息，请[访问此处](https://nifi.apache.org/docs/nifi-docs/html/user-guide.html#site-to-site)

