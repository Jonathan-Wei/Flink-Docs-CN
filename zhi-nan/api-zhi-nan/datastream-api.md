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

```text
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

```text
package wikiedits;

public class WikipediaAnalysis {

    public static void main(String[] args) throws Exception {

    }
}
```

这个只是基础代码，我们会在其中填充我们的代码。请注意，我不会在此处提供import语句，因为IDE可以自动添加它们。在本节结束时，如果您只想跳过并在编辑器中输入，我将使用import语句显示完整的代码。

Flink程序的第一步是创建一个`StreamExecutionEnvironment` （或者`ExecutionEnvironment`如果您正在编写批处理作业）。这可用于设置执行参数并创建从外部系统读取的源。所以让我们继续把它添加到main方法：  


```text
StreamExecutionEnvironment see = StreamExecutionEnvironment.getExecutionEnvironment();
```

接下来，我们将创建一个从Wikipedia IRC日志中读取的源：

```text
DataStream<WikipediaEditEvent> edits = see.addSource(new WikipediaEditsSource());
```



