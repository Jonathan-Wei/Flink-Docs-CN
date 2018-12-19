# 本地配置

只需几个简单的步骤即可启动并运行Flink示例程序

## 设置：下载并启动Flink

Flink可在**Linux，Mac OS X和Windows上运行**。为了能够运行Flink，唯一的要求是安装一个有效的**Java 8.x.** Windows用户，请查看[Windows](https://ci.apache.org/projects/flink/flink-docs-master/tutorials/flink_on_windows.html)上的[Flink](https://ci.apache.org/projects/flink/flink-docs-master/tutorials/flink_on_windows.html)指南，该指南介绍了如何在Windows上运行Flink以进行本地设置

您可以通过发出以下命令来检查Java的正确安装：

```bash
java -version
```

如果你有Java 8，输出将如下所示：

```bash
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

### 下载并编译

从github源码库clone flink源代码：

```bash
$ git clone https://github.com/apache/flink.git
$ cd flink
$ mvn clean package -DskipTests # this will take up to 10 minutes
$ cd build-target               # this is where Flink is installed to
```

### 启动本地集群

```bash
$ ./bin/start-cluster.sh  # Start Flink
```

检查程序调度Web前端 [http://localhost:8081](http://localhost:8081/)，并确保一切都正常运行。Web前端应报告单个可用的TaskManager实例。

![](../../.gitbook/assets/jobmanager-1.png)

还可以通过检查`logs`目录中的日志文件来验证系统是否正在运行：

```bash
$ tail log/flink-*-standalonesession-*.log
INFO ... - Rest endpoint listening at localhost:8081
INFO ... - http://localhost:8081 was granted leadership ...
INFO ... - Web frontend listening at http://localhost:8081.
INFO ... - Starting RPC endpoint for StandaloneResourceManager at akka://flink/user/resourcemanager .
INFO ... - Starting RPC endpoint for StandaloneDispatcher at akka://flink/user/dispatcher .
INFO ... - ResourceManager akka.tcp://flink@localhost:6123/user/resourcemanager was granted leadership ...
INFO ... - Starting the SlotManager.
INFO ... - Dispatcher akka.tcp://flink@localhost:6123/user/dispatcher was granted leadership ...
INFO ... - Recovering all persisted jobs.
INFO ... - Registering TaskManager ... under ... at the SlotManager.
```

## 阅读代码

您可以在GitHub上找到[Scala](https://github.com/apache/flink/blob/master/flink-examples/flink-examples-streaming/src/main/scala/org/apache/flink/streaming/scala/examples/socket/SocketWindowWordCount.scala)和[Java](https://github.com/apache/flink/blob/master/flink-examples/flink-examples-streaming/src/main/java/org/apache/flink/streaming/examples/socket/SocketWindowWordCount.java)的SocketWindowWordCount示例的完整源代码。

{% tabs %}
{% tab title="Scala" %}
```scala
object SocketWindowWordCount {

    def main(args: Array[String]) : Unit = {

        // the port to connect to
        val port: Int = try {
            ParameterTool.fromArgs(args).getInt("port")
        } catch {
            case e: Exception => {
                System.err.println("No port specified. Please run 'SocketWindowWordCount --port <port>'")
                return
            }
        }

        // get the execution environment
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

        // get input data by connecting to the socket
        val text = env.socketTextStream("localhost", port, '\n')

        // parse the data, group it, window it, and aggregate the counts
        val windowCounts = text
            .flatMap { w => w.split("\\s") }
            .map { w => WordWithCount(w, 1) }
            .keyBy("word")
            .timeWindow(Time.seconds(5), Time.seconds(1))
            .sum("count")

        // print the results with a single thread, rather than in parallel
        windowCounts.print().setParallelism(1)

        env.execute("Socket Window WordCount")
    }

    // Data type for words with count
    case class WordWithCount(word: String, count: Long)
}
```
{% endtab %}

{% tab title="Java" %}
```java
public class SocketWindowWordCount {

    public static void main(String[] args) throws Exception {

        // the port to connect to
        final int port;
        try {
            final ParameterTool params = ParameterTool.fromArgs(args);
            port = params.getInt("port");
        } catch (Exception e) {
            System.err.println("No port specified. Please run 'SocketWindowWordCount --port <port>'");
            return;
        }

        // get the execution environment
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // get input data by connecting to the socket
        DataStream<String> text = env.socketTextStream("localhost", port, "\n");

        // parse the data, group it, window it, and aggregate the counts
        DataStream<WordWithCount> windowCounts = text
            .flatMap(new FlatMapFunction<String, WordWithCount>() {
                @Override
                public void flatMap(String value, Collector<WordWithCount> out) {
                    for (String word : value.split("\\s")) {
                        out.collect(new WordWithCount(word, 1L));
                    }
                }
            })
            .keyBy("word")
            .timeWindow(Time.seconds(5), Time.seconds(1))
            .reduce(new ReduceFunction<WordWithCount>() {
                @Override
                public WordWithCount reduce(WordWithCount a, WordWithCount b) {
                    return new WordWithCount(a.word, a.count + b.count);
                }
            });

        // print the results with a single thread, rather than in parallel
        windowCounts.print().setParallelism(1);

        env.execute("Socket Window WordCount");
    }

    // Data type for words with count
    public static class WordWithCount {

        public String word;
        public long count;

        public WordWithCount() {}

        public WordWithCount(String word, long count) {
            this.word = word;
            this.count = count;
        }

        @Override
        public String toString() {
            return word + " : " + count;
        }
    }
}

```
{% endtab %}
{% endtabs %}

## 运行示例

* 首先，我们使用**netcat**来启动本地服务器

```bash
$ nc -l 9000
```

* 提交Flink计划：

```bash
$ ./bin/flink run examples/streaming/SocketWindowWordCount.jar --port 9000
Starting execution of program
```

程序连接到Socket并等待输入。您可以检查Web界面以验证作业是否按预期运行：

![](../../.gitbook/assets/jobmanager-2.png)

![](../../.gitbook/assets/jobmanager-3.png)

* 单词每5秒统计一次（处理时间，翻滚窗口）并打印到`stdout`。监视TaskManager的输出文件并且在`nc`里有文本信息输出（输入在点击后逐行发送到Flink）：

```bash
$ nc -l 9000
lorem ipsum
ipsum ipsum ipsum
bye
```

`.out`文件将在每个时间窗口结束时，打印word的统计信息，例如：

```bash
$ tail -f log/flink-*-taskexecutor-*.out
lorem : 1
bye : 1
ipsum : 4
```

停止Flink可以运行如下脚本：

```bash
$ ./bin/stop-cluster.sh
```

