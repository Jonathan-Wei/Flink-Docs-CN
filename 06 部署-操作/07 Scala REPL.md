# Scala REPL
Flink附带了一个集成的交互式Scala Shell。它可以在本地设置和群集设置中使用。

要将shell与集成的Flink集群一起使用，只需执行：
```
bin/start-scala-shell.sh local
```
脚本存放在Flink根目录的bin目录下。要在群集上运行Shell，请参阅下面的“Setup”部分。

# 用法
shell支持Batch和Streaming。启动后会自动预先绑定两个不同的ExecutionEnvironments。使用“benv”和“senv”分别访问Batch和Streaming环境。

## DataSet API
以下示例将在Scala shell中执行wordcount程序：
```
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
print()命令将自动将指定的任务发送到JobManager执行，并在终端中显示计算结果。
可以将结果写入文件。但是，在这种情况下，您需要调用execute，以运行您的程序：
```
Scala-Flink> benv.execute("MyProgram")
```

## DataStream API
与上面的批处理程序类似，我们可以通过DataStream API执行流程序：
```
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
请注意，在Streaming情况下，打印操作不会直接触发执行。
Flink Shell附带命令历史记录和自动完成功能。

# 添加外部依赖项
可以将外部类路径添加到Scala-shell。当调用execute时，它们将与shell程序一起自动发送到Jobmanager。
使用参数-a <path/to/jar.jar>或--addclasspath <path/to/jar.jar>加载其他类。
```
bin/start-scala-shell.sh [local | remote <host> <port> | yarn] --addclasspath <path/to/jar.jar>
```

# Setup
要了解Scala Shell提供的选项，请使用
```
bin/start-scala-shell.sh --help
```

## Local(本地)
要将shell与集成的Flink集群一起使用，只需执行：
```
bin/start-scala-shell.sh local
```
## Remote(远程)
要将其与正在运行的集群一起使用，请启动scala shell的时候使用关键字`remote`,并为JobManager提供主机和端口：
```
bin/start-scala-shell.sh remote <hostname> <portnumber>
```

## Yarn Scala Shell cluster(集群)
shell可以将Flink集群部署到Yarn上，Yarn是Shell专用的。通过参数-n <arg>可以控制Yarn容器的数量。shell在Yarn上部署一个新的Flink集群并连接集群。您还可以为Yarn集群指定一些选项，如JobManager内存、Yarn应用程序名称等。

例如，要为Scala Shell启动一个纱线集群，使用以下两个任务管理器:
```
 bin/start-scala-shell.sh yarn -n 2
```
对于所有其他选项，请参阅底部的完整参考。

## Yarn Session(会话)
如果您之前使用Flink Yarn会话部署了Flink集群，则Scala shell可以使用以下命令与其连接：
```
 bin/start-scala-shell.sh yarn
```

# 完整参考
```
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

