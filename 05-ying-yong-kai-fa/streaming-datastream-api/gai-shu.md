# 概述

## 示例程序

以下程序是流窗口字数统计应用程序的完整工作示例，它在5秒窗口中对来自Web Socket的单词进行计数。可以复制并粘贴代码以在本地运行它。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class WindowWordCount {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Tuple2<String, Integer>> dataStream = env
                .socketTextStream("localhost", 9999)
                .flatMap(new Splitter())
                .keyBy(0)
                .timeWindow(Time.seconds(5))
                .sum(1);

        dataStream.print();

        env.execute("Window WordCount");
    }

    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }

}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time

object WindowWordCount {
  def main(args: Array[String]) {

    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val text = env.socketTextStream("localhost", 9999)

    val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
      .map { (_, 1) }
      .keyBy(0)
      .timeWindow(Time.seconds(5))
      .sum(1)

    counts.print()

    env.execute("Window Stream WordCount")
  }
}
```
{% endtab %}
{% endtabs %}

要运行示例程序，首先从终端使用netcat启动输入流：

```text
nc -lk 9999
```



## 数据源



## DataStream转换

有关可用流转换的概述，请参阅[运算符](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/index.html)。

## Data Sinks

## Iterations

## 执行参数

### 容错

[State＆Checkpointing](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/checkpointing.html)描述了如何启用和配置Flink的检查点机制。

### 控制延迟

## Debugging

### 本地执行环境

### Collection数据源

### 迭代Data Sink

