# 概述

Flink中的DataSet程序是实现数据集转换的常规程序（例如，过滤，映射，连接，分组）。数据集最初是从某些来源创建的（例如，通过读取文件或从本地集合创建）。结果通过接收器返回，接收器可以例如将数据写入（分布式）文件或标准输出（例如命令行终端）。Flink程序可以在各种环境中运行，独立运行或嵌入其他程序中。执行可以在本地JVM中执行，也可以在许多计算机的集群上执行。

有关Flink API [基本概念](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html)的介绍，请参阅[基本概念](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html)。

为了创建自己的Flink DataSet程序，我们鼓励你从[Flink程序](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#anatomy-of-a-flink-program)的[解剖](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#anatomy-of-a-flink-program)开始， 逐步添加自己的 [转换](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/#dataset-transformations)。其余部分充当其他操作和高级功能的参考。

## 示例程序

以下程序是WordCount的完整工作示例。您可以复制并粘贴代码以在本地运行它。您只需要在项目中包含正确的Flink库（请参见[使用Flink链接](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking_with_flink.html)）并指定导入。然后你准备好了！

{% tabs %}
{% tab title="Java" %}
```java
public class WordCountExample {
    public static void main(String[] args) throws Exception {
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        DataSet<String> text = env.fromElements(
            "Who's there?",
            "I think I hear them. Stand, ho! Who's there?");

        DataSet<Tuple2<String, Integer>> wordCounts = text
            .flatMap(new LineSplitter())
            .groupBy(0)
            .sum(1);

        wordCounts.print();
    }

    public static class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String line, Collector<Tuple2<String, Integer>> out) {
            for (String word : line.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.scala._

object WordCount {
  def main(args: Array[String]) {

    val env = ExecutionEnvironment.getExecutionEnvironment
    val text = env.fromElements(
      "Who's there?",
      "I think I hear them. Stand, ho! Who's there?")

    val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
      .map { (_, 1) }
      .groupBy(0)
      .sum(1)

    counts.print()
  }
}
```
{% endtab %}
{% endtabs %}

## 数据集\(DataSet\)转换

数据转换将一个或多个DataSet转换为新的DataSet。程序可以将多个转换组合成复杂的程序集。

本节简要概述了可用的转换。该[转换文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html)用示例全整的完整描述了所有转换。

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Map</b>
      </td>
      <td style="text-align:left">
        <p>&#x83B7;&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;</p>
        <p>data<b>.</b>map(<b>new</b> MapFunction<b>&lt;</b>String, Integer<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b> Integer <b>map</b>(String value) <b>{</b>  <b>return</b> Integer<b>.</b>parseInt(value); <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>FlatMap</b>
      </td>
      <td style="text-align:left">
        <p>&#x83B7;&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x96F6;&#x4E2A;&#xFF0C;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;</p>
        <p>data<b>.</b>flatMap(<b>new</b> FlatMapFunction<b>&lt;</b>String, String<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b>  <b>void</b>  <b>flatMap</b>(String value, Collector<b>&lt;</b>String<b>&gt;</b> out) <b>{</b>
        </p>
        <p> <b>for</b> (String s <b>:</b> value<b>.</b>split(&quot; &quot;)) <b>{</b>
        </p>
        <p>out<b>.</b>collect(s);</p>
        <p> <b>}</b>
        </p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MapPartition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x5355;&#x4E2A;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x4E2D;&#x8F6C;&#x6362;&#x5E76;&#x884C;&#x5206;&#x533A;&#x3002;&#x8BE5;&#x51FD;&#x6570;&#x5C06;&#x5206;&#x533A;&#x4F5C;&#x4E3A;<b>Iterable</b>&#x6D41;&#x6765;&#x83B7;&#x53D6;&#xFF0C;&#x5E76;&#x4E14;&#x53EF;&#x4EE5;&#x751F;&#x6210;&#x4EFB;&#x610F;&#x6570;&#x91CF;&#x7684;&#x7ED3;&#x679C;&#x503C;&#x3002;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x4E2D;&#x7684;&#x5143;&#x7D20;&#x6570;&#x91CF;&#x53D6;&#x51B3;&#x4E8E;&#x5E76;&#x884C;&#x5EA6;&#x548C;&#x5148;&#x524D;&#x7684;&#x64CD;&#x4F5C;&#x3002;</p>
        <p>data<b>.</b>mapPartition(<b>new</b> MapPartitionFunction<b>&lt;</b>String,
          Long<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b>  <b>void</b>  <b>mapPartition</b>(Iterable<b>&lt;</b>String<b>&gt;</b> values,
          Collector<b>&lt;</b>Long<b>&gt;</b> out) <b>{</b>
        </p>
        <p> <b>long</b> c <b>=</b> 0;</p>
        <p> <b>for</b> (String s <b>:</b> values) <b>{</b>
        </p>
        <p>c<b>++</b>;</p>
        <p> <b>}</b>
        </p>
        <p>out<b>.</b>collect(c);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Filter</b>
      </td>
      <td style="text-align:left">
        <p>&#x8BA1;&#x7B97;&#x6BCF;&#x4E2A;&#x5143;&#x7D20;&#x7684;&#x5E03;&#x5C14;&#x51FD;&#x6570;&#xFF0C;&#x5E76;&#x4FDD;&#x7559;&#x51FD;&#x6570;&#x8FD4;&#x56DE;true&#x7684;&#x5143;&#x7D20;&#x3002;
          <br
          />&#x91CD;&#x8981;&#x4FE1;&#x606F;&#xFF1A;&#x7CFB;&#x7EDF;&#x5047;&#x5B9A;&#x8BE5;&#x51FD;&#x6570;&#x4E0D;&#x4F1A;&#x4FEE;&#x6539;&#x5E94;&#x7528;&#x8C13;&#x8BCD;&#x7684;&#x5143;&#x7D20;&#x3002;&#x8FDD;&#x53CD;&#x6B64;&#x5047;&#x8BBE;&#x53EF;&#x80FD;&#x4F1A;&#x5BFC;&#x81F4;&#x9519;&#x8BEF;&#x7684;&#x7ED3;&#x679C;&#x3002;</p>
        <p>data<b>.</b>filter(<b>new</b> FilterFunction<b>&lt;</b>Integer<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Integer value) <b>{</b>  <b>return</b> value <b>&gt;</b> 1000; <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Reduce</b>
      </td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x5C06;&#x4E24;&#x4E2A;&#x5143;&#x7D20;&#x91CD;&#x590D;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x5C06;&#x4E00;&#x7EC4;&#x5143;&#x7D20;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;Reduce&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>data<b>.</b>reduce(<b>new</b> ReduceFunction<b>&lt;</b>Integer<b>&gt;</b>  <b>{</b>
        </p>
        <p> <b>public</b> Integer <b>reduce</b>(Integer a, Integer b) <b>{</b>  <b>return</b> a <b>+</b> b; <b>}</b>
        </p>
        <p><b>}</b>);</p>
        <p>&#x5982;&#x679C;&#x5C06;reduce&#x5E94;&#x7528;&#x4E8E;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#xFF0C;&#x5219;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;&#x63D0;&#x4F9B;CombineHintto
          &#x6765;&#x6307;&#x5B9A;&#x8FD0;&#x884C;&#x65F6;&#x6267;&#x884C;reduce&#x7684;&#x7EC4;&#x5408;&#x9636;&#x6BB5;&#x7684;&#x65B9;&#x5F0F;
          setCombineHint&#x3002;&#x5728;&#x5927;&#x591A;&#x6570;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x57FA;&#x4E8E;&#x6563;&#x5217;&#x7684;&#x7B56;&#x7565;&#x5E94;&#x8BE5;&#x66F4;&#x5FEB;&#xFF0C;&#x7279;&#x522B;&#x662F;&#x5982;&#x679C;&#x4E0D;&#x540C;&#x952E;&#x7684;&#x6570;&#x91CF;&#x4E0E;&#x8F93;&#x5165;&#x5143;&#x7D20;&#x7684;&#x6570;&#x91CF;&#x76F8;&#x6BD4;&#x8F83;&#x5C0F;&#xFF08;&#x4F8B;&#x5982;1/10&#xFF09;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>ReduceGroup</b>
      </td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x7EC4;&#x5143;&#x7D20;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;<b>ReduceGroup</b>&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>data<b>.</b>reduceGroup(<b>new</b> GroupReduceFunction<b>&lt;</b>Integer,
          Integer<b>&gt;</b>  <b>{</b>
        </p>
        <p> <b>public</b>  <b>void</b>  <b>reduce</b>(Iterable<b>&lt;</b>Integer<b>&gt;</b> values,
          Collector<b>&lt;</b>Integer<b>&gt;</b> out) <b>{</b>
        </p>
        <p> <b>int</b> prefixSum <b>=</b> 0;</p>
        <p> <b>for</b> (Integer i <b>:</b> values) <b>{</b>
        </p>
        <p>prefixSum <b>+=</b> i;</p>
        <p>out<b>.</b>collect(prefixSum);</p>
        <p> <b>}</b>
        </p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregate</b>
      </td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x7EC4;&#x503C;&#x805A;&#x5408;&#x4E3A;&#x5355;&#x4E2A;&#x503C;&#x3002;&#x805A;&#x5408;&#x51FD;&#x6570;&#x53EF;&#x4EE5;&#x88AB;&#x8BA4;&#x4E3A;&#x662F;&#x5185;&#x7F6E;&#x7684;<b>reduce</b>&#x51FD;&#x6570;&#x3002;&#x805A;&#x5408;&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>Dataset<b>&lt;</b>Tuple3<b>&lt;</b>Integer, String, Double<b>&gt;&gt;</b> input <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, String, Double<b>&gt;&gt;</b> output <b>=</b> input<b>.</b>aggregate(SUM,
          0)<b>.</b>and(MIN, 2);</p>
        <p>&#x8FD8;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x7B80;&#x5199;&#x8BED;&#x6CD5;&#x8FDB;&#x884C;&#x6700;&#x5C0F;&#xFF0C;&#x6700;&#x5927;&#x548C;&#x603B;&#x548C;&#x805A;&#x5408;&#x3002;</p>
        <p>Dataset<b>&lt;</b>Tuple3<b>&lt;</b>Integer, String, Double<b>&gt;&gt;</b> input <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, String, Double<b>&gt;&gt;</b> output <b>=</b> input<b>.</b>sum(0)<b>.</b>andMin(2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct</b>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x636E;&#x96C6;&#x7684;&#x4E0D;&#x540C;&#x5143;&#x7D20;&#x3002;&#x5B83;&#x4ECE;&#x8F93;&#x5165;&#x6570;&#x636E;&#x96C6;&#x4E2D;&#x5220;&#x9664;&#x4E0E;&#x5143;&#x7D20;&#x7684;&#x6240;&#x6709;&#x5B57;&#x6BB5;&#x6216;&#x5B57;&#x6BB5;&#x5B50;&#x96C6;&#x76F8;&#x5173;&#x7684;&#x91CD;&#x590D;&#x6761;&#x76EE;&#x3002;</p>
        <p><code>data.distinct();</code>
        </p>
        <p>Distinct&#x662F;&#x4F7F;&#x7528;reduce&#x51FD;&#x6570;&#x5B9E;&#x73B0;&#x7684;&#x3002;&#x60A8;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;&#x5411;setCombineHint&#x63D0;&#x4F9B;&#x4E00;&#x4E2A;CombineHint&#x6765;&#x6307;&#x5B9A;&#x8FD0;&#x884C;&#x65F6;&#x6267;&#x884C;reduce&#x7684;combine&#x9636;&#x6BB5;&#x7684;&#x65B9;&#x5F0F;&#x3002;&#x5728;&#x5927;&#x591A;&#x6570;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x57FA;&#x4E8E;&#x54C8;&#x5E0C;&#x7684;&#x7B56;&#x7565;&#x5E94;&#x8BE5;&#x66F4;&#x5FEB;&#x4E00;&#x4E9B;&#xFF0C;&#x7279;&#x522B;&#x662F;&#x5F53;&#x4E0D;&#x540C;&#x952E;&#x7684;&#x6570;&#x91CF;&#x4E0E;&#x8F93;&#x5165;&#x5143;&#x7D20;&#x7684;&#x6570;&#x91CF;&#x76F8;&#x6BD4;&#x8F83;&#x5C0F;&#x65F6;(&#x4F8B;&#x5982;&#xFF0C;1/10)&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Join</b>
      </td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x521B;&#x5EFA;&#x5728;&#x5176;&#x952E;&#x4E0A;&#x76F8;&#x7B49;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x6765;&#x8FDE;&#x63A5;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x3002;&#x53EF;&#x9009;&#x5730;&#x4F7F;&#x7528;JoinFunction&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x6216;&#x4F7F;&#x7528;FlatJoinFunction&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x4EFB;&#x610F;&#x591A;&#x4E2A;&#xFF08;&#x5305;&#x62EC;&#x65E0;&#xFF09;&#x5143;&#x7D20;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys">&#x952E;&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;&#x8FDE;&#x63A5;&#x952E;&#x3002;</p>
        <p>result <b>=</b> input1<b>.</b>join(input2)</p>
        <p> <b>.</b>where(0) // key of the first input (tuple field 0)</p>
        <p> <b>.</b>equalTo(1); // key of the second input (tuple field 1)</p>
        <p>&#x53EF;&#x4EE5;&#x901A;&#x8FC7;Join Hints&#x6307;&#x5B9A;&#x8FD0;&#x884C;&#x65F6;&#x6267;&#x884C;&#x8FDE;&#x63A5;&#x7684;&#x65B9;&#x5F0F;&#x3002;&#x63D0;&#x793A;&#x63CF;&#x8FF0;&#x4E86;&#x901A;&#x8FC7;&#x5206;&#x533A;&#x6216;&#x5E7F;&#x64AD;&#x8FDB;&#x884C;&#x8FDE;&#x63A5;&#xFF0C;&#x4EE5;&#x53CA;&#x5B83;&#x662F;&#x4F7F;&#x7528;&#x57FA;&#x4E8E;&#x6392;&#x5E8F;&#x8FD8;&#x662F;&#x57FA;&#x4E8E;&#x6563;&#x5217;&#x7684;&#x7B97;&#x6CD5;&#x3002;&#x6709;&#x5173;&#x53EF;&#x80FD;&#x7684;&#x63D0;&#x793A;&#x548C;&#x793A;&#x4F8B;&#x7684;&#x5217;&#x8868;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;&#x201C;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#join-algorithm-hints">&#x8F6C;&#x6362;&#x6307;&#x5357;&#x201D;</a>&#x3002;</p>
        <p>&#x5982;&#x679C;&#x672A;&#x6307;&#x5B9A;&#x63D0;&#x793A;&#xFF0C;&#x7CFB;&#x7EDF;&#x5C06;&#x5C1D;&#x8BD5;&#x4F30;&#x7B97;&#x8F93;&#x5165;&#x5927;&#x5C0F;&#xFF0C;&#x5E76;&#x6839;&#x636E;&#x8FD9;&#x4E9B;&#x4F30;&#x8BA1;&#x9009;&#x62E9;&#x6700;&#x4F73;&#x7B56;&#x7565;&#x3002;</p>
        <p>// This executes a join by broadcasting the first data set</p>
        <p>// using a hash table for the broadcast data</p>
        <p>result <b>=</b> input1<b>.</b>join(input2, JoinHint<b>.</b>BROADCAST_HASH_FIRST)</p>
        <p> <b>.</b>where(0)<b>.</b>equalTo(1);</p>
        <p>&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x8FDE;&#x63A5;&#x8F6C;&#x6362;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x7B49;&#x8FDE;&#x63A5;&#x3002;&#x5176;&#x4ED6;&#x8FDE;&#x63A5;&#x7C7B;&#x578B;&#x9700;&#x8981;&#x4F7F;&#x7528;OuterJoin&#x6216;CoGroup&#x8868;&#x793A;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>OuterJoin</b>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x4E0A;&#x6267;&#x884C;&#x5DE6;&#xFF0C;&#x53F3;&#x6216;&#x5168;&#x5916;&#x8FDE;&#x63A5;&#x3002;&#x5916;&#x8FDE;&#x63A5;&#x7C7B;&#x4F3C;&#x4E8E;&#x5E38;&#x89C4;&#xFF08;&#x5185;&#x90E8;&#xFF09;&#x8FDE;&#x63A5;&#xFF0C;&#x5E76;&#x521B;&#x5EFA;&#x5728;&#x5176;&#x952E;&#x4E0A;&#x76F8;&#x7B49;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5982;&#x679C;&#x5728;&#x53E6;&#x4E00;&#x4FA7;&#x6CA1;&#x6709;&#x627E;&#x5230;&#x5339;&#x914D;&#x7684;&#x5BC6;&#x94A5;&#xFF0C;&#x5219;&#x4FDD;&#x7559;&#x201C;&#x5916;&#x90E8;&#x201D;&#x4FA7;&#xFF08;&#x5DE6;&#x4FA7;&#xFF0C;&#x53F3;&#x4FA7;&#x6216;&#x4E24;&#x8005;&#x90FD;&#x6EE1;&#xFF09;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5339;&#x914D;&#x5143;&#x7D20;&#x5BF9;&#xFF08;&#x6216;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x548C;null&#x53E6;&#x4E00;&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x503C;&#xFF09;&#x88AB;&#x8D4B;&#x4E88;JoinFunction&#x4EE5;&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x6216;&#x8005;&#x7ED9;&#x4E88;FlatJoinFunction&#x4EE5;&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x4EFB;&#x610F;&#x591A;&#x4E2A;&#xFF08;&#x5305;&#x62EC;&#x65E0;&#xFF09;&#x5143;&#x7D20;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys">&#x952E;&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;&#x8FDE;&#x63A5;&#x952E;&#x3002;</p>
        <p>input1<b>.</b>leftOuterJoin(input2) // rightOuterJoin or fullOuterJoin
          for right or full outer joins</p>
        <p> <b>.</b>where(0) // key of the first input (tuple field 0)</p>
        <p> <b>.</b>equalTo(1) // key of the second input (tuple field 1)</p>
        <p> <b>.</b>with(<b>new</b> JoinFunction<b>&lt;</b>String, String, String<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b> String <b>join</b>(String v1, String v2) <b>{</b>
        </p>
        <p>// NOTE:</p>
        <p>// - v2 might be null for leftOuterJoin</p>
        <p>// - v1 might be null for rightOuterJoin</p>
        <p>// - v1 OR v2 might be null for fullOuterJoin</p>
        <p> <b>}</b>
        </p>
        <p> <b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>CoGroup</b>
      </td>
      <td style="text-align:left">
        <p><b>reduce</b>&#x64CD;&#x4F5C;&#x7684;&#x4E8C;&#x7EF4;&#x53D8;&#x4F53;&#x3002;&#x5C06;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x4E0A;&#x7684;&#x6BCF;&#x4E2A;&#x8F93;&#x5165;&#x5206;&#x7EC4;&#xFF0C;&#x7136;&#x540E;&#x52A0;&#x5165;&#x7EC4;&#x3002;&#x6BCF;&#x5BF9;&#x7EC4;&#x8C03;&#x7528;&#x8F6C;&#x6362;&#x51FD;&#x6570;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys"><b>keys</b>&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;<b>coGroup</b>&#x952E;&#x3002;</p>
        <p>data1<b>.</b>coGroup(data2)</p>
        <p> <b>.</b>where(0)</p>
        <p> <b>.</b>equalTo(1)</p>
        <p> <b>.</b>with(<b>new</b> CoGroupFunction<b>&lt;</b>String, String, String<b>&gt;</b>() <b>{</b>
        </p>
        <p> <b>public</b>  <b>void</b>  <b>coGroup</b>(Iterable<b>&lt;</b>String<b>&gt;</b> in1,
          Iterable<b>&lt;</b>String<b>&gt;</b> in2, Collector<b>&lt;</b>String<b>&gt;</b> out) <b>{</b>
        </p>
        <p>out<b>.</b>collect(<b>...</b>);</p>
        <p> <b>}</b>
        </p>
        <p> <b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Cross</b>
      </td>
      <td style="text-align:left">
        <p>&#x6784;&#x5EFA;&#x4E24;&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x7B1B;&#x5361;&#x5C14;&#x79EF;&#xFF08;&#x4EA4;&#x53C9;&#x4E58;&#x79EF;&#xFF09;&#xFF0C;&#x521B;&#x5EFA;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x3002;&#x53EF;&#x9009;&#x62E9;&#x4F7F;&#x7528;CrossFunction&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;</p>
        <p>DataSet<b>&lt;</b>Integer<b>&gt;</b> data1 <b>=</b> // [...]</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> data2 <b>=</b> // [...]</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>Integer, String<b>&gt;&gt;</b> result <b>=</b> data1<b>.</b>cross(data2);</p>
        <p>&#x6CE8;&#xFF1A;&#x4EA4;&#x53C9;&#x662F;&#x4E00;&#x4E2A;&#x6F5C;&#x5728;&#x7684;&#x975E;&#x5E38;&#x8BA1;&#x7B97;&#x5BC6;&#x96C6;&#x578B;&#x64CD;&#x4F5C;&#x5B83;&#x751A;&#x81F3;&#x53EF;&#x4EE5;&#x6311;&#x6218;&#x5927;&#x7684;&#x8BA1;&#x7B97;&#x96C6;&#x7FA4;&#xFF01;&#x5EFA;&#x8BAE;&#x4F7F;&#x7528;crossWithTiny()&#x548C;crossWithHuge()&#x6765;&#x63D0;&#x793A;&#x7CFB;&#x7EDF;&#x7684;DataSet&#x5927;&#x5C0F;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Union</b>
      </td>
      <td style="text-align:left">
        <p>&#x751F;&#x6210;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x7684;&#x5E76;&#x96C6;&#x3002;</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> data1 <b>=</b> // [...]</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> data2 <b>=</b> // [...]</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> result <b>=</b> data1<b>.</b>union(data2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Rebalance</b>
      </td>
      <td style="text-align:left">
        <p>&#x5747;&#x5300;&#x5730;&#x91CD;&#x65B0;&#x5E73;&#x8861;&#x6570;&#x636E;&#x96C6;&#x7684;&#x5E76;&#x884C;&#x5206;&#x533A;&#x4EE5;&#x6D88;&#x9664;&#x6570;&#x636E;&#x504F;&#x5DEE;&#x3002;&#x53EA;&#x6709;&#x7C7B;&#x4F3C;&#x5730;&#x56FE;&#x7684;&#x8F6C;&#x6362;&#x53EF;&#x80FD;&#x4F1A;&#x9075;&#x5FAA;&#x91CD;&#x65B0;&#x5E73;&#x8861;&#x8F6C;&#x6362;&#x3002;</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> in <b>=</b> // [...]</p>
        <p>DataSet<b>&lt;</b>String<b>&gt;</b> result <b>=</b> in<b>.</b>rebalance()</p>
        <p> <b>.</b>map(<b>new</b> Mapper());</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Hash-Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7ED9;&#x5B9A;&#x952E;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x8FDB;&#x884C;Hash&#x5206;&#x533A;&#x3002;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Integer<b>&gt;</b> result <b>=</b> in<b>.</b>partitionByHash(0)</p>
        <p> <b>.</b>mapPartition(<b>new</b> PartitionMapper());</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Range-Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7ED9;&#x5B9A;&#x952E;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x8FDB;&#x884C;&#x8303;&#x56F4;&#x5206;&#x533A;&#x3002;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Integer<b>&gt;</b> result <b>=</b> in<b>.</b>partitionByRange(0)</p>
        <p> <b>.</b>mapPartition(<b>new</b> PartitionMapper());</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Custom Partitioning</b>
      </td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x81EA;&#x5B9A;&#x4E49;&#x5206;&#x533A;&#x7A0B;&#x5E8F;&#x529F;&#x80FD;&#x57FA;&#x4E8E;&#x7279;&#x5B9A;&#x5206;&#x533A;&#x7684;&#x952E;&#x5206;&#x914D;&#x8BB0;&#x5F55;&#x3002;&#x5BC6;&#x94A5;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x5F0F;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;<b><br /></b> &#x6CE8;&#x610F;&#xFF1A;&#x6B64;&#x65B9;&#x6CD5;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x5355;&#x4E2A;&#x5B57;&#x6BB5;&#x952E;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Integer<b>&gt;</b> result <b>=</b> in<b>.</b>partitionCustom(partitioner,
          key)</p>
        <p> <b>.</b>mapPartition(<b>new</b> PartitionMapper());</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Sort Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x672C;&#x5730;&#x6309;&#x6307;&#x5B9A;&#x987A;&#x5E8F;&#x5BF9;&#x6307;&#x5B9A;&#x5B57;&#x6BB5;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x7684;&#x6240;&#x6709;&#x5206;&#x533A;&#x8FDB;&#x884C;&#x6392;&#x5E8F;&#x3002;&#x53EF;&#x4EE5;&#x5C06;&#x5B57;&#x6BB5;&#x6307;&#x5B9A;&#x4E3A;&#x5143;&#x7EC4;&#x4F4D;&#x7F6E;&#x6216;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x901A;&#x8FC7;&#x94FE;&#x63A5;<b>sortPartition()</b>&#x8C03;&#x7528;&#x6765;&#x5B8C;&#x6210;&#x5BF9;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x7684;&#x6392;&#x5E8F;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Integer<b>&gt;</b> result <b>=</b> in<b>.</b>sortPartition(1,
          Order<b>.</b>ASCENDING)</p>
        <p> <b>.</b>mapPartition(<b>new</b> PartitionMapper());</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>First-n</b>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x636E;&#x96C6;&#x7684;&#x524D;n&#x4E2A;&#xFF08;&#x4EFB;&#x610F;&#xFF09;&#x5143;&#x7D20;&#x3002;First-n&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5E38;&#x89C4;&#x6570;&#x636E;&#x96C6;&#xFF0C;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6392;&#x5E8F;&#x6570;&#x636E;&#x96C6;&#x3002;&#x5206;&#x7EC4;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x6216;&#x5B57;&#x6BB5;&#x4F4D;&#x7F6E;&#x952E;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>// regular data set</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> result1 <b>=</b> in<b>.</b>first(3);</p>
        <p>// grouped data set</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> result2 <b>=</b> in<b>.</b>groupBy(0)</p>
        <p> <b>.</b>first(3);</p>
        <p>// grouped-sorted data set</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String,Integer<b>&gt;&gt;</b> result3 <b>=</b> in<b>.</b>groupBy(0)</p>
        <p> <b>.</b>sortGroup(1, Order<b>.</b>ASCENDING)</p>
        <p> <b>.</b>first(3);</p>
      </td>
    </tr>
  </tbody>
</table>

元组数据集可进行以下转换:

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Project</b>
      </td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x5143;&#x7EC4;&#x4E2D;&#x9009;&#x62E9;&#x5B57;&#x6BB5;&#x7684;&#x5B50;&#x96C6;</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, Double, String<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>DataSet<b>&lt;</b>Tuple2<b>&lt;</b>String, Integer<b>&gt;&gt;</b> out <b>=</b> in<b>.</b>project(2,0);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MinBy / MaxBy</b>
      </td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x4E00;&#x7EC4;&#x5143;&#x7EC4;&#x4E2D;&#x9009;&#x62E9;&#x4E00;&#x4E2A;&#x5143;&#x7EC4;&#xFF0C;&#x5176;&#x5143;&#x7EC4;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x7684;&#x503C;&#x6700;&#x5C0F;&#xFF08;&#x6700;&#x5927;&#xFF09;&#x3002;&#x7528;&#x4E8E;&#x6BD4;&#x8F83;&#x7684;&#x5B57;&#x6BB5;&#x5FC5;&#x987B;&#x662F;&#x6709;&#x6548;&#x7684;&#x5173;&#x952E;&#x5B57;&#x6BB5;&#xFF0C;&#x5373;&#x53EF;&#x6BD4;&#x8F83;&#x7684;&#x5B57;&#x6BB5;&#x3002;&#x5982;&#x679C;&#x591A;&#x4E2A;&#x5143;&#x7EC4;&#x5177;&#x6709;&#x6700;&#x5C0F;&#xFF08;&#x6700;&#x5927;&#xFF09;&#x5B57;&#x6BB5;&#x503C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x8FD9;&#x4E9B;&#x5143;&#x7EC4;&#x7684;&#x4EFB;&#x610F;&#x5143;&#x7EC4;&#x3002;MinBy&#xFF08;MaxBy&#xFF09;&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, Double, String<b>&gt;&gt;</b> in <b>=</b> //
          [...]</p>
        <p>// a DataSet with a single tuple with minimum values for the Integer and
          String fields.</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, Double, String<b>&gt;&gt;</b> out <b>=</b> in<b>.</b>minBy(0,
          2);</p>
        <p>// a DataSet with one tuple for each group with the minimum value for
          the Double field.</p>
        <p>DataSet<b>&lt;</b>Tuple3<b>&lt;</b>Integer, Double, String<b>&gt;&gt;</b> out2 <b>=</b> in<b>.</b>groupBy(2)</p>
        <p> <b>.</b>minBy(1);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Map</b>
      </td>
      <td style="text-align:left">
        <p>&#x83B7;&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;</p>
        <p>data<b>.</b>map <b>{</b> x <b>=&gt;</b> x<b>.</b>toInt <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>FlatMap</b>
      </td>
      <td style="text-align:left">
        <p>&#x83B7;&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x96F6;&#x4E2A;&#xFF0C;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;</p>
        <p>data<b>.</b>flatMap <b>{</b> str <b>=&gt;</b> str<b>.</b>split(&quot; &quot;) <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MapPartition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x5355;&#x4E2A;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x4E2D;&#x8F6C;&#x6362;&#x5E76;&#x884C;&#x5206;&#x533A;&#x3002;&#x8BE5;&#x51FD;&#x6570;&#x5C06;&#x5206;&#x533A;&#x4F5C;&#x4E3A;&#x201C;&#x8FED;&#x4EE3;&#x5668;&#x201D;&#xFF0C;&#x5E76;&#x53EF;&#x4EE5;&#x751F;&#x6210;&#x4EFB;&#x610F;&#x6570;&#x91CF;&#x7684;&#x7ED3;&#x679C;&#x503C;&#x3002;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x4E2D;&#x7684;&#x5143;&#x7D20;&#x6570;&#x91CF;&#x53D6;&#x51B3;&#x4E8E;&#x5E76;&#x884C;&#x5EA6;&#x548C;&#x5148;&#x524D;&#x7684;&#x64CD;&#x4F5C;&#x3002;</p>
        <p>data<b>.</b>mapPartition <b>{</b> in <b>=&gt;</b> in map <b>{</b> (<b>_</b>,
          1) <b>}</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Filter</b>
      </td>
      <td style="text-align:left">
        <p>&#x8BA1;&#x7B97;&#x6BCF;&#x4E2A;&#x5143;&#x7D20;&#x7684;&#x5E03;&#x5C14;&#x51FD;&#x6570;&#xFF0C;&#x5E76;&#x4FDD;&#x7559;&#x51FD;&#x6570;&#x8FD4;&#x56DE;true&#x7684;&#x5143;&#x7D20;&#x3002;
          <br
          />&#x91CD;&#x8981;&#x4FE1;&#x606F;&#xFF1A;&#x7CFB;&#x7EDF;&#x5047;&#x5B9A;&#x8BE5;&#x51FD;&#x6570;&#x4E0D;&#x4F1A;&#x4FEE;&#x6539;&#x5E94;&#x7528;&#x8C13;&#x8BCD;&#x7684;&#x5143;&#x7D20;&#x3002;&#x8FDD;&#x53CD;&#x6B64;&#x5047;&#x8BBE;&#x53EF;&#x80FD;&#x4F1A;&#x5BFC;&#x81F4;&#x9519;&#x8BEF;&#x7684;&#x7ED3;&#x679C;&#x3002;</p>
        <p>data<b>.</b>filter <b>{</b>  <b>_</b>  <b>&gt;</b> 1000 <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Reduce</b>
      </td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x5C06;&#x4E24;&#x4E2A;&#x5143;&#x7D20;&#x91CD;&#x590D;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x5C06;&#x4E00;&#x7EC4;&#x5143;&#x7D20;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;Reduce&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>data<b>.</b>reduce <b>{</b>  <b>_</b>  <b>+</b>  <b>_</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>ReduceGroup</b>
      </td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x7EC4;&#x5143;&#x7D20;&#x7EC4;&#x5408;&#x6210;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;ReduceGroup&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>data<b>.</b>reduceGroup <b>{</b> elements <b>=&gt;</b> elements<b>.</b>sum <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregate</b>
      </td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x7EC4;&#x503C;&#x805A;&#x5408;&#x4E3A;&#x5355;&#x4E2A;&#x503C;&#x3002;&#x805A;&#x5408;&#x51FD;&#x6570;&#x53EF;&#x4EE5;&#x88AB;&#x8BA4;&#x4E3A;&#x662F;&#x5185;&#x7F6E;&#x7684;<b>reduce</b>&#x51FD;&#x6570;&#x3002;&#x805A;&#x5408;&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p><b>val</b> input<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>, <b>Double</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> output<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>, <b>Double</b>)] <b>=</b> input<b>.</b>aggregate(<b>SUM</b>,
          0)<b>.</b>aggregate(<b>MIN</b>, 2)</p>
        <p>&#x8FD8;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x7B80;&#x5199;&#x8BED;&#x6CD5;&#x8FDB;&#x884C;&#x6700;&#x5C0F;&#xFF0C;&#x6700;&#x5927;&#x548C;&#x603B;&#x548C;&#x805A;&#x5408;&#x3002;</p>
        <p><b>val</b> input<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>, <b>Double</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> output<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>, <b>Double</b>)] <b>=</b> input<b>.</b>sum(0)<b>.</b>min(2)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct</b>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x636E;&#x96C6;&#x7684;&#x4E0D;&#x540C;&#x5143;&#x7D20;&#x3002;&#x76F8;&#x5BF9;&#x4E8E;&#x5143;&#x7D20;&#x7684;&#x6240;&#x6709;&#x5B57;&#x6BB5;&#x6216;&#x5B57;&#x6BB5;&#x5B50;&#x96C6;&#x4ECE;&#x8F93;&#x5165;DataSet&#x4E2D;&#x5220;&#x9664;&#x91CD;&#x590D;&#x6761;&#x76EE;&#x3002;</p>
        <p>data<b>.</b>distinct()</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Join</b>
      </td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x521B;&#x5EFA;&#x5728;&#x5176;&#x952E;&#x4E0A;&#x76F8;&#x7B49;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x6765;&#x5173;&#x8054;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x3002;&#x53EF;&#x9009;&#x5730;&#x4F7F;&#x7528;JoinFunction&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x6216;&#x4F7F;&#x7528;FlatJoinFunction&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x4EFB;&#x610F;&#x591A;&#x4E2A;&#xFF08;&#x5305;&#x62EC;&#x65E0;&#xFF09;&#x5143;&#x7D20;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys">&#x952E;&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;&#x5173;&#x8054;&#x952E;&#x3002;</p>
        <p>// In this case tuple fields are used as keys. &quot;0&quot; is the join
          field on the first tuple</p>
        <p>// &quot;1&quot; is the join field on the second tuple.</p>
        <p><b>val</b> result <b>=</b> input1<b>.</b>join(input2)<b>.</b>where(0)<b>.</b>equalTo(1)</p>
        <p>&#x53EF;&#x4EE5;&#x901A;&#x8FC7;Join Hints&#x6307;&#x5B9A;&#x8FD0;&#x884C;&#x65F6;&#x6267;&#x884C;&#x5173;&#x8054;&#x7684;&#x65B9;&#x5F0F;&#x3002;&#x63D0;&#x793A;&#x63CF;&#x8FF0;&#x4E86;&#x901A;&#x8FC7;&#x5206;&#x533A;&#x6216;&#x5E7F;&#x64AD;&#x8FDB;&#x884C;&#x8FDE;&#x63A5;&#xFF0C;&#x4EE5;&#x53CA;&#x5B83;&#x662F;&#x4F7F;&#x7528;&#x57FA;&#x4E8E;&#x6392;&#x5E8F;&#x8FD8;&#x662F;&#x57FA;&#x4E8E;&#x6563;&#x5217;&#x7684;&#x7B97;&#x6CD5;&#x3002;&#x6709;&#x5173;&#x53EF;&#x80FD;&#x7684;&#x63D0;&#x793A;&#x548C;&#x793A;&#x4F8B;&#x7684;&#x5217;&#x8868;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;&#x201C;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#join-algorithm-hints">&#x8F6C;&#x6362;&#x6307;&#x5357;&#x201D;</a>&#x3002;</p>
        <p>&#x5982;&#x679C;&#x672A;&#x6307;&#x5B9A;&#x63D0;&#x793A;&#xFF0C;&#x7CFB;&#x7EDF;&#x5C06;&#x5C1D;&#x8BD5;&#x4F30;&#x7B97;&#x8F93;&#x5165;&#x5927;&#x5C0F;&#xFF0C;&#x5E76;&#x6839;&#x636E;&#x8FD9;&#x4E9B;&#x4F30;&#x8BA1;&#x9009;&#x62E9;&#x6700;&#x4F73;&#x7B56;&#x7565;&#x3002;</p>
        <p>// This executes a join by broadcasting the first data set</p>
        <p>// using a hash table for the broadcast data</p>
        <p><b>val</b> result <b>=</b> input1<b>.</b>join(input2, <b>JoinHint.BROADCAST_HASH_FIRST</b>)</p>
        <p> <b>.</b>where(0)<b>.</b>equalTo(1)</p>
        <p>&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x8FDE;&#x63A5;&#x8F6C;&#x6362;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x7B49;&#x8FDE;&#x63A5;&#x3002;&#x5176;&#x4ED6;&#x8FDE;&#x63A5;&#x7C7B;&#x578B;&#x9700;&#x8981;&#x4F7F;&#x7528;OuterJoin&#x6216;CoGroup&#x8868;&#x793A;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>OuterJoin</b>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x4E0A;&#x6267;&#x884C;&#x5DE6;&#xFF0C;&#x53F3;&#x6216;&#x5168;&#x5916;&#x5173;&#x8054;&#x3002;&#x5916;&#x5173;&#x8054;&#x7C7B;&#x4F3C;&#x4E8E;&#x5E38;&#x89C4;&#xFF08;&#x5185;&#x90E8;&#xFF09;&#x8FDE;&#x63A5;&#xFF0C;&#x5E76;&#x521B;&#x5EFA;&#x5728;&#x5176;&#x952E;&#x4E0A;&#x76F8;&#x7B49;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5982;&#x679C;&#x5728;&#x53E6;&#x4E00;&#x4FA7;&#x6CA1;&#x6709;&#x627E;&#x5230;&#x5339;&#x914D;&#x7684;&#x5BC6;&#x94A5;&#xFF0C;&#x5219;&#x4FDD;&#x7559;&#x201C;&#x5916;&#x90E8;&#x201D;&#x4FA7;&#xFF08;&#x5DE6;&#x4FA7;&#xFF0C;&#x53F3;&#x4FA7;&#x6216;&#x4E24;&#x8005;&#x90FD;&#x6EE1;&#xFF09;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5339;&#x914D;&#x5143;&#x7D20;&#x5BF9;&#xFF08;&#x6216;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x548C;&#x53E6;&#x4E00;&#x4E2A;&#x8F93;&#x5165;&#x7684;`null`&#x503C;&#xFF09;&#x88AB;&#x8D4B;&#x4E88;JoinFunction&#x4EE5;&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;&#xFF0C;&#x6216;&#x8005;&#x7ED9;&#x4E88;FlatJoinFunction&#x4EE5;&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x4EFB;&#x610F;&#x591A;&#x4E2A;&#xFF08;&#x5305;&#x62EC;&#x65E0;&#xFF09;&#x5143;&#x7D20;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys">&#x952E;&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;&#x5173;&#x8054;&#x952E;&#x3002;</p>
        <p><b>val</b> joined <b>=</b> left<b>.</b>leftOuterJoin(right)<b>.</b>where(0)<b>.</b>equalTo(1) <b>{</b>
        </p>
        <p>(left, right) <b>=&gt;</b>
        </p>
        <p> <b>val</b> a <b>=</b>  <b>if</b> (left <b>==</b>  <b>null</b>) &quot;none&quot; <b>else</b> left<b>.</b>_1</p>
        <p>(a, right)</p>
        <p> <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>CoGroup</b>
      </td>
      <td style="text-align:left">
        <p><b>reduce</b>&#x64CD;&#x4F5C;&#x7684;&#x4E8C;&#x7EF4;&#x53D8;&#x4F53;&#x3002;&#x5C06;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x4E0A;&#x7684;&#x6BCF;&#x4E2A;&#x8F93;&#x5165;&#x5206;&#x7EC4;&#xFF0C;&#x7136;&#x540E;&#x52A0;&#x5165;&#x7EC4;&#x3002;&#x6BCF;&#x5BF9;&#x7EC4;&#x8C03;&#x7528;&#x8F6C;&#x6362;&#x51FD;&#x6570;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#specifying-keys"><b>keys</b>&#x90E8;&#x5206;</a>&#x4EE5;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5B9A;&#x4E49;<b>coGroup</b>&#x952E;&#x3002;</p>
        <p>data1<b>.</b>coGroup(data2)<b>.</b>where(0)<b>.</b>equalTo(1)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Cross</b>
      </td>
      <td style="text-align:left">
        <p>&#x6784;&#x5EFA;&#x4E24;&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x7B1B;&#x5361;&#x5C14;&#x79EF;&#xFF08;&#x4EA4;&#x53C9;&#x4E58;&#x79EF;&#xFF09;&#xFF0C;&#x521B;&#x5EFA;&#x6240;&#x6709;&#x5143;&#x7D20;&#x5BF9;&#x3002;&#x53EF;&#x9009;&#x62E9;&#x4F7F;&#x7528;<b>CrossFunction</b>&#x5C06;&#x5143;&#x7D20;&#x5BF9;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x4E2A;&#x5143;&#x7D20;</p>
        <p><b>val</b> data1<b>:</b>  <b>DataSet</b>[<b>Int</b>] <b>=</b> // [...]</p>
        <p><b>val</b> data2<b>:</b>  <b>DataSet</b>[<b>String</b>] <b>=</b> // [...]</p>
        <p><b>val</b> result<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> data1<b>.</b>cross(data2)</p>
        <p>&#x6CE8;&#xFF1A;&#x4EA4;&#x53C9;&#x662F;&#x4E00;&#x4E2A;&#x6F5C;&#x5728;&#x7684;&#x975E;&#x5E38;&#x8BA1;&#x7B97;&#x5BC6;&#x96C6;&#x578B;&#x64CD;&#x4F5C;&#x5B83;&#x751A;&#x81F3;&#x53EF;&#x4EE5;&#x6311;&#x6218;&#x5927;&#x7684;&#x8BA1;&#x7B97;&#x96C6;&#x7FA4;&#xFF01;&#x5EFA;&#x8BAE;&#x4F7F;&#x7528;crossWithTiny()&#x548C;crossWithHuge()&#x6765;&#x63D0;&#x793A;&#x7CFB;&#x7EDF;&#x7684;<b>DataSet</b>&#x5927;&#x5C0F;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Union</b>
      </td>
      <td style="text-align:left">
        <p>&#x751F;&#x6210;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x96C6;&#x7684;&#x5E76;&#x96C6;&#x3002;</p>
        <p>data<b>.</b>union(data2)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Rebalance</b>
      </td>
      <td style="text-align:left">
        <p>&#x5747;&#x5300;&#x5730;&#x91CD;&#x65B0;&#x5E73;&#x8861;&#x6570;&#x636E;&#x96C6;&#x7684;&#x5E76;&#x884C;&#x5206;&#x533A;&#x4EE5;&#x6D88;&#x9664;&#x6570;&#x636E;&#x504F;&#x5DEE;&#x3002;&#x53EA;&#x6709;&#x7C7B;&#x4F3C;&#x5730;&#x56FE;&#x7684;&#x8F6C;&#x6362;&#x53EF;&#x80FD;&#x4F1A;&#x9075;&#x5FAA;&#x91CD;&#x65B0;&#x5E73;&#x8861;&#x8F6C;&#x6362;&#x3002;</p>
        <p><b>val</b> data1<b>:</b>  <b>DataSet</b>[<b>Int</b>] <b>=</b> // [...]</p>
        <p><b>val</b> result<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> data1<b>.</b>rebalance()<b>.</b>map(<b>...</b>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Hash-Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7ED9;&#x5B9A;&#x952E;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x8FDB;&#x884C;Hash&#x5206;&#x533A;&#x3002;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;</p>
        <p><b>val</b> in<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> result <b>=</b> in<b>.</b>partitionByHash(0)<b>.</b>mapPartition <b>{</b>  <b>...</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Range-Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7ED9;&#x5B9A;&#x952E;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x8FDB;&#x884C;&#x8303;&#x56F4;&#x5206;&#x533A;&#x3002;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;</p>
        <p><b>val</b> in<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> result <b>=</b> in<b>.</b>partitionByRange(0)<b>.</b>mapPartition <b>{</b>  <b>...</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Custom Partitioning</b>
      </td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x81EA;&#x5B9A;&#x4E49;&#x5206;&#x533A;&#x7A0B;&#x5E8F;&#x529F;&#x80FD;&#x57FA;&#x4E8E;&#x7279;&#x5B9A;&#x5206;&#x533A;&#x7684;&#x952E;&#x5206;&#x914D;&#x8BB0;&#x5F55;&#x3002;&#x5BC6;&#x94A5;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x4F4D;&#x7F6E;&#x952E;&#xFF0C;&#x8868;&#x8FBE;&#x5F0F;&#x952E;&#x548C;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#x3002;<b><br /></b> &#x6CE8;&#x610F;&#xFF1A;&#x6B64;&#x65B9;&#x6CD5;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x5355;&#x4E2A;&#x5B57;&#x6BB5;&#x952E;&#x3002;</p>
        <p><b>val</b> in<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> result <b>=</b> in</p>
        <p> <b>.</b>partitionCustom(partitioner, key)<b>.</b>mapPartition <b>{</b>  <b>...</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Sort Partition</b>
      </td>
      <td style="text-align:left">
        <p>&#x672C;&#x5730;&#x6309;&#x6307;&#x5B9A;&#x987A;&#x5E8F;&#x5BF9;&#x6307;&#x5B9A;&#x5B57;&#x6BB5;&#x4E0A;&#x7684;&#x6570;&#x636E;&#x96C6;&#x7684;&#x6240;&#x6709;&#x5206;&#x533A;&#x8FDB;&#x884C;&#x6392;&#x5E8F;&#x3002;&#x53EF;&#x4EE5;&#x5C06;&#x5B57;&#x6BB5;&#x6307;&#x5B9A;&#x4E3A;&#x5143;&#x7EC4;&#x4F4D;&#x7F6E;&#x6216;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x901A;&#x8FC7;&#x94FE;&#x63A5;<b>sortPartition()</b>&#x8C03;&#x7528;&#x6765;&#x5B8C;&#x6210;&#x5BF9;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x7684;&#x6392;&#x5E8F;&#x3002;</p>
        <p><b>val</b> in<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> //
          [...]</p>
        <p><b>val</b> result <b>=</b> in<b>.</b>sortPartition(1, <b>Order.ASCENDING</b>)<b>.</b>mapPartition <b>{</b>  <b>...</b>  <b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>First-n</b>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x636E;&#x96C6;&#x7684;&#x524D;<b>n</b>&#x4E2A;&#xFF08;&#x4EFB;&#x610F;&#xFF09;&#x5143;&#x7D20;&#x3002;<b>First-n</b>&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5E38;&#x89C4;&#x6570;&#x636E;&#x96C6;&#xFF0C;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6392;&#x5E8F;&#x6570;&#x636E;&#x96C6;&#x3002;&#x5206;&#x7EC4;&#x952E;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x952E;&#x9009;&#x62E9;&#x5668;&#x529F;&#x80FD;&#xFF0C;&#x5143;&#x7EC4;&#x4F4D;&#x7F6E;&#x6216;&#x6848;&#x4F8B;&#x7C7B;&#x5B57;&#x6BB5;&#x3002;</p>
        <p><b>val</b> in<b>:</b>  <b>DataSet</b>[(<b>Int</b>, <b>String</b>)] <b>=</b> //
          [...]</p>
        <p>// regular data set</p>
        <p><b>val</b> result1 <b>=</b> in<b>.</b>first(3)</p>
        <p>// grouped data set</p>
        <p><b>val</b> result2 <b>=</b> in<b>.</b>groupBy(0)<b>.</b>first(3)</p>
        <p>// grouped-sorted data set</p>
        <p><b>val</b> result3 <b>=</b> in<b>.</b>groupBy(0)<b>.</b>sortGroup(1, <b>Order.ASCENDING</b>)<b>.</b>first(3)</p>
      </td>
    </tr>
  </tbody>
</table>

以下转换可用于元组的数据集：

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>MinBy / MaxBy</b>
      </td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x4E00;&#x7EC4;&#x5143;&#x7EC4;&#x4E2D;&#x9009;&#x62E9;&#x4E00;&#x4E2A;&#x5143;&#x7EC4;&#xFF0C;&#x5176;&#x5143;&#x7EC4;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5B57;&#x6BB5;&#x7684;&#x503C;&#x6700;&#x5C0F;&#xFF08;&#x6700;&#x5927;&#xFF09;&#x3002;&#x7528;&#x4E8E;&#x6BD4;&#x8F83;&#x7684;&#x5B57;&#x6BB5;&#x5FC5;&#x987B;&#x662F;&#x6709;&#x6548;&#x7684;&#x5173;&#x952E;&#x5B57;&#x6BB5;&#xFF0C;&#x5373;&#x53EF;&#x6BD4;&#x8F83;&#x7684;&#x5B57;&#x6BB5;&#x3002;&#x5982;&#x679C;&#x591A;&#x4E2A;&#x5143;&#x7EC4;&#x5177;&#x6709;&#x6700;&#x5C0F;&#xFF08;&#x6700;&#x5927;&#xFF09;&#x5B57;&#x6BB5;&#x503C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x8FD9;&#x4E9B;&#x5143;&#x7EC4;&#x7684;&#x4EFB;&#x610F;&#x5143;&#x7EC4;&#x3002;MinBy&#xFF08;MaxBy&#xFF09;&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;&#x5B8C;&#x6574;&#x6570;&#x636E;&#x96C6;&#x6216;&#x5206;&#x7EC4;&#x6570;&#x636E;&#x96C6;&#x3002;</p>
        <p>val in: DataSet[(Int, Double, String)] <b>=</b> // [...]</p>
        <p>// a data set with a single tuple with minimum values for the Int and
          String fields.</p>
        <p>val out: DataSet[(Int, Double, String)] <b>=</b> in<b>.</b>minBy(0, 2)</p>
        <p>// a data set with one tuple for each group with the minimum value for
          the Double field.</p>
        <p>val out2: DataSet[(Int, Double, String)] <b>=</b> in<b>.</b>groupBy(2)</p>
        <p> <b>.</b>minBy(1)</p>
      </td>
    </tr>
  </tbody>
</table>

通过匿名模式匹配从元组，案例类和集合中提取，如下所示：

```scala
val data: DataSet[(Int, String, Double)] = // [...]
data.map {
  case (id, name, temperature) => // [...]
}
```

API不支持开箱即用。要使用此功能，您应该使用[Scala API扩展](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/scala_api_extensions.html)。
{% endtab %}
{% endtabs %}

转换的[并行性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/parallel.html)可以通过setParallelism\(int\)定义，而name\(String\)则为转换分配一个定制的名称，这有助于调试。[数据源](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/#data-sources)和[数据接收器](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/#data-sinks)也可以这样做。

withParameters\(Configuration\)传递配置对象，可以从用户函数中的open\(\)方法访问配置对象。

## 数据源

数据源创建初始数据集，例如来自文件或Java集合的数据集。创建数据集的一般机制抽象在`InputFormat`后面。Flink提供了几种内置格式，可以从通用文件格式创建数据集。它们中的许多在`ExecutionEnvironment`上都有快捷方法。

{% tabs %}
{% tab title="Java" %}
基于文件的：

* `readTextFile(path)`/ `TextInputFormat`- 按行读取文件并将其作为字符串返回。
* `readTextFileWithValue(path)`/ `TextValueInputFormat`- 按行读取文件并将它们作为StringValues返回。StringValues是可变字符串。
* `readCsvFile(path)`/ `CsvInputFormat`- 解析逗号（或其他字符）分隔字段的文件。返回元组或POJO的DataSet。支持基本的java类型及其Value对应的字段类型。
* `readFileOfPrimitives(path, Class)`/ `PrimitiveInputFormat`- 使用给定的分隔符解析以新行\(或其他字符序列\)分隔的基本数据类型\(例如`String`或`Integer`\)的文件。
* `readFileOfPrimitives(path, delimiter, Class)`/ `PrimitiveInputFormat`- 使用给定的分隔符解析以新行\(或其他字符序列\)分隔的基本数据类型\(例如`String`或`Integer`\)的文件。
* `readSequenceFile(Key, Value, path)`/ `SequenceFileInputFormat`- 创建一个JobConf并从类型为SequenceFileInputFormat，Key class和Value类的指定路径中读取文件，并将它们返回为Tuple2 &lt;Key，Value&gt;。

基于集合：

* `fromCollection(Collection)` - 从Java Java.util.Collection创建数据集。集合中的所有元素必须属于同一类型。
* `fromCollection(Iterator, Class)` - 从迭代器创建数据集。该类指定迭代器返回的元素的数据类型。
* `fromElements(T ...)` - 根据给定的对象序列创建数据集。所有对象必须属于同一类型。
* `fromParallelCollection(SplittableIterator, Class)` - 并行地从迭代器创建数据集。该类指定迭代器返回的元素的数据类型。
* `generateSequence(from, to)` - 并行生成给定间隔中的数字序列。

通用：

* `readFile(inputFormat, path)`/ `FileInputFormat`- 接受文件输入格式。
* `createInput(inputFormat)`/ `InputFormat`- 接受通用输入格式。

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// read text file from local files system
DataSet<String> localLines = env.readTextFile("file:///path/to/my/textfile");

// read text file from a HDFS running at nnHost:nnPort
DataSet<String> hdfsLines = env.readTextFile("hdfs://nnHost:nnPort/path/to/my/textfile");

// read a CSV file with three fields
DataSet<Tuple3<Integer, String, Double>> csvInput = env.readCsvFile("hdfs:///the/CSV/file")
	                       .types(Integer.class, String.class, Double.class);

// read a CSV file with five fields, taking only two of them
DataSet<Tuple2<String, Double>> csvInput = env.readCsvFile("hdfs:///the/CSV/file")
                               .includeFields("10010")  // take the first and the fourth field
	                       .types(String.class, Double.class);

// read a CSV file with three fields into a POJO (Person.class) with corresponding fields
DataSet<Person>> csvInput = env.readCsvFile("hdfs:///the/CSV/file")
                         .pojoType(Person.class, "name", "age", "zipcode");

// read a file from the specified path of type SequenceFileInputFormat
DataSet<Tuple2<IntWritable, Text>> tuples =
 env.readSequenceFile(IntWritable.class, Text.class, "hdfs://nnHost:nnPort/path/to/file");

// creates a set from some given elements
DataSet<String> value = env.fromElements("Foo", "bar", "foobar", "fubar");

// generate a number sequence
DataSet<Long> numbers = env.generateSequence(1, 10000000);

// Read data from a relational database using the JDBC input format
DataSet<Tuple2<String, Integer> dbData =
    env.createInput(
      JDBCInputFormat.buildJDBCInputFormat()
                     .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")
                     .setDBUrl("jdbc:derby:memory:persons")
                     .setQuery("select name, age from persons")
                     .setRowTypeInfo(new RowTypeInfo(BasicTypeInfo.STRING_TYPE_INFO, BasicTypeInfo.INT_TYPE_INFO))
                     .finish()
    );

// Note: Flink's program compiler needs to infer the data types of the data items which are returned
// by an InputFormat. If this information cannot be automatically inferred, it is necessary to
// manually provide the type information as shown in the examples above.
```

**配置CSV解析**

Flink提供了许多用于CSV解析的配置选项：

* `types(Class ... types)`指定要解析的字段的类型。**必须配置已解析字段的类型。** 对于类型类Boolean.class，“True”（不区分大小写），“False”（不区分大小写），“1”和“0”被视为布尔值。
* `lineDelimiter(String del)`指定单个记录的分隔符。默认行分隔符是换行符`'\n'`。
* `fieldDelimiter(String del)`指定用于分隔记录字段的分隔符。默认字段分隔符是逗号字符`','`。
* `includeFields(boolean ... flag)`,, `includeFields(String mask)`或`includeFields(long bitMask)`定义从输入文件中读取哪些字段（以及要忽略的字段）。默认情况下，将解析前_n个_字段（由`types()`调用中的类型数定义）。
* `parseQuotedStrings(char quoteChar)`启用带引号的字符串解析。如果字符串字段的第一个字符是引号字符（_未_剪裁前导或拖尾空格），则字符串将被解析为带引号的字符串。引用字符串中的字段分隔符将被忽略。如果带引号的字符串字段的最后一个字符不是引号字符，或者引号字符出现在某个不是引用字符串字段的开头或结尾的点上（除非引号字符使用''转义，否则引用字符串解析失败）。如果启用了带引号的字符串解析并且该字段的第一个字符_不是_引用字符串，则该字符串将被解析为不带引号的字符串。默认情况下，禁用带引号的字符串解析。
* `ignoreComments(String commentPrefix)`指定注释前缀。所有以指定注释前缀开头的行都不会被解析和忽略。默认情况下，不会忽略任何行。
* `ignoreInvalidLines()`启用宽松解析，即忽略无法正确解析的行。默认情况下，禁用宽松解析，无效行引发异常。
* `ignoreFirstLine()`配置InputFormat以忽略输入文件的第一行。默认情况下，不会忽略任何行。

**递归遍历输入路径目录**

对于基于文件的输入，当输入路径是目录时，默认情况下不会枚举嵌套文件。相反，只读取基目录中的文件，而忽略嵌套文件。可以通过`recursive.file.enumeration`配置参数启用嵌套文件的递归枚举，如下例所示。

```java
// enable recursive enumeration of nested input files
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// create a configuration object
Configuration parameters = new Configuration();

// set the recursive enumeration parameter
parameters.setBoolean("recursive.file.enumeration", true);

// pass the configuration to the data source
DataSet<String> logs = env.readTextFile("file:///path/with.nested/files")
			  .withParameters(parameters);
```
{% endtab %}

{% tab title="Scala" %}
基于文件的：

* `readTextFile(path)`/ `TextInputFormat`- 按行读取文件并将其作为字符串返回。
* `readTextFileWithValue(path)`/ `TextValueInputFormat`- 按行读取文件并将它们作为StringValues返回。StringValues是可变字符串。
* `readCsvFile(path)`/ `CsvInputFormat`- 解析逗号（或其他字符）分隔字段的文件。返回元组或POJO的DataSet。支持基本的java类型及其Value对应的字段类型。
* `readFileOfPrimitives(path, delimiter`

  `)`/ `PrimitiveInputFormat`- 使用给定的分隔符解析以新行\(或其他字符序列\)分隔的基本数据类型\(例如`String`或`Integer`\)的文件。

* `readSequenceFile(Key, Value, path)`/ `SequenceFileInputFormat`- 创建一个JobConf并从类型为SequenceFileInputFormat，Key class和Value类的指定路径中读取文件，并将它们返回为Tuple2 &lt;Key，Value&gt;。

基于集合：

* `fromCollection(Collection)` - 从Java Java.util.Collection创建数据集。集合中的所有元素必须属于同一类型。
* `fromCollection(Iterator)` - 从迭代器创建数据集。该类指定迭代器返回的元素的数据类型。
* `fromElements(elements: _*)` - 根据给定的对象序列创建数据集。所有对象必须属于同一类型。
* `fromParallelCollection(SplittableIterator)` - 并行地从迭代器创建数据集。该类指定迭代器返回的元素的数据类型。
* `generateSequence(from, to)` - 并行生成给定间隔中的数字序列。

通用：

* `readFile(inputFormat, path)`/ `FileInputFormat`- 接受文件输入格式。
* `createInput(inputFormat)`/ `InputFormat`- 接受通用输入格式。

```scala
val env  = ExecutionEnvironment.getExecutionEnvironment

// read text file from local files system
val localLines = env.readTextFile("file:///path/to/my/textfile")

// read text file from a HDFS running at nnHost:nnPort
val hdfsLines = env.readTextFile("hdfs://nnHost:nnPort/path/to/my/textfile")

// read a CSV file with three fields
val csvInput = env.readCsvFile[(Int, String, Double)]("hdfs:///the/CSV/file")

// read a CSV file with five fields, taking only two of them
val csvInput = env.readCsvFile[(String, Double)](
  "hdfs:///the/CSV/file",
  includedFields = Array(0, 3)) // take the first and the fourth field

// CSV input can also be used with Case Classes
case class MyCaseClass(str: String, dbl: Double)
val csvInput = env.readCsvFile[MyCaseClass](
  "hdfs:///the/CSV/file",
  includedFields = Array(0, 3)) // take the first and the fourth field

// read a CSV file with three fields into a POJO (Person) with corresponding fields
val csvInput = env.readCsvFile[Person](
  "hdfs:///the/CSV/file",
  pojoFields = Array("name", "age", "zipcode"))

// create a set from some given elements
val values = env.fromElements("Foo", "bar", "foobar", "fubar")

// generate a number sequence
val numbers = env.generateSequence(1, 10000000)

// read a file from the specified path of type SequenceFileInputFormat
val tuples = env.readSequenceFile(classOf[IntWritable], classOf[Text],
 "hdfs://nnHost:nnPort/path/to/file")
```

**配置CSV解析**

Flink提供了许多用于CSV解析的配置选项：

* `lineDelimiter: String`指定单个记录的分隔符。默认行分隔符是换行符`'\n'`。
* `fieldDelimiter: String`指定用于分隔记录字段的分隔符。默认字段分隔符是逗号字符`','`。
* `includeFields: Array[Int]`定义要从输入文件中读取的字段（以及要忽略的字段）。默认情况下，将解析前_n个_字段（由`types()`调用中的类型数定义）。
* `pojoFields: Array[String]`指定映射到CSV字段的POJO的字段。CSV字段的解析器将根据POJO字段的类型和顺序自动初始化。
* `parseQuotedStrings: Character`启用带引号的字符串解析。如果字符串字段的第一个字符是引号字符（_未_剪裁前导或拖尾空格），则字符串将被解析为带引号的字符串。引用字符串中的字段分隔符将被忽略。如果带引号的字符串字段的最后一个字符不是引号字符，则引用的字符串解析将失败。如果启用了带引号的字符串解析并且该字段的第一个字符_不是_引用字符串，则该字符串将被解析为不带引号的字符串。默认情况下，禁用带引号的字符串解析。
* `ignoreComments: String`指定注释前缀。所有以指定注释前缀开头的行都不会被解析和忽略。默认情况下，不会忽略任何行。
* `lenient: Boolean`启用宽松解析，即忽略无法正确解析的行。默认情况下，禁用宽松解析，无效行引发异常。
* `ignoreFirstLine: Boolean`配置InputFormat以忽略输入文件的第一行。默认情况下，不会忽略任何行。

**递归遍历输入路径目录**

对于基于文件的输入，当输入路径是目录时，默认情况下不会枚举嵌套文件。相反，只读取基目录中的文件，而忽略嵌套文件。可以通过`recursive.file.enumeration`配置参数启用嵌套文件的递归枚举，如下例所示。

```scala
// enable recursive enumeration of nested input files
val env  = ExecutionEnvironment.getExecutionEnvironment

// create a configuration object
val parameters = new Configuration

// set the recursive enumeration parameter
parameters.setBoolean("recursive.file.enumeration", true)

// pass the configuration to the data source
env.readTextFile("file:///path/with.nested/files").withParameters(parameters)
```
{% endtab %}
{% endtabs %}

### 读取压缩文件

如果输入文件被标记为适当的文件扩展名，Flink目前支持对这些文件进行透明的解压缩。特别是，这意味着不需要进一步配置输入格式，而且任何`FileInputFormat`都支持压缩，包括自定义输入格式。请注意，压缩文件可能无法并行读取，从而影响作业的可伸缩性。

下表列出了当前支持的压缩方法。

| 压缩方法 | 文件扩展名 | 可并行 |
| :--- | :--- | :--- |
| **DEFLATE** | `.deflate` | 没有 |
| **GZip** | `.gz`， `.gzip` | 没有 |
| **bzip2** | `.bz2` | 没有 |
| **XZ** | `.xz` | 没有 |

## 数据接收

{% tabs %}
{% tab title="Java" %}
  
数据接收器使用DataSet并用于存储或返回它们。使用[OutputFormat](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/io/OutputFormat.java)描述数据接收器操作 。Flink带有各种内置输出格式，这些格式封装在DataSet上的操作后面：

* `writeAsText()`/ `TextOutputFormat`- 按字符串顺序写入元素。通过调用每个元素的_`toString()`_方法获得字符串。
* `writeAsFormattedText()`/ `TextOutputFormat`- 按字符串顺序编写元素。通过为每个元素调用用户定义的_`format()`_方法来获取字符串。
* `writeAsCsv(...)`/ `CsvOutputFormat`- 将元组写为逗号分隔值文件。行和字段分隔符是可配置的。每个字段的值来自对象的_`toString()`_方法。
* `print()`/ `printToErr()`/ `print(String msg)`/ `printToErr(String msg)`- 在标准输出/标准错误流上打印每个元素的_`toString()`_值。可选地，可以提供`prefix (msg)`，其前缀为输出。这有助于区分不同的_打印_调用。如果并行度大于1，则输出也将以生成输出的任务的标识符为前缀。
* `write()`/ `FileOutputFormat`- 自定义文件输出的方法和基类。支持自定义对象到字节的转换。
* `output()`/ `OutputFormat`- 大多数通用输出方法，用于非基于文件的数据接收器（例如将结果存储在数据库中）。

可以将DataSet输入到多个操作。程序可以编写或打印数据集，同时对它们执行其他转换。  


**例子**

标准数据接收方法：

```java
// text data
DataSet<String> textData = // [...]

// write DataSet to a file on the local file system
textData.writeAsText("file:///my/result/on/localFS");

// write DataSet to a file on a HDFS with a namenode running at nnHost:nnPort
textData.writeAsText("hdfs://nnHost:nnPort/my/result/on/localFS");

// write DataSet to a file and overwrite the file if it exists
textData.writeAsText("file:///my/result/on/localFS", WriteMode.OVERWRITE);

// tuples as lines with pipe as the separator "a|b|c"
DataSet<Tuple3<String, Integer, Double>> values = // [...]
values.writeAsCsv("file:///path/to/the/result/file", "\n", "|");

// this writes tuples in the text formatting "(a, b, c)", rather than as CSV lines
values.writeAsText("file:///path/to/the/result/file");

// this writes values as strings using a user-defined TextFormatter object
values.writeAsFormattedText("file:///path/to/the/result/file",
    new TextFormatter<Tuple2<Integer, Integer>>() {
        public String format (Tuple2<Integer, Integer> value) {
            return value.f1 + " - " + value.f0;
        }
    });
```

使用自定义输出格式：

```java
DataSet<Tuple3<String, Integer, Double>> myResult = [...]

// write Tuple DataSet to a relational database
myResult.output(
    // build and configure OutputFormat
    JDBCOutputFormat.buildJDBCOutputFormat()
                    .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")
                    .setDBUrl("jdbc:derby:memory:persons")
                    .setQuery("insert into persons (name, age, height) values (?,?,?)")
                    .finish()
    );
```

**本地排序输出**

可以使用[元组字段位置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-for-tuples)或[字段表达式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-using-field-expressions)以指定顺序在指定字段上对数据接收器的输出进行本地排序。这适用于每种输出格式。

以下示例展示如何使用此功能：

```java
DataSet<Tuple3<Integer, String, Double>> tData = // [...]
DataSet<Tuple2<BookPojo, Double>> pData = // [...]
DataSet<String> sData = // [...]

// sort output on String field in ascending order
tData.sortPartition(1, Order.ASCENDING).print();

// sort output on Double field in descending and Integer field in ascending order
tData.sortPartition(2, Order.DESCENDING).sortPartition(0, Order.ASCENDING).print();

// sort output on the "author" field of nested BookPojo in descending order
pData.sortPartition("f0.author", Order.DESCENDING).writeAsText(...);

// sort output on the full tuple in ascending order
tData.sortPartition("*", Order.ASCENDING).writeAsCsv(...);

// sort atomic type (String) output in descending order
sData.sortPartition("*", Order.DESCENDING).writeAsText(...);
```

尚不支持全局排序的输出。
{% endtab %}

{% tab title="Scala" %}
  
数据接收器使用DataSet并用于存储或返回它们。使用[OutputFormat](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/io/OutputFormat.java)描述数据接收器操作 。Flink带有各种内置输出格式，这些格式封装在DataSet上的操作后面：

* `writeAsText()`/ `TextOutputFormat`- 按字符串顺序写入元素。通过调用每个元素的_`toString()`_方法获得字符串。
* `writeAsCsv(...)`/ `CsvOutputFormat`- 将元组写为逗号分隔值文件。行和字段分隔符是可配置的。每个字段的值来自对象的_`toString()`_方法。
* `print()`/ `printToErr()`- 在标准输出/标准错误流上打印每个元素的_`toString()`_值。
* `write()`/ `FileOutputFormat`- 自定义文件输出的方法和基类。支持自定义对象到字节的转换。
* `output()`/ `OutputFormat`- 大多数通用输出方法，用于非基于文件的数据接收器（例如将结果存储在数据库中）。

可以将DataSet输入到多个操作。程序可以编写或打印数据集，同时对它们执行其他转换。

**例子**

标准数据接收方法：

```scala
// text data
val textData: DataSet[String] = // [...]

// write DataSet to a file on the local file system
textData.writeAsText("file:///my/result/on/localFS")

// write DataSet to a file on a HDFS with a namenode running at nnHost:nnPort
textData.writeAsText("hdfs://nnHost:nnPort/my/result/on/localFS")

// write DataSet to a file and overwrite the file if it exists
textData.writeAsText("file:///my/result/on/localFS", WriteMode.OVERWRITE)

// tuples as lines with pipe as the separator "a|b|c"
val values: DataSet[(String, Int, Double)] = // [...]
values.writeAsCsv("file:///path/to/the/result/file", "\n", "|")

// this writes tuples in the text formatting "(a, b, c)", rather than as CSV lines
values.writeAsText("file:///path/to/the/result/file")

// this writes values as strings using a user-defined formatting
values map { tuple => tuple._1 + " - " + tuple._2 }
  .writeAsText("file:///path/to/the/result/file")
```

**本地排序输出**

可以使用[元组字段位置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-for-tuples)或[字段表达式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-using-field-expressions)以指定顺序在指定字段上对数据接收器的输出进行本地排序。这适用于每种输出格式。

以下示例展示如何使用此功能：

```scala
val tData: DataSet[(Int, String, Double)] = // [...]
val pData: DataSet[(BookPojo, Double)] = // [...]
val sData: DataSet[String] = // [...]

// sort output on String field in ascending order
tData.sortPartition(1, Order.ASCENDING).print()

// sort output on Double field in descending and Int field in ascending order
tData.sortPartition(2, Order.DESCENDING).sortPartition(0, Order.ASCENDING).print()

// sort output on the "author" field of nested BookPojo in descending order
pData.sortPartition("_1.author", Order.DESCENDING).writeAsText(...)

// sort output on the full tuple in ascending order
tData.sortPartition("_", Order.ASCENDING).writeAsCsv(...)

// sort atomic type (String) output in descending order
sData.sortPartition("_", Order.DESCENDING).writeAsText(...)
```

尚不支持全局排序的输出。
{% endtab %}
{% endtabs %}

## 迭代运算符

迭代在Flink程序中实现循环。迭代运算符封装程序的一部分并重复执行它，将一次迭代的结果（部分解）反馈到下一次迭代中。Flink中有两种类型的迭代：**BulkIteration**和 **DeltaIteration**。

本节提供有关如何使用这两个运算符的快速示例。查看“ [迭代简介”](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/iterations.html)页面以获取更详细的介绍。

{% tabs %}
{% tab title="Java" %}
**批量迭代**

若要创建`BulkIteration`，请调用**DataSet**的`iterate(int)`方法，迭代应该从该方法开始。这将返回一个`IterativeDataSet`，可以使用常规操作符对其进行转换。迭代调用的单个参数指定迭代的最大次数。

要指定迭代的结束，请调用`IterativeDataSet`上的`closeWith(DataSet)`方法来指定应该将哪个转换反馈给下一个迭代。你可以选择使用`closeWith(DataSet, DataSet)`指定终止条件，如果该数据集为空，则使用它计算第二个数据集并终止迭代。如果没有指定终止条件，则迭代在给定的最大迭代次数之后终止。

以下示例迭代地估计数量Pi。目标是计算落入单位圆的随机点数。在每次迭代中，挑选一个随机点。如果此点位于单位圆内，我们会增加计数。然后估计Pi作为结果计数除以迭代次数乘以4。

```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// Create initial IterativeDataSet
IterativeDataSet<Integer> initial = env.fromElements(0).iterate(10000);

DataSet<Integer> iteration = initial.map(new MapFunction<Integer, Integer>() {
    @Override
    public Integer map(Integer i) throws Exception {
        double x = Math.random();
        double y = Math.random();

        return i + ((x * x + y * y < 1) ? 1 : 0);
    }
});

// Iteratively transform the IterativeDataSet
DataSet<Integer> count = initial.closeWith(iteration);

count.map(new MapFunction<Integer, Double>() {
    @Override
    public Double map(Integer count) throws Exception {
        return count / (double) 10000 * 4;
    }
}).print();

env.execute("Iterative Pi Example");
```

还可以查看 [K-Means示例](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/clustering/KMeans.scala)，该[示例](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/clustering/KMeans.scala)使用BulkIteration对一组未标记的点进行聚类。

**增量迭代**

增量迭代利用的事实是，某些算法不会在每次迭代中更改解决方案的每个数据点。

除了在每个迭代中反馈的部分解决方案\(称为工作集\)之外，增量迭代在迭代之间维护状态\(称为解决方案集\)，可以通过增量进行更新。迭代计算的结果是最后一次迭代后的状态。有关增量迭代的基本原理的概述，请参阅迭代的介绍。

定义`DeltaIteration`类似于定义`BulkIteration`。对于delta迭代，两个数据集形成每次迭代的输入（工作集和解决方案集），并且在每次迭代中生成两个数据集作为结果（新工作集，解决方案集增量）。

要在初始解决方案集中创建一个`DeltaIteration`可以调用`iterateDelta(initialWorkset, maxiteration, key)`。阶梯函数接受两个参数:\(`solutionSet`, `workset`\)，并且必须返回两个值:\(`solutionSetDelta`, `newWorkset`\)。

下面是增量迭代的语法示例

```java
// read the initial data sets
DataSet<Tuple2<Long, Double>> initialSolutionSet = // [...]

DataSet<Tuple2<Long, Double>> initialDeltaSet = // [...]

int maxIterations = 100;
int keyPosition = 0;

DeltaIteration<Tuple2<Long, Double>, Tuple2<Long, Double>> iteration = initialSolutionSet
    .iterateDelta(initialDeltaSet, maxIterations, keyPosition);

DataSet<Tuple2<Long, Double>> candidateUpdates = iteration.getWorkset()
    .groupBy(1)
    .reduceGroup(new ComputeCandidateChanges());

DataSet<Tuple2<Long, Double>> deltas = candidateUpdates
    .join(iteration.getSolutionSet())
    .where(0)
    .equalTo(0)
    .with(new CompareChangesToCurrent());

DataSet<Tuple2<Long, Double>> nextWorkset = deltas
    .filter(new FilterByThreshold());

iteration.closeWith(deltas, nextWorkset)
	.writeAsCsv(outputPath);
```
{% endtab %}

{% tab title="Scala" %}
**批量迭代**

要创建`BulkIteration`调用**DataSet**的`iterate(int)`方法，迭代应该从这个方法开始，并指定一个阶梯函数。阶梯函数获取当前迭代的输入数据集，并且必须返回一个新的数据集。迭代调用的参数是要停止的最大迭代次数。

还有`iteratewithterminate (int)`函数，它接受一个阶梯函数，该函数返回两个数据集:迭代阶梯的结果和一个终止条件。一旦终止标准数据集为空，迭代就会停止。

以下示例迭代地估计数量Pi。目标是计算落入单位圆的随机点数。在每次迭代中，挑选一个随机点。如果此点位于单位圆内，我们会增加计数。然后估计Pi作为结果计数除以迭代次数乘以4。

```scala
val env = ExecutionEnvironment.getExecutionEnvironment()

// Create initial DataSet
val initial = env.fromElements(0)

val count = initial.iterate(10000) { iterationInput: DataSet[Int] =>
  val result = iterationInput.map { i =>
    val x = Math.random()
    val y = Math.random()
    i + (if (x * x + y * y < 1) 1 else 0)
  }
  result
}

val result = count map { c => c / 10000.0 * 4 }

result.print()

env.execute("Iterative Pi Example")
```

还可以查看 [K-Means示例](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/clustering/KMeans.scala)，该[示例](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/clustering/KMeans.scala)使用BulkIteration对一组未标记的点进行聚类。

**增量迭代**

增量迭代利用的事实是，某些算法不会在每次迭代中更改解决方案的每个数据点。

除了在每个迭代中反馈的部分解决方案\(称为工作集\)之外，增量迭代在迭代之间维护状态\(称为解决方案集\)，可以通过增量进行更新。迭代计算的结果是最后一次迭代后的状态。有关增量迭代的基本原理的概述，请参阅迭代的介绍。

定义`DeltaIteration`类似于定义`BulkIteration`。对于delta迭代，两个数据集形成每次迭代的输入（工作集和解决方案集），并且在每次迭代中生成两个数据集作为结果（新工作集，解决方案集增量）。

要在初始解决方案集中创建一个`DeltaIteration`可以调用`iterateDelta(initialWorkset, maxiteration, key)`。阶梯函数接受两个参数:\(`solutionSet`, `workset`\)，并且必须返回两个值:\(`solutionSetDelta`, `newWorkset`\)。

下面是增量迭代的语法示例

```scala
// read the initial data sets
val initialSolutionSet: DataSet[(Long, Double)] = // [...]

val initialWorkset: DataSet[(Long, Double)] = // [...]

val maxIterations = 100
val keyPosition = 0

val result = initialSolutionSet.iterateDelta(initialWorkset, maxIterations, Array(keyPosition)) {
  (solution, workset) =>
    val candidateUpdates = workset.groupBy(1).reduceGroup(new ComputeCandidateChanges())
    val deltas = candidateUpdates.join(solution).where(0).equalTo(0)(new CompareChangesToCurrent())

    val nextWorkset = deltas.filter(new FilterByThreshold())

    (deltas, nextWorkset)
}

result.writeAsCsv(outputPath)

env.execute()
```
{% endtab %}
{% endtabs %}

## 在函数中操作数据对象

Flink的运行时以Java对象的形式与用户函数交换数据。函数从运行时接收输入对象作为方法参数，并返回输出对象作为结果。由于这些对象是由用户函数和运行时代码访问的，因此理解并遵循有关用户代码如何访问（即读取和修改）这些对象的规则非常重要。

用户函数可以作为常规方法参数\(如`MapFunction`\)或通过Iterable参数\(如`GroupReduceFunction`\)从Flink的运行时接收对象。我们将运行时传递给用户函数的对象称为输入对象。用户函数可以将对象作为方法返回值\(如`MapFunction`\)或通过`Collector`\(如`FlatMapFunction`\)发送到Flink运行时。我们将用户函数向运行时发出的对象称为输出对象。

Flink的DataSet API具有两种模式，这些模式在Flink的运行时创建或重用输入对象方面有所不同。此行为会影响用户函数如何与输入和输出对象进行交互的保证和约束。以下部分定义了这些规则，并给出了编写安全用户功能代码的编码指南。

### 禁止对象重用\(DEFAULT\)

默认情况下，Flink在禁用对象重用模式下运行。此模式可确保函数始终在函数调用中接收新的输入对象。禁用对象重用模式可提供更好的保证，并且使用起来更安全。但是，它带来了一定的处理开销，可能会导致更高的Java垃圾回收活动。下表说明了用户函数如何在禁用对象重用模式下访问输入和输出对象。



<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x4FDD;&#x8BC1;&#x548C;&#x9650;&#x5236;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>&#x8BFB;&#x53D6;&#x8F93;&#x5165;&#x5BF9;&#x8C61;</b>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x65B9;&#x6CD5;&#x8C03;&#x7528;&#x4E2D;&#xFF0C;&#x5B83;&#x4FDD;&#x8BC1;&#x8F93;&#x5165;&#x5BF9;&#x8C61;&#x7684;&#x503C;&#x4E0D;&#x4F1A;&#x6539;&#x53D8;&#x3002;</p>
        <p>&#x8FD9;&#x5305;&#x62EC;&#x7531;&#x8FED;&#x4EE3;&#x5668;&#x63D0;&#x4F9B;&#x7684;&#x5BF9;&#x8C61;&#x3002;&#x4F8B;&#x5982;&#xFF0C;&#x5728;List&#x6216;</p>
        <p>Map&#x4E2D;&#x6536;&#x96C6;&#x7531;&#x8FED;&#x4EE3;&#x5668;&#x63D0;&#x4F9B;&#x7684;&#x8F93;&#x5165;&#x5BF9;&#x8C61;&#x662F;&#x5B89;&#x5168;&#x7684;&#x3002;</p>
        <p>&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x5BF9;&#x8C61;&#x53EF;&#x80FD;&#x5728;&#x65B9;&#x6CD5;&#x8C03;&#x7528;&#x4E4B;&#x540E;&#x88AB;&#x4FEE;&#x6539;&#x3002;&#x8DE8;</p>
        <p>&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8BB0;&#x4F4F;&#x5BF9;&#x8C61;&#x662F;<b>&#x4E0D;&#x5B89;&#x5168;&#x7684;</b>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x4FEE;&#x6539;&#x8F93;&#x5165;&#x5BF9;&#x8C61;</b>
      </td>
      <td style="text-align:left">&#x53EF;&#x4EE5;&#x4FEE;&#x6539;&#x8F93;&#x5165;&#x5BF9;&#x8C61;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x53D1;&#x5C04;&#x8F93;&#x5165;&#x5BF9;&#x8C61;</b>
      </td>
      <td style="text-align:left">
        <p>&#x53EF;&#x4EE5;&#x53D1;&#x51FA;&#x8F93;&#x5165;&#x5BF9;&#x8C61;&#x3002;&#x8F93;&#x5165;&#x5BF9;&#x8C61;&#x7684;&#x503C;&#x5728;&#x53D1;&#x51FA;</p>
        <p>&#x540E;&#x53EF;&#x80FD;&#x5DF2;&#x66F4;&#x6539;&#x3002;&#x5728;&#x8F93;&#x51FA;&#x5BF9;&#x8C61;&#x540E;&#xFF0C;&#x8BFB;&#x53D6;&#x5B83;&#x662F;</p>
        <p><b>&#x4E0D;&#x5B89;&#x5168;&#x7684;</b>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x8BFB;&#x53D6;&#x8F93;&#x51FA;&#x5BF9;&#x8C61;</b>
      </td>
      <td style="text-align:left">
        <p>&#x63D0;&#x4F9B;&#x7ED9;&#x6536;&#x96C6;&#x5668;&#x6216;&#x4F5C;&#x4E3A;&#x65B9;&#x6CD5;&#x7ED3;&#x679C;&#x8FD4;&#x56DE;&#x7684;&#x5BF9;&#x8C61;</p>
        <p>&#x53EF;&#x80FD;&#x5DF2;&#x66F4;&#x6539;&#x5176;&#x503C;&#x3002;&#x8BFB;&#x53D6;&#x8F93;&#x51FA;&#x5BF9;&#x8C61;&#x662F;<b>&#x4E0D;&#x5B89;&#x5168;&#x7684;</b>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x4FEE;&#x6539;&#x8F93;&#x51FA;&#x5BF9;&#x8C61;</b>
      </td>
      <td style="text-align:left">&#x53EF;&#x4EE5;&#x5728;&#x53D1;&#x5C04;&#x5BF9;&#x8C61;&#x540E;&#x5BF9;&#x5176;&#x8FDB;&#x884C;&#x4FEE;&#x6539;&#x5E76;&#x518D;&#x6B21;&#x53D1;&#x5C04;&#x3002;</td>
    </tr>
  </tbody>
</table>

**禁用对象重用（默认）模式的编码指南：**

* 不要在方法调用之间记住和读取输入对象。 
* 发出对象后不要读取它们。

### 启用对象重用

在对象重用启用模式下，Flink的运行时最小化对象实例化的数量。这可以提高性能并可以减少Java垃圾收集压力。通过调用`ExecutionConfig.enableObjectReuse()`激活对象重用启用模式。下表说明了用户函数如何在对象重用启用模式下访问输入和输出对象。



| 操作 | 保证和限制 |
| :--- | :--- |
| **读取作为常规方法参数接收的输入对象** | 作为常规方法参数接收的输入对象不会在函数调用中修改。在离开方法调用后，可以修改对象。在函数调用中记住对象是**不安全的**。 |
| **读取从Iterable参数接收的输入对象** | 从Iterable接收的输入对象只在调用next\(\)方法之前有效。可迭代器或迭代器可以多次服务相同的对象实例。记住从Iterable接收的输入对象是**不安全**的，例如，将它们放入List或Map中。 |
| **修改输入对象** | 除了MapFunction，FlatMapFunction，MapPartitionFunction，GroupReduceFunction，GroupCombineFunction，CoGroupFunction和InputFormat.next\(reuse\)的输入对象外，**不能**修改输入对象。 |
| **发射输入对象** | 除了MapFunction，FlatMapFunction，MapPartitionFunction，GroupReduceFunction，GroupCombineFunction，CoGroupFunction和InputFormat.next\(reuse\)的输入对象外， **不能**发出输入对象。 |
| **读取输出对象** | 提供给收集器或作为方法结果返回的对象可能已更改其值。读取输出对象是**不安全的**。 |
| **修改输出对象** | 您可以修改输出对象并再次发出它。 |

**启用对象重用的编码指南：**

* 不要在方法调用之间记住和读取输入对象。 
* 发出对象后不要读取它们。 
* 不要记住从迭代器接收到的输入对象。 
* 不要在方法调用之间记住和读取输入对象。 
* 不要修改或发出输入对象，但`MapFunction`、`FlatMapFunction`、`MapPartitionFunction`、`GroupReduceFunction`、`GroupCombineFunction`、`CoGroupFunction`和`InputFormat.next(reuse)`的输入对象除外。 
* 为了减少对象实例化，始终可以发出一个专用的输出对象，该输出对象会被反复修改，但不会被读取。

## 调试

在对分布式集群中的大型数据集运行数据分析程序之前，最好确保实现的算法按预期工作。因此，实施数据分析程序通常是检查结果，调试和改进的增量过程。

Flink提供了一些很好的功能，通过支持IDE内的本地调试，测试数据的注入和结果数据的收集，显着简化了数据分析程序的开发过程。本节提供了一些如何简化Flink程序开发的提示。

### 本地执行环境

`LocalEnvironment`在它创建的JVM进程中启动Flink系统。如果从IDE启动`LocalEnvironment`，可以在代码中设置断点并轻松调试程序。

创建并使用`LocalEnvironment`如下所示:

{% tabs %}
{% tab title="Java" %}
```java
final ExecutionEnvironment env = ExecutionEnvironment.createLocalEnvironment();

DataSet<String> lines = env.readTextFile(pathToTextFile);
// build your program

env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.createLocalEnvironment()

val lines = env.readTextFile(pathToTextFile)
// build your program

env.execute()
```
{% endtab %}
{% endtabs %}

### 收集数据源和接收器

通过创建输入文件和读取输出文件来完成分析程序的输入并检查其输出是很麻烦的。Flink具有特殊的数据源和接收器，由Java集合支持以简化测试。一旦程序经过测试，源和接收器可以很容易地被读取/写入外部数据存储（如HDFS）的源和接收器替换。

集合数据源可以如下使用：

{% tabs %}
{% tab title="Java" %}
```java
final ExecutionEnvironment env = ExecutionEnvironment.createLocalEnvironment();

// Create a DataSet from a list of elements
DataSet<Integer> myInts = env.fromElements(1, 2, 3, 4, 5);

// Create a DataSet from any Java collection
List<Tuple2<String, Integer>> data = ...
DataSet<Tuple2<String, Integer>> myTuples = env.fromCollection(data);

// Create a DataSet from an Iterator
Iterator<Long> longIt = ...
DataSet<Long> myLongs = env.fromCollection(longIt, Long.class);
```

集合数据接收器指定如下：

```text
DataSet<Tuple2<String, Integer>> myResult = ...

List<Tuple2<String, Integer>> outData = new ArrayList<Tuple2<String, Integer>>();
myResult.output(new LocalCollectionOutputFormat(outData));
```

**注意：**目前，集合数据接收器仅限于本地执行，作为调试工具。
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.createLocalEnvironment()

// Create a DataSet from a list of elements
val myInts = env.fromElements(1, 2, 3, 4, 5)

// Create a DataSet from any Collection
val data: Seq[(String, Int)] = ...
val myTuples = env.fromCollection(data)

// Create a DataSet from an Iterator
val longIt: Iterator[Long] = ...
val myLongs = env.fromCollection(longIt)
```
{% endtab %}
{% endtabs %}

**注意：**目前，集合数据源要求实现数据类型和迭代器 `Serializable`。此外，收集数据源不能并行执行（并行度= 1）。

## 语义注释

语义注释可用于提供有关函数行为的Flink提示。它们告诉系统函数的输入的哪些字段是函数读取和计算的，以及函数未修改的哪些字段是从输入转发到输出的。语义注释是一种加速执行的强大方法，因为它们允许系统考虑跨多个操作重用排序顺序或分区。使用语义注释最终可能使程序避免不必要的数据变换或不必要的排序，并显著提高程序的性能。

**注意：**语义注释的使用是可选的。但是，在提供语义注释时保守是绝对至关重要的！不正确的语义注释会导致Flink对您的程序做出错误的假设，并最终可能导致错误的结果。如果操作符的行为不明确可预测，则不应提供注释。请仔细阅读文档。

目前支持以下语义注释。

### **转发字段注释**

转发字段信息声明输入字段，这些输入字段未被修改，由函数转发到相同位置或输出中的另一个位置。优化器使用此信息来推断函数是否保留了诸如排序或分区之类的数据属性。对于输入元件，诸如一组操作的函数`GroupReduce`，`GroupCombine`，`CoGroup`，和`MapPartition`，被定义为转发字段的所有字段必须始终共同从相同的输入元件转发。由分组函数发出的每个元素的转发字段可以源自函数输入组的不同元素。

使用[字段表达式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-using-field-expressions)指定字段转发信息。转发到输出中相同位置的字段可以通过其位置指定。指定的位置必须对输入和输出数据类型有效，并且具有相同的类型。例如，String `"f2"`声明Java输入元组的第三个字段始终等于输出元组中的第三个字段。

通过将输入中的源字段和输出中的目标字段指定为字段表达式来声明未修改的字段转发到输出中的另一个位置。String `"f0->f2"`表示Java输入元组的第一个字段未更改，复制到Java输出元组的第三个字段。通配符表达式`*`可用于指代整个输入或输出类型，即`"f0->*"`表示函数的输出始终等于其Java输入元组的第一个字段。

可以在一个字符串中声明多个转发字段，方法是使用分号将它们分隔为“f0;f2 - &gt; f1;f3-&gt;f2”或在单独的字符串“f0”，“f2-&gt;f1”，“f3-&gt;f2”。在指定转发字段时，并不要求声明所有转发字段，但所有声明必须正确。

可以通过在函数类定义上附加Java注释或在调用DataSet上的函数后将它们作为操作符参数传递来声明转发的字段信息，如下所示。

**函数类注释**

* `@ForwardedFields` 用于单输入功能，例如Map和Reduce。
* `@ForwardedFieldsFirst` 用于第一次输入具有两个输入的函数，例如Join和CoGroup。
* `@ForwardedFieldsSecond` 用于具有两个输入的函数的第二个输入，例如Join和CoGroup。

**操作符参数**

使用以下注释将未转发的字段信息指定为函数类注释：

* `data.map(myMapFnc).withForwardedFields()` 用于单输入功能，例如Map和Reduce。
* `data1.join(data2).where().equalTo().with(myJoinFnc).withForwardFieldsFirst()` 对于具有两个输入的函数的第一个输入，例如Join和CoGroup。
* `data1.join(data2).where().equalTo().with(myJoinFnc).withForwardFieldsSecond()` 对于具有两个输入的函数的第二个输入，例如Join和CoGroup。

请注意，无法覆盖由操作符参数指定为类注释的字段转发信息。

**例**

以下示例显示如何使用函数类注释声明转发的字段信息：

{% tabs %}
{% tab title="Java" %}
```java
@ForwardedFields("f0->f2")
public class MyMap implements
              MapFunction<Tuple2<Integer, Integer>, Tuple3<String, Integer, Integer>> {
  @Override
  public Tuple3<String, Integer, Integer> map(Tuple2<Integer, Integer> val) {
    return new Tuple3<String, Integer, Integer>("foo", val.f1 / 2, val.f0);
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
@ForwardedFields("_1->_3")
class MyMap extends MapFunction[(Int, Int), (String, Int, Int)]{
   def map(value: (Int, Int)): (String, Int, Int) = {
    return ("foo", value._2 / 2, value._1)
  }
}
```
{% endtab %}
{% endtabs %}

### **非转发字段**

非转发字段信息声明在函数输出的相同位置上未保存的所有字段。所有其他字段的值都被认为保存在输出中的相同位置。因此，非转发字段信息与转发字段信息是相反的。分组操作符\(如`GroupReduce`、`GroupCombine`、`CoGroup`和`MapPartition`\)的非转发字段信息必须满足与转发字段信息相同的要求。

**重要信息**：非转发字段信息的规范是可选的。但如果使用， **全部！**必须指定非转发字段，因为所有其他字段都被视为在适当位置转发。将转发字段声明为非转发是安全的。

非转发字段指定为[字段表达式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#define-keys-using-field-expressions)列表。列表既可以作为一个字段表达式由分号分隔的单个字符串给出，也可以作为多个字符串给出。例如“f1;f3”和“f1”、“f3”声明Java元组的第二个和第四个字段没有保存在适当的位置，所有其他字段都保存在适当的位置。非转发字段信息只能指定具有相同输入和输出类型的函数。

使用以下注释将未转发的字段信息指定为函数类注释：

* `@NonForwardedFields` 用于单输入功能，例如Map和Reduce。
* `@NonForwardedFieldsFirst` 对于具有两个输入的函数的第一个输入，例如Join和CoGroup。
* `@NonForwardedFieldsSecond` 对于具有两个输入的函数的第二个输入，例如Join和CoGroup。

**例**

以下示例显示如何声明未转发的字段信息：

{% tabs %}
{% tab title="Java" %}
```java
@NonForwardedFields("f1") // second field is not forwarded
public class MyMap implements
              MapFunction<Tuple2<Integer, Integer>, Tuple2<Integer, Integer>> {
  @Override
  public Tuple2<Integer, Integer> map(Tuple2<Integer, Integer> val) {
    return new Tuple2<Integer, Integer>(val.f0, val.f1 / 2);
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
@NonForwardedFields("_2") // second field is not forwarded
class MyMap extends MapFunction[(Int, Int), (Int, Int)]{
  def map(value: (Int, Int)): (Int, Int) = {
    return (value._1, value._2 / 2)
  }
}
```
{% endtab %}
{% endtabs %}

### **读取字段**

读取字段信息声明一个函数访问和计算的所有字段，即，该函数用于计算其结果的所有字段。例如，在指定读字段信息时，在条件语句中计算或用于计算的字段必须标记为read。只将未修改的字段转发到输出而不计算它们的值，或者根本不访问的字段不被认为是读取的。

**重要提示**:读取字段信息的规范是可选的。然而，如果使用，**ALL!**必须指定读字段。将非读字段声明为read是安全的。

Read字段被指定为字段表达式的列表。列表既可以作为一个字段表达式由分号分隔的单个字符串给出，也可以作为多个字符串给出。例如“f1;f3”和“f1”，“f3”声明Java元组的第二个和第四个字段是由函数读取和计算的。

使用以下注释将读取字段信息指定为函数类注释：

* `@ReadFields` 用于单输入功能，例如Map和Reduce。
* `@ReadFieldsFirst` 对于具有两个输入的函数的第一个输入，例如Join和CoGroup。
* `@ReadFieldsSecond` 对于具有两个输入的函数的第二个输入，例如Join和CoGroup。

**例**

以下示例显示如何声明读取字段信息：

{% tabs %}
{% tab title="Java" %}
```java
@ReadFields("f0; f3") // f0 and f3 are read and evaluated by the function.
public class MyMap implements
              MapFunction<Tuple4<Integer, Integer, Integer, Integer>,
                          Tuple2<Integer, Integer>> {
  @Override
  public Tuple2<Integer, Integer> map(Tuple4<Integer, Integer, Integer, Integer> val) {
    if(val.f0 == 42) {
      return new Tuple2<Integer, Integer>(val.f0, val.f1);
    } else {
      return new Tuple2<Integer, Integer>(val.f3+10, val.f1);
    }
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
@ReadFields("_1; _4") // _1 and _4 are read and evaluated by the function.
class MyMap extends MapFunction[(Int, Int, Int, Int), (Int, Int)]{
   def map(value: (Int, Int, Int, Int)): (Int, Int) = {
    if (value._1 == 42) {
      return (value._1, value._2)
    } else {
      return (value._4 + 10, value._2)
    }
  }
}
```
{% endtab %}
{% endtabs %}

## 广播变量

除了常规的操作输入之外，广播变量还允许将数据集用于操作的所有并行实例。这对辅助数据集或数据相关参数化非常有用。之后，操作符可以将数据集作为集合访问。

* **广播**：广播集通过名称注册`withBroadcastSet(DataSet, String)`，和
* **访问**：可通过`getRuntimeContext().getBroadcastVariable(String)`目标操作符访问。

{% tabs %}
{% tab title="Java" %}
```java
// 1. The DataSet to be broadcast
DataSet<Integer> toBroadcast = env.fromElements(1, 2, 3);

DataSet<String> data = env.fromElements("a", "b");

data.map(new RichMapFunction<String, String>() {
    @Override
    public void open(Configuration parameters) throws Exception {
      // 3. Access the broadcast DataSet as a Collection
      Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("broadcastSetName");
    }


    @Override
    public String map(String value) throws Exception {
        ...
    }
}).withBroadcastSet(toBroadcast, "broadcastSetName"); // 2. Broadcast the DataSet
```
{% endtab %}

{% tab title="Scala" %}
```scala
// 1. The DataSet to be broadcast
val toBroadcast = env.fromElements(1, 2, 3)

val data = env.fromElements("a", "b")

data.map(new RichMapFunction[String, String]() {
    var broadcastSet: Traversable[String] = null

    override def open(config: Configuration): Unit = {
      // 3. Access the broadcast DataSet as a Collection
      broadcastSet = getRuntimeContext().getBroadcastVariable[String]("broadcastSetName").asScala
    }

    def map(in: String): String = {
        ...
    }
}).withBroadcastSet(toBroadcast, "broadcastSetName") // 2. Broadcast the DataSet
```
{% endtab %}
{% endtabs %}

`broadcastSetName`注册和访问广播数据集时，请确保名称（在前面的示例中）匹配。有关完整的示例程序，请查看 [KMeans算法](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/clustering/KMeans.scala)。

**注意**：由于广播变量的内容保存在每个节点的内存中，因此不应该变得太大。对于像标量值这样的简单事物，您可以简单地将参数作为函数闭包的一部分，或者使用该`withParameters(...)`方法传递配置。

## 分布式缓存

Flink提供了类似于Apache Hadoop的分布式缓存，可以让并行用户函数实例本地访问文件。此功能可用于共享包含静态外部数据\(如字典或机器学习的回归模型\)的文件。

缓存的工作方式如下。程序将[本地或远程文件系统\(如HDFS或S3\)](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/connectors.html#reading-from-file-systems)的文件或目录作为缓存文件注册到`ExecutionEnvironment`中的特定名称下。当程序执行时，Flink自动将文件或目录复制到所有worker的本地文件系统。用户函数可以查找指定名称下的文件或目录，并从**worker**的本地文件系统访问。

分布式缓存使用如下：

{% tabs %}
{% tab title="Java" %}
注册中的文件或目录`ExecutionEnvironment`。

```text
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// register a file from HDFS
env.registerCachedFile("hdfs:///path/to/your/file", "hdfsFile")

// register a local executable file (script, executable, ...)
env.registerCachedFile("file:///path/to/exec/file", "localExecFile", true)

// define your program and execute
...
DataSet<String> input = ...
DataSet<Integer> result = input.map(new MyMapper());
...
env.execute();
```

访问用户函数中的缓存文件或目录（此处为a `MapFunction`）。该函数必须扩展[RichFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#rich-functions)类，因为它需要访问它`RuntimeContext`。

```text
// extend a RichFunction to have access to the RuntimeContext
public final class MyMapper extends RichMapFunction<String, Integer> {

    @Override
    public void open(Configuration config) {

      // access cached file via RuntimeContext and DistributedCache
      File myFile = getRuntimeContext().getDistributedCache().getFile("hdfsFile");
      // read the file (or navigate the directory)
      ...
    }

    @Override
    public Integer map(String value) throws Exception {
      // use content of cached file
      ...
    }
}
```
{% endtab %}

{% tab title="Scala" %}
注册中的文件或目录`ExecutionEnvironment`。

```text
val env = ExecutionEnvironment.getExecutionEnvironment

// register a file from HDFS
env.registerCachedFile("hdfs:///path/to/your/file", "hdfsFile")

// register a local executable file (script, executable, ...)
env.registerCachedFile("file:///path/to/exec/file", "localExecFile", true)

// define your program and execute
...
val input: DataSet[String] = ...
val result: DataSet[Integer] = input.map(new MyMapper())
...
env.execute()
```

访问用户函数中的缓存文件（此处为a `MapFunction`）。该函数必须扩展[RichFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#rich-functions)类，因为它需要访问它`RuntimeContext`。

```text
// extend a RichFunction to have access to the RuntimeContext
class MyMapper extends RichMapFunction[String, Int] {

  override def open(config: Configuration): Unit = {

    // access cached file via RuntimeContext and DistributedCache
    val myFile: File = getRuntimeContext.getDistributedCache.getFile("hdfsFile")
    // read the file (or navigate the directory)
    ...
  }

  override def map(value: String): Int = {
    // use content of cached file
    ...
  }
}
```
{% endtab %}
{% endtabs %}

## 将参数传递给函数

可以使用构造函数或`withParameters(Configuration)`方法将参数传递给函数。参数被序列化为函数对象的一部分并传送到所有并行任务实例。

可查看[有关如何将命令行参数传递给函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/best_practices.html#parsing-command-line-arguments-and-passing-them-around-in-your-flink-application)的[最佳实践指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/best_practices.html#parsing-command-line-arguments-and-passing-them-around-in-your-flink-application)。

### **通过构造函数**

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Integer> toFilter = env.fromElements(1, 2, 3);

toFilter.filter(new MyFilter(2));

private static class MyFilter implements FilterFunction<Integer> {

  private final int limit;

  public MyFilter(int limit) {
    this.limit = limit;
  }

  @Override
  public boolean filter(Integer value) throws Exception {
    return value > limit;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
val toFilter = env.fromElements(1, 2, 3)

toFilter.filter(new MyFilter(2))

class MyFilter(limit: Int) extends FilterFunction[Int] {
  override def filter(value: Int): Boolean = {
    value > limit
  }
}
```
{% endtab %}
{% endtabs %}

### **通过 withParameters\(Configuration\)**

此方法将Configuration对象作为参数，将其传递给[rich函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#rich-functions)的`open()` 方法。Configuration对象是从String键到不同值类型的Map。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Integer> toFilter = env.fromElements(1, 2, 3);

Configuration config = new Configuration();
config.setInteger("limit", 2);

toFilter.filter(new RichFilterFunction<Integer>() {
    private int limit;

    @Override
    public void open(Configuration parameters) throws Exception {
      limit = parameters.getInteger("limit", 0);
    }

    @Override
    public boolean filter(Integer value) throws Exception {
      return value > limit;
    }
}).withParameters(config);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val toFilter = env.fromElements(1, 2, 3)

val c = new Configuration()
c.setInteger("limit", 2)

toFilter.filter(new RichFilterFunction[Int]() {
    var limit = 0

    override def open(config: Configuration): Unit = {
      limit = config.getInteger("limit", 0)
    }

    def filter(in: Int): Boolean = {
        in > limit
    }
}).withParameters(c)
```
{% endtab %}
{% endtabs %}

### **全局通过 ExecutionConfig**

Flink还允许将自定义配置值传递到`ExecutionConfig`环境的接口。由于执行配置可在所有（丰富）用户功能中访问，因此自定义配置将在所有功能中全局可用。

**设置自定义全局配置**

{% tabs %}
{% tab title="Java" %}
```java
Configuration conf = new Configuration();
conf.setString("mykey","myvalue");
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().setGlobalJobParameters(conf);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment
val conf = new Configuration()
conf.setString("mykey", "myvalue")
env.getConfig.setGlobalJobParameters(conf)
```
{% endtab %}
{% endtabs %}

请注意，您还可以传递一个扩展`ExecutionConfig`的自定义类。类作为执行配置的全局作业参数。该接口允许实现`Map<String, String> toMap()` 方法，该方法将依次显示web前端配置中的值。

**从全局配置中访问值**

全局作业参数中的对象可在系统中的许多位置访问。实现`RichFunction`接口的所有用户函数都可以通过运行时上下文访问。

```java
public static final class Tokenizer extends RichFlatMapFunction<String, Tuple2<String, Integer>> {

    private String mykey;
    @Override
    public void open(Configuration parameters) throws Exception {
      super.open(parameters);
      ExecutionConfig.GlobalJobParameters globalParams = getRuntimeContext().getExecutionConfig().getGlobalJobParameters();
      Configuration globConf = (Configuration) globalParams;
      mykey = globConf.getString("mykey", null);
    }
    // ... more here ...
```

