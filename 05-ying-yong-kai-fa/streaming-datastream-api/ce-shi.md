---
description: 本页简要讨论如何在IDE或本地环境中测试Flink应用程序。
---

# 测试

## 单元测试

通常，我们可以假设Flink在用户定义的函数之外产生正确的结果。因此，建议使用单元测试尽可能多地测试包含主要业务逻辑的函数类。

例如，如果实现以下`ReduceFunction`:

{% tabs %}
{% tab title="Java" %}
```java
public class SumReduce implements ReduceFunction<Long> {

    @Override
    public Long reduce(Long value1, Long value2) throws Exception {
        return value1 + value2;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class SumReduce extends ReduceFunction[Long] {

    override def reduce(value1: java.lang.Long, value2: java.lang.Long): java.lang.Long = {
        value1 + value2
    }
}
```
{% endtab %}
{% endtabs %}

通过传递适当的参数和验证输出，可以很容易地用自己喜欢的框架对其进行单元测试:

{% tabs %}
{% tab title="Java" %}
```java
public class SumReduceTest {

    @Test
    public void testSum() throws Exception {
        // instantiate your function
        SumReduce sumReduce = new SumReduce();

        // call the methods that you have implemented
        assertEquals(42L, sumReduce.reduce(40L, 2L));
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class SumReduceTest extends FlatSpec with Matchers {

    "SumReduce" should "add values" in {
        // instantiate your function
        val sumReduce: SumReduce = new SumReduce()

        // call the methods that you have implemented
        sumReduce.reduce(40L, 2L) should be (42L)
    }
}
```
{% endtab %}
{% endtabs %}

## 集成测试

为了端到端测试Flink流管道，您还可以编写针对本地Flink MiniCluster执行的集成测试。

为此，添加测试依赖项`flink-test-utils`：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-test-utils_2.11</artifactId>
  <version>1.7.0</version>
</dependency>
```

例如，如果要测试以下`MapFunction`的内容：

{% tabs %}
{% tab title="Java" %}
```java
public class MultiplyByTwo implements MapFunction<Long, Long> {

    @Override
    public Long map(Long value) throws Exception {
        return value * 2;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MultiplyByTwo extends MapFunction[Long, Long] {

    override def map(value: Long): Long = {
        value * 2
    }
}
```
{% endtab %}
{% endtabs %}

可以编写以下集成测试：

{% tabs %}
{% tab title="Java" %}
```java
public class ExampleIntegrationTest extends AbstractTestBase {

    @Test
    public void testMultiply() throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // configure your test environment
        env.setParallelism(1);

        // values are collected in a static variable
        CollectSink.values.clear();

        // create a stream of custom elements and apply transformations
        env.fromElements(1L, 21L, 22L)
                .map(new MultiplyByTwo())
                .addSink(new CollectSink());

        // execute
        env.execute();

        // verify your results
        assertEquals(Lists.newArrayList(2L, 42L, 44L), CollectSink.values);
    }

    // create a testing sink
    private static class CollectSink implements SinkFunction<Long> {

        // must be static
        public static final List<Long> values = new ArrayList<>();

        @Override
        public synchronized void invoke(Long value) throws Exception {
            values.add(value);
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class ExampleIntegrationTest extends AbstractTestBase {

    @Test
    def testMultiply(): Unit = {
        val env = StreamExecutionEnvironment.getExecutionEnvironment

        // configure your test environment
        env.setParallelism(1)

        // values are collected in a static variable
        CollectSink.values.clear()

        // create a stream of custom elements and apply transformations
        env
            .fromElements(1L, 21L, 22L)
            .map(new MultiplyByTwo())
            .addSink(new CollectSink())

        // execute
        env.execute()

        // verify your results
        assertEquals(Lists.newArrayList(2L, 42L, 44L), CollectSink.values)
    }
}    

// create a testing sink
class CollectSink extends SinkFunction[Long] {

    override def invoke(value: java.lang.Long): Unit = {
        synchronized {
            values.add(value)
        }
    }
}

object CollectSink {

    // must be static
    val values: List[Long] = new ArrayList()
}
```
{% endtab %}
{% endtabs %}

这里使用CollectSink中的静态变量是因为Flink在将所有运算符分布到集群之前将其序列化。通过静态变量与本地Flink MiniCluster实例化的操作符通信是解决这个问题的一种方法。或者，您可以使用测试Sink将数据写入临时目录中的文件。您还可以实现自己的用于发送watermarks的自定义源。

## 测试检查点和状态处理

测试状态处理的一种方法是在集成测试中启用检查点。

您可以通过`StreamExecutionEnvironment`在测试中配置来完成此操作：

{% tabs %}
{% tab title="Java" %}
```java
env.enableCheckpointing(500);
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 100));
```
{% endtab %}

{% tab title="Scala" %}
```scala
env.enableCheckpointing(500)
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 100))
```
{% endtab %}
{% endtabs %}

例如，在Flink应用程序中添加一个标识映射器操作符，该操作符每1000毫秒抛出一个异常。然而，由于操作之间的时间依赖性，编写这样的测试可能比较棘手。

另一种方法是使用Flink内部测试实用工具`AbstractStreamOperatorTestHarness`编写单元测试，该实用工具来自`Flink -streaming-java`模块。

要了解如何做到这一点的示例，请查看`org.apache.stream.runtime.operators.window.windowoperatortest`，它也在`flink-streaming-java`模块中。

请注意`AbstractStreamOperatorTestHarness`目前不是公共API的一部分，可能会发生更改。  


