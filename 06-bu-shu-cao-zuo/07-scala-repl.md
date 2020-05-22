# Scala REPL

## Scala REPL

Flink附带了一个集成的交互式Scala Shell。它可以在本地设置和群集设置中使用。

要将shell与集成的Flink集群一起使用，只需执行：

```text
bin/start-scala-shell.sh local
```

脚本存放在Flink根目录的bin目录下。要在群集上运行Shell，请参阅下面的“Setup”部分。

## 用法

Shell支持DataSet，DataStream，Table API和SQL。启动后会自动预绑定四个不同的环境。使用“ benv”和“ senv”分别访问Batch和Streaming ExecutionEnvironment。使用“ btenv”和“ stenv”分别访问BatchTableEnvironment和StreamTableEnvironment。

### DataSet API

以下示例将在Scala shell中执行wordcount程序：

```scala
Scala-Flink> val text = benv.fromElements(
  "To be, or not to be,--that is the question:--",
  "Whether 'tis nobler in the mind to suffer",
  "The slings and arrows of outrageous fortune",
  "Or to take arms against a sea of troubles,")
Scala-Flink> val counts = text
    .flatMap { _.toLowerCase.split("\\W+") }
    .map { (_, 1) }.groupBy(0).sum(1)
Scala-Flink> counts.print()
```

print\(\)命令将自动将指定的任务发送到JobManager执行，并在终端中显示计算结果。 可以将结果写入文件。但是，在这种情况下，您需要调用execute，以运行您的程序：

```scala
Scala-Flink> benv.execute("MyProgram")
```

### DataStream API

与上面的批处理程序类似，我们可以通过DataStream API执行流程序：

```scala
Scala-Flink> val textStreaming = senv.fromElements(
  "To be, or not to be,--that is the question:--",
  "Whether 'tis nobler in the mind to suffer",
  "The slings and arrows of outrageous fortune",
  "Or to take arms against a sea of troubles,")
Scala-Flink> val countsStreaming = textStreaming
    .flatMap { _.toLowerCase.split("\\W+") }
    .map { (_, 1) }.keyBy(0).sum(1)
Scala-Flink> countsStreaming.print()
Scala-Flink> senv.execute("Streaming Wordcount")
```

请注意，在Streaming情况下，打印操作不会直接触发执行。 Flink Shell附带命令历史记录和自动完成功能。

### Table API

下面的示例是使用Table API的单词计数程序：

{% tabs %}
{% tab title="流" %}
```scala
Scala-Flink> import org.apache.flink.table.functions.TableFunction
Scala-Flink> val textSource = stenv.fromDataStream(
  senv.fromElements(
    "To be, or not to be,--that is the question:--",
    "Whether 'tis nobler in the mind to suffer",
    "The slings and arrows of outrageous fortune",
    "Or to take arms against a sea of troubles,"),
  'text)
Scala-Flink> class $Split extends TableFunction[String] {
    def eval(s: String): Unit = {
      s.toLowerCase.split("\\W+").foreach(collect)
    }
  }
Scala-Flink> val split = new $Split
Scala-Flink> textSource.join(split('text) as 'word).
    groupBy('word).select('word, 'word.count as 'count).
    toRetractStream[(String, Long)].print
Scala-Flink> senv.execute("Table Wordcount")
```
{% endtab %}

{% tab title="批量" %}
```scala
Scala-Flink> import org.apache.flink.table.functions.TableFunction
Scala-Flink> val textSource = btenv.fromDataSet(
  benv.fromElements(
    "To be, or not to be,--that is the question:--",
    "Whether 'tis nobler in the mind to suffer",
    "The slings and arrows of outrageous fortune",
    "Or to take arms against a sea of troubles,"), 
  'text)
Scala-Flink> class $Split extends TableFunction[String] {
    def eval(s: String): Unit = {
      s.toLowerCase.split("\\W+").foreach(collect)
    }
  }
Scala-Flink> val split = new $Split
Scala-Flink> textSource.join(split('text) as 'word).
    groupBy('word).select('word, 'word.count as 'count).
    toDataSet[(String, Long)].print
```
{% endtab %}
{% endtabs %}

请注意，使用$作为TableFunction类名的前缀是scala错误生成内部类名的问题的解决方法。

### SQL

以下示例是用SQL编写的单词计数程序：

{% tabs %}
{% tab title="流" %}
```scala
Scala-Flink> import org.apache.flink.table.functions.TableFunction
Scala-Flink> val textSource = stenv.fromDataStream(
  senv.fromElements(
    "To be, or not to be,--that is the question:--",
    "Whether 'tis nobler in the mind to suffer",
    "The slings and arrows of outrageous fortune",
    "Or to take arms against a sea of troubles,"), 
  'text)
Scala-Flink> stenv.createTemporaryView("text_source", textSource)
Scala-Flink> class $Split extends TableFunction[String] {
    def eval(s: String): Unit = {
      s.toLowerCase.split("\\W+").foreach(collect)
    }
  }
Scala-Flink> stenv.registerFunction("split", new $Split)
Scala-Flink> val result = stenv.sqlQuery("""SELECT T.word, count(T.word) AS `count` 
    FROM text_source 
    JOIN LATERAL table(split(text)) AS T(word) 
    ON TRUE 
    GROUP BY T.word""")
Scala-Flink> result.toRetractStream[(String, Long)].print
Scala-Flink> senv.execute("SQL Wordcount")
```
{% endtab %}

{% tab title="批量" %}
```scala
Scala-Flink> import org.apache.flink.table.functions.TableFunction
Scala-Flink> val textSource = btenv.fromDataSet(
  benv.fromElements(
    "To be, or not to be,--that is the question:--",
    "Whether 'tis nobler in the mind to suffer",
    "The slings and arrows of outrageous fortune",
    "Or to take arms against a sea of troubles,"), 
  'text)
Scala-Flink> btenv.createTemporaryView("text_source", textSource)
Scala-Flink> class $Split extends TableFunction[String] {
    def eval(s: String): Unit = {
      s.toLowerCase.split("\\W+").foreach(collect)
    }
  }
Scala-Flink> btenv.registerFunction("split", new $Split)
Scala-Flink> val result = btenv.sqlQuery("""SELECT T.word, count(T.word) AS `count` 
    FROM text_source 
    JOIN LATERAL table(split(text)) AS T(word) 
    ON TRUE 
    GROUP BY T.word""")
Scala-Flink> result.toDataSet[(String, Long)].print
```
{% endtab %}
{% endtabs %}

## 添加外部依赖项

可以将外部类路径添加到Scala-shell。当调用execute时，它们将与shell程序一起自动发送到Jobmanager。 使用参数-a 或--addclasspath 加载其他类。

```text
bin/start-scala-shell.sh [local | remote <host> <port> | yarn] --addclasspath <path/to/jar.jar>
```

## Setup

要了解Scala Shell提供的选项，请使用

```text
bin/start-scala-shell.sh --help
```

### Local\(本地\)

要将shell与集成的Flink集群一起使用，只需执行：

```text
bin/start-scala-shell.sh local
```

### Remote\(远程\)

要将其与正在运行的集群一起使用，请启动scala shell的时候使用关键字`remote`,并为JobManager提供主机和端口：

```text
bin/start-scala-shell.sh remote <hostname> <portnumber>
```

### Yarn Scala Shell cluster\(集群\)

shell可以将Flink集群部署到Yarn上，Yarn是Shell专用的。通过参数-n 可以控制Yarn容器的数量。shell在Yarn上部署一个新的Flink集群并连接集群。您还可以为Yarn集群指定一些选项，如JobManager内存、Yarn应用程序名称等。

例如，要为Scala Shell启动一个纱线集群，使用以下两个任务管理器:

```text
 bin/start-scala-shell.sh yarn -n 2
```

对于所有其他选项，请参阅底部的完整参考。

### Yarn Session\(会话\)

如果您之前使用Flink Yarn会话部署了Flink集群，则Scala shell可以使用以下命令与其连接：

```text
 bin/start-scala-shell.sh yarn
```

## 完整参考

```text
Flink Scala Shell
Usage: start-scala-shell.sh [local|remote|yarn] [options] <args>...

Command: local [options]
使用本地Flink集群启动Flink scala shell
  -a <path/to/jar> | --addclasspath <path/to/jar>
        指定在Flink中使用的其他jar
Command: remote [options] <host> <port>
启动连接到远程集群的scala shell
  <host>
        远程主机名作为字符串
  <port>
        远程端口为整数

  -a <path/to/jar> | --addclasspath <path/to/jar>
        指定在Flink中使用的其他jar
Command: yarn [options]
启动连接到Yarn集群的Flink scala shell
  -n arg | --container arg
        分配Yarn容器数量(=任务管理器数量)
  -jm arg | --jobManagerMemory arg
        带有可选单元的JobManager容器的内存(默认:MB)
  -nm <value> | --name <value>
        为Yarn上的应用程序设置自定义名称
  -qu <arg> | --queue <arg>
        指定Yarn队列
  -s <arg> | --slots <arg>
        每个任务管理器的槽数
  -tm <arg> | --taskManagerMemory <arg>
        每个带有可选单元的任务管理器容器的内存(默认:MB)
  -a <path/to/jar> | --addclasspath <path/to/jar>
        指定在Flink中使用的其他jar
  --configDir <value>
        配置目录。
  -h | --help
        打印此用法文本
```

