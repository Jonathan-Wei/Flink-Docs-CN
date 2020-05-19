# 实验特性

本节介绍了DataStream API中的实验功能。实验功能仍在不断发展，可能会变得不稳定，不完整，或者在以后的版本中可能会发生重大变化。

## 将预分区的数据流重新解释为键控流

我们可以将预分区的数据流重新解释为键控流，以避免混排。

{% hint style="danger" %}
**警告:**重新解释的数据流**必须**已经**完全**按照Flink的keyBy在随机分配w.r.t.键组分配中对数据进行分区的方式进行了预先分区。
{% endhint %}

一个使用场景可以是两个作业之间的物化改组：第一个作业执行keyBy改组并将每个输出物化为一个分区。第二个作业的源对于每个并行实例都从第一个作业创建的相应分区中读取。现在可以将这些源重新解释为键控流，例如以应用窗口。请注意，此技巧使第二个工作尴尬地并行进行，这对于细粒度的恢复方案很有帮助。

通过`DataStreamUtils`以下方式公开此重新解释功能：

```text
	static <T, K> KeyedStream<T, K> reinterpretAsKeyedStream(
		DataStream<T> stream,
		KeySelector<T, K> keySelector,
		TypeInformation<K> typeInfo)
```

给定基本流，键选择器和类型信息，该方法从基本流创建键控流。

代码示例：

{% tabs %}
{% tab title="Java" %}
```java
   StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<Integer> source = ...
        DataStreamUtils.reinterpretAsKeyedStream(source, (in) -> in, TypeInformation.of(Integer.class))
            .timeWindow(Time.seconds(1))
            .reduce((a, b) -> a + b)
            .addSink(new DiscardingSink<>());
        env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
 val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val source = ...
    new DataStreamUtils(source).reinterpretAsKeyedStream((in) => in)
      .timeWindow(Time.seconds(1))
      .reduce((a, b) => a + b)
      .addSink(new DiscardingSink[Int])
    env.execute()
```
{% endtab %}
{% endtabs %}

