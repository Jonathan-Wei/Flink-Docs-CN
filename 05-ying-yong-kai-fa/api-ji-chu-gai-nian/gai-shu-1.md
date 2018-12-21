# 概述

## DataSet 和 DataStream

## Flink程序剖析

Flink程序看起来像转换数据集合的常规程序。每个程序由相同的基本部分组成:

1. 获得一个execution environment
2. 加载/创建初始数据
3. 指定此数据上的转换
4. 指定放置计算结果的位置
5. 触发程序执行

{% tabs %}
{% tab title="Java" %}
我们现在将概述每个步骤，请参阅相应部分以获取更多详细信息。请注意，Java DataSet API的所有核心类都可以在[org.apache.flink.api.java](https://github.com/apache/flink/blob/master//flink-java/src/main/java/org/apache/flink/api/java)包中找到， 而Java DataStream API的类可以在[org.apache.flink.streaming.api中](https://github.com/apache/flink/blob/master//flink-streaming-java/src/main/java/org/apache/flink/streaming/api)找到 。

`StreamExecutionEnvironment`是所有Flink程序的基础。您可以使用以下静态方法获取一个`StreamExecutionEnvironment`：

```java
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(String host, int port, String... jarFiles)
```
{% endtab %}

{% tab title="Scala" %}
我们现在将概述每个步骤，请参阅相应部分以获取更多详细信息。请注意，Scala DataSet API的所有核心类都可以在[org.apache.flink.api.scala](https://github.com/apache/flink/blob/master//flink-scala/src/main/scala/org/apache/flink/api/scala)包中找到， 而Scala DataStream API的类可以在[org.apache.flink.streaming.api.scala中](https://github.com/apache/flink/blob/master//flink-streaming-scala/src/main/scala/org/apache/flink/streaming/api/scala)找到 。

  
`StreamExecutionEnvironment`是所有Flink程序的基础。您可以使用以下静态方法获取一个`StreamExecutionEnvironment`：

```scala
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(host: String, port: Int, jarFiles: String*)
```
{% endtab %}
{% endtabs %}



