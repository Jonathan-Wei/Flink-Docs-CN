# DataStream API教程

在本指南中，我们将从头开始，从建立Flink项目到在Flink集群上运行流分析程序。

维基百科提供了一个IRC通道，所有对维基的编辑都被记录在这个通道中。我们将在Flink中读取这个通道，并计算每个用户在给定时间窗口内编辑的字节数。使用Flink在几分钟内实现这一点非常简单，但是它将为您开始构建更复杂的分析程序打下良好的基础。

## 配置Maven工程

我们将使用Flink Maven原型来创建项目结构。请参阅Java API Quickstart了解更多细节。出于我们的目的，要运行的命令如下:

```text
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-quickstart-java \
    -DarchetypeVersion=1.7.0 \
    -DgroupId=wiki-edits \
    -DartifactId=wiki-edits \
    -Dversion=0.1 \
    -Dpackage=wikiedits \
    -DinteractiveMode=false
```

您可以编辑`groupId`，`artifactId`而`package`。使用上面的参数，Maven将创建一个如下所示的项目结构：

```text
$ tree wiki-edits
wiki-edits/
├── pom.xml
└── src
    └── main
        ├── java
        │   └── wikiedits
        │       ├── BatchJob.java
        │       ├── SocketTextStreamWordCount.java
        │       ├── StreamingJob.java
        │       └── WordCount.java
        └── resources
            └── log4j.properties
```

`pom.xml`文件已经在根目录中添加了Flink依赖项，并且Flink程序`src/main/java`目录下有几个示例。我们可以删除示例程序，因为我们将从头开始：

```text
$ rm wiki-edits/src/main/java/wikiedits/*.java
```

作为最后一步，我们需要将Flink Wikipedia连接器添加为依赖关系，以便我们可以在我们的程序中使用它。编辑它的`dependencies`部分`pom.xml`，具体代码如下：

```markup
<dependencies>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-clients_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-wikiedits_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
</dependencies>
```

注意`flink-connector-wikiedits_2.11`添加的依赖项。（此示例和Wikipedia连接器的灵感来自Apache Samza 的_Hello Samza_示例。）

## 编写Flink程序

可以开始编码了。启动IDE软件并导入Maven项目或打开文本编辑器并创建文件`src/main/java/wikiedits/WikipediaAnalysis.java`：

```java
package wikiedits;

public class WikipediaAnalysis {

    public static void main(String[] args) throws Exception {

    }
}
```

这个只是基础代码，我们会在其中填充我们的代码。请注意，我不会在此处提供import语句，因为IDE可以自动添加它们。在本节结束时，如果您只想跳过并在编辑器中输入，我将使用import语句显示完整的代码。

Flink程序的第一步是创建一个`StreamExecutionEnvironment` （或者`ExecutionEnvironment`如果您正在编写批处理作业）。这可用于设置执行参数并创建从外部系统读取的源。所以让我们继续把它添加到main方法：

```java
StreamExecutionEnvironment see = StreamExecutionEnvironment.getExecutionEnvironment();
```

接下来，我们将创建一个从Wikipedia IRC日志中读取的源：

```java
DataStream<WikipediaEditEvent> edits = see.addSource(new WikipediaEditsSource());
```

这创建了一个我们可以进一步处理`DataStream`的`WikipediaEditEvent`元素。出于本示例的目的，我们感兴趣的是确定每个用户在特定时间窗口中添加或删除的字节数，比如说五秒。为此，我们首先必须指定我们要在用户名上键入流，也就是说此流上的操作应考虑用户名。在我们的例子中，窗口中编辑的字节的总和应该是每个唯一的用户。对于键入流，我们必须提供一个`KeySelector`，如下所示：

```java
KeyedStream<WikipediaEditEvent, String> keyedEdits = edits
    .keyBy(new KeySelector<WikipediaEditEvent, String>() {
        @Override
        public String getKey(WikipediaEditEvent event) {
            return event.getUser();
        }
    });

```

这给了我们一个维基百科事件流，它有一个字符串键，用户名。现在，我们可以指定要将窗口强加于此流，并基于这些窗口中的元素计算结果。窗口指定要在其上执行计算的流的切片。在无限元素流上计算聚合时，需要使用Windows。在我们的示例中，我们将说我们希望每5秒聚合一次编辑字节的总和：

```java
DataStream<Tuple2<String, Long>> result = keyedEdits
    .timeWindow(Time.seconds(5))
    .fold(new Tuple2<>("", 0L), new FoldFunction<WikipediaEditEvent, Tuple2<String, Long>>() {
        @Override
        public Tuple2<String, Long> fold(Tuple2<String, Long> acc, WikipediaEditEvent event) {
            acc.f0 = event.getUser();
            acc.f1 += event.getByteDiff();
            return acc;
        }
    });
```

第一个调用`.timeWindow()`指定要有5秒的滚动（不重叠）窗口。第二个调用为每个唯一键在每个窗口切片上指定一个折叠转换\(`flod`\)。在我们的例子中，我们从初始值（“”，0L）开始，并为用户添加该时间窗口中每次编辑的字节差。结果流现在包含每个用户的`Tuple2<String, Long>`，每5秒发出一次。

剩下要做的就是将流打印到控制台并开始执行：

```text
result.print();

see.execute();
```

最后一次调用是启动实际Flink作业所必需的。所有操作（例如创建源，转换和接收器）仅构建内部操作的图形。只有在`execute()`被调用时 才会在集群上抛出或在本地计算机上执行此操作图。

到目前为止完整的代码是这样的：

```java
package wikiedits;

import org.apache.flink.api.common.functions.FoldFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.connectors.wikiedits.WikipediaEditEvent;
import org.apache.flink.streaming.connectors.wikiedits.WikipediaEditsSource;

public class WikipediaAnalysis {

  public static void main(String[] args) throws Exception {

    StreamExecutionEnvironment see = StreamExecutionEnvironment.getExecutionEnvironment();

    DataStream<WikipediaEditEvent> edits = see.addSource(new WikipediaEditsSource());

    KeyedStream<WikipediaEditEvent, String> keyedEdits = edits
      .keyBy(new KeySelector<WikipediaEditEvent, String>() {
        @Override
        public String getKey(WikipediaEditEvent event) {
          return event.getUser();
        }
      });

    DataStream<Tuple2<String, Long>> result = keyedEdits
      .timeWindow(Time.seconds(5))
      .fold(new Tuple2<>("", 0L), new FoldFunction<WikipediaEditEvent, Tuple2<String, Long>>() {
        @Override
        public Tuple2<String, Long> fold(Tuple2<String, Long> acc, WikipediaEditEvent event) {
          acc.f0 = event.getUser();
          acc.f1 += event.getByteDiff();
          return acc;
        }
      });

    result.print();

    see.execute();
  }
}
```

您可以使用Maven在IDE或命令行上运行此示例：

```bash
$ mvn clean package
$ mvn exec:java -Dexec.mainClass=wikiedits.WikipediaAnalysis
```

第一个命令构建我们的项目，第二个命令执行我们的主类。输出应该类似于：

```text
1> (Fenix down,114)
6> (AnomieBOT,155)
8> (BD2412bot,-3690)
7> (IgnorantArmies,49)
3> (Ckh3111,69)
5> (Slade360,0)
7> (Narutolovehinata5,2195)
6> (Vuyisa2001,79)
4> (Ms Sarah Welch,269)
4> (KasparBot,-245)
```

每行前面的数字表示输出生成的打印接收器的哪个并行实例。

这应该可以让你开始编写自己的Flink程序。要了解更多信息，您可以查看我们的[基本概念](https://ci.apache.org/projects/flink/flink-docs-master/dev/api_concepts.html)指南和 [DataStream API](https://ci.apache.org/projects/flink/flink-docs-master/dev/datastream_api.html)。如果您想了解如何在自己的机器上设置Flink群集并将结果写入[Kafka](http://kafka.apache.org/)，请坚持参加额外练习。

## 额外练习：在群集上运行并写入Kafka

请按照我们的[本地安装教程](https://ci.apache.org/projects/flink/flink-docs-master/tutorials/local_setup.html)在您的机器上启动Flink，并在继续操作之前参考[Kafka快速入门](https://kafka.apache.org/documentation.html#quickstart)以配置安装Kafka。

作为第一步，我们必须添加Flink Kafka连接器作为依赖关系，以便我们可以使用Kafka接收器。添加依赖项到`pom.xml`文件中：

```markup
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.8_2.11</artifactId>
    <version>${flink.version}</version>
</dependency>
```

接下来，我们需要修改我们的程序。我们将移除`print()`Sink，而是使用Kafka Sink。新代码如下所示：

```scala
result
    .map(new MapFunction<Tuple2<String,Long>, String>() {
        @Override
        public String map(Tuple2<String, Long> tuple) {
            return tuple.toString();
        }
    })
    .addSink(new FlinkKafkaProducer08<>("localhost:9092", "wiki-result", new SimpleStringSchema()));
```

还需要导入相关的类：

```scala
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer08;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.functions.MapFunction;
```

请注意，我们如何首先使`MapFunction`将`Tuple2<String, Long>`的流转换为`String`流。我们这样做是因为向Kafka写纯字符串更容易。然后创建一个Kafka Sink。这里可能需要根据设置调整主机名和端口。“wiki result”是程序运行之前要创建的kafka流的名称。使用maven构建项目，因为我们需要在集群上运行jar文件：

```bash
$ mvn clean package
```

生成的jar文件将位于`target`子文件夹中：`target/wiki-edits-0.1.jar`。我们稍后会用到。

准备启动Flink集群并运行写入Kafka的程序。转到安装Flink的位置并启动本地群集：

```bash
$ cd my/flink/directory
$ bin/start-cluster.sh
```

必须创建Kafka主题，以便我们的程序可以写入它：

```bash
$ cd my/kafka/directory
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic wiki-results
```

本地Flink集群上运行我们的jar文件：

```bash
$ cd my/flink/directory
$ bin/flink run -c wikiedits.WikipediaAnalysis path/to/wikiedits-0.1.jar
```

如果一切按计划进行，那么该命令的输出应该与此类似：

```bash
03/08/2016 15:09:27 Job execution switched to status RUNNING.
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to SCHEDULED
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to DEPLOYING
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to SCHEDULED
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to DEPLOYING
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to RUNNING
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to RUNNING
```

您可以看到各个操作符\(Operator\)如何开始运行。这里只有两个，因为出于性能原因，窗口之后的操作被折叠成一个操作。在Flink，我们称之为_链接\(chaining\)_。

您可以通过使用`kafka-console-consumer.sh`检查Kafka Topic来观察程序的输出：

```bash
bin/kafka-console-consumer.sh  --zookeeper localhost:2181 --topic wiki-result
```

还可以在[http:// localhost:8081](http://localhost:8081/)查看Flink仪表板。您将获得群集资源和正在运行的作业的概述：

![](../../../.gitbook/assets/jobmanager-overview.png)

如果您单击正在运行的作业，您将看到一个视图，您可以在其中检查各个操作，例如，查看处理的元素的数量:

![](../../../.gitbook/assets/jobmanager-job.png)

