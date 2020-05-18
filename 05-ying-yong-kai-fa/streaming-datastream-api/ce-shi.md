---
description: 本页简要讨论如何在IDE或本地环境中测试Flink应用程序。
---

# 测试

测试是每个软件开发过程中不可或缺的一部分，因为Apache Flink附带了工具，可以在测试金字塔的多个级别上测试应用程序代码。

## 测试用户定义的功能

通常，可以假定Flink在用户定义的函数之外产生正确的结果。因此，建议尽可能用单元测试来测试那些包含主要业务逻辑的类。

### 无状态、无时间限制的udf单元测试

例如，让我们采用以下无状态`MapFunction`。

{% tabs %}
{% tab title="Java" %}
```java
public class IncrementMapFunction implements MapFunction<Long, Long> {

    @Override
    public Long map(Long record) throws Exception {
        return record + 1;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class IncrementMapFunction extends MapFunction[Long, Long] {

    override def map(record: Long): Long = {
        record + 1
    }
}
```
{% endtab %}
{% endtabs %}

通过传递合适的参数并验证输出，使用你喜欢的测试框架对此类功能进行单元测试非常容易。

{% tabs %}
{% tab title="Java" %}
```java
public class IncrementMapFunctionTest {

    @Test
    public void testIncrement() throws Exception {
        // instantiate your function
        IncrementMapFunction incrementer = new IncrementMapFunction();

        // call the methods that you have implemented
        assertEquals(3L, incrementer.map(2L));
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class IncrementMapFunctionTest extends FlatSpec with Matchers {

    "IncrementMapFunction" should "increment values" in {
        // instantiate your function
        val incrementer: IncrementMapFunction = new IncrementMapFunction()

        // call the methods that you have implemented
        incremeter.map(2) should be (3)
    }
}
```
{% endtab %}
{% endtabs %}

类似地，用户定义的函数使用`org.apache.flink.util.Collector`提供模拟对象而不是实际的收集器，可以很容易地测试收集器\(例如，`FlatMapFunction`或`ProcessFunction`\)。与`IncrementMapFunction`具有相同功能的FlatMapFunction可以按如下方式进行单元测试。

{% tabs %}
{% tab title="Java" %}
```java
public class IncrementFlatMapFunctionTest {

    @Test
    public void testIncrement() throws Exception {
        // instantiate your function
        IncrementFlatMapFunction incrementer = new IncrementFlatMapFunction();

        Collector<Integer> collector = mock(Collector.class);

        // call the methods that you have implemented
        incrementer.flatMap(2L, collector);

        //verify collector was called with the right output
        Mockito.verify(collector, times(1)).collect(3L);
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class IncrementFlatMapFunctionTest extends FlatSpec with MockFactory {

    "IncrementFlatMapFunction" should "increment values" in {
       // instantiate your function
      val incrementer : IncrementFlatMapFunction = new IncrementFlatMapFunction()

      val collector = mock[Collector[Integer]]

      //verify collector was called with the right output
      (collector.collect _).expects(3)

      // call the methods that you have implemented
      flattenFunction.flatMap(2, collector)
  }
}
```
{% endtab %}
{% endtabs %}

### 有状态或及时的udf和自定义操作符单元测试

测试用户定义函数的功能（利用托管状态或计时器）更加困难，因为它涉及测试用户代码与Flink运行时之间的交互。为此，Flink附带了一组所谓的测试工具，可用于测试此类用户定义的函数以及自定义运算符：

* `OneInputStreamOperatorTestHarness`（适用于的运营商`DataStreams`）
* `KeyedOneInputStreamOperatorTestHarness`（适用于的运营商`KeyedStream`）
* `TwoInputStreamOperatorTestHarness`（关于运营商`ConnectedStreams`的两个`DataStream`或多个）
* `KeyedTwoInputStreamOperatorTestHarness`（对于`ConnectedStreams`两个中`KeyedStream`的运算符）

要使用测试工具，还需要一组其他依赖项（测试作用域）。

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-test-utils_2.11</artifactId>
  <version>1.10.0</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-runtime_2.11</artifactId>
  <version>1.10.0</version>
  <scope>test</scope>
  <classifier>tests</classifier>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-java_2.11</artifactId>
  <version>1.10.0</version>
  <scope>test</scope>
  <classifier>tests</classifier>
</dependency>
```

现在，可以使用测试工具将记录和水印推送到用户定义的函数或自定义操作符中，控制处理时间，并最终对操作符的输出\(包括侧输出\)进行断言。

{% tabs %}
{% tab title="Java" %}
```java
public class StatefulFlatMapTest {
    private OneInputStreamOperatorTestHarness<Long, Long> testHarness;
    private StatefulFlatMap statefulFlatMapFunction;

    @Before
    public void setupTestHarness() throws Exception {

        //instantiate user-defined function
        statefulFlatMapFunction = new StatefulFlatMapFunction();

        // wrap user defined function into a the corresponding operator
        testHarness = new OneInputStreamOperatorTestHarness<>(new StreamFlatMap<>(statefulFlatMapFunction));

        // optionally configured the execution environment
        testHarness.getExecutionConfig().setAutoWatermarkInterval(50);

        // open the test harness (will also call open() on RichFunctions)
        testHarness.open();
    }

    @Test
    public void testingStatefulFlatMapFunction() throws Exception {

        //push (timestamped) elements into the operator (and hence user defined function)
        testHarness.processElement(2L, 100L);

        //trigger event time timers by advancing the event time of the operator with a watermark
        testHarness.processWatermark(100L);

        //trigger processing time timers by advancing the processing time of the operator directly
        testHarness.setProcessingTime(100L);

        //retrieve list of emitted records for assertions
        assertThat(testHarness.getOutput(), containsInExactlyThisOrder(3L));

        //retrieve list of records emitted to a specific side output for assertions (ProcessFunction only)
        //assertThat(testHarness.getSideOutput(new OutputTag<>("invalidRecords")), hasSize(0))
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class StatefulFlatMapFunctionTest extends FlatSpec with Matchers with BeforeAndAfter {

  private var testHarness: OneInputStreamOperatorTestHarness[Long, Long] = null
  private var statefulFlatMap: StatefulFlatMapFunction = null

  before {
    //instantiate user-defined function
    statefulFlatMap = new StatefulFlatMap

    // wrap user defined function into a the corresponding operator
    testHarness = new OneInputStreamOperatorTestHarness[Long, Long](new StreamFlatMap(statefulFlatMap))

    // optionally configured the execution environment
    testHarness.getExecutionConfig().setAutoWatermarkInterval(50);

    // open the test harness (will also call open() on RichFunctions)
    testHarness.open();
  }

  "StatefulFlatMap" should "do some fancy stuff with timers and state" in {


    //push (timestamped) elements into the operator (and hence user defined function)
    testHarness.processElement(2, 100);

    //trigger event time timers by advancing the event time of the operator with a watermark
    testHarness.processWatermark(100);

    //trigger proccesign time timers by advancing the processing time of the operator directly
    testHarness.setProcessingTime(100);

    //retrieve list of emitted records for assertions
    testHarness.getOutput should contain (3)

    //retrieve list of records emitted to a specific side output for assertions (ProcessFunction only)
    //testHarness.getSideOutput(new OutputTag[Int]("invalidRecords")) should have size 0
  }
}
```
{% endtab %}
{% endtabs %}

`KeyedOneInputStreamOperatorTestHarness`和`KeyedTwoInputStreamOperatorTestHarness`通过另外提供一个`KeySelector`来实例化，该选择器包括键的类的类型信息。

{% tabs %}
{% tab title="Java" %}
```java
public class StatefulFlatMapFunctionTest {
    private OneInputStreamOperatorTestHarness<String, Long, Long> testHarness;
    private StatefulFlatMap statefulFlatMapFunction;

    @Before
    public void setupTestHarness() throws Exception {

        //instantiate user-defined function
        statefulFlatMapFunction = new StatefulFlatMapFunction();

        // wrap user defined function into a the corresponding operator
        testHarness = new KeyedOneInputStreamOperatorTestHarness<>(new StreamFlatMap<>(statefulFlatMapFunction), new MyStringKeySelector(), Types.STRING);

        // open the test harness (will also call open() on RichFunctions)
        testHarness.open();
    }

    //tests

}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class StatefulFlatMapTest extends FlatSpec with Matchers with BeforeAndAfter {

  private var testHarness: OneInputStreamOperatorTestHarness[String, Long, Long] = null
  private var statefulFlatMapFunction: FlattenFunction = null

  before {
    //instantiate user-defined function
    statefulFlatMapFunction = new StateFulFlatMap

    // wrap user defined function into a the corresponding operator
    testHarness = new KeyedOneInputStreamOperatorTestHarness(new StreamFlatMap(statefulFlatMapFunction),new MyStringKeySelector(), Types.STRING())

    // open the test harness (will also call open() on RichFunctions)
    testHarness.open();
  }

  //tests

}
```
{% endtab %}
{% endtabs %}

在Flink代码库中可以找到更多有关使用这些测试工具的示例，例如：

* `org.apache.flink.streaming.runtime.operators.windowing.WindowOperatorTest` 是测试操作员和用户定义的函数（取决于处理或事件时间）的一个很好的例子。
* `org.apache.flink.streaming.api.functions.sink.filesystem.LocalStreamingFileSinkTest`展示了如何使用来测试自定义接收器`AbstractStreamOperatorTestHarness`。具体来说，它使用`AbstractStreamOperatorTestHarness.snapshot`和`AbstractStreamOperatorTestHarness.initializeState`测试其与Flink的检查点机制的交互。

{% hint style="info" %}
 请注意，`AbstractStreamOperatorTestHarness`及其派生类当前不属于公共API，并且可能会发生变化。
{% endhint %}

#### **单元测试**ProcessFunction

考虑到它的重要性，除了前面可以直接用于测试ProcessFunction的测试工具之外， Flink还提供了名为的测试工具工厂`ProcessFunctionTestHarnesses`，以简化测试工具实例化。考虑以下示例：

{% hint style="info" %}
请注意，要使用此测试工具，还需要介绍上一节中提到的依赖项。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
public static class PassThroughProcessFunction extends ProcessFunction<Integer, Integer> {

	@Override
	public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
        out.collect(value);
	}
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class PassThroughProcessFunction extends ProcessFunction[Integer, Integer] {

    @throws[Exception]
    override def processElement(value: Integer, ctx: ProcessFunction[Integer, Integer]#Context, out: Collector[Integer]): Unit = {
      out.collect(value)
    }
}
```
{% endtab %}
{% endtabs %}

 `ProcessFunctionTestHarnesses`通过传递合适的参数并验证输出，非常容易对此类函数进行单元测试。

{% tabs %}
{% tab title="Java" %}
```java
public class PassThroughProcessFunctionTest {

    @Test
    public void testPassThrough() throws Exception {

        //instantiate user-defined function
        PassThroughProcessFunction processFunction = new PassThroughProcessFunction();

        // wrap user defined function into a the corresponding operator
        OneInputStreamOperatorTestHarness<Integer, Integer> harness = ProcessFunctionTestHarnesses
        	.forProcessFunction(processFunction);

        //push (timestamped) elements into the operator (and hence user defined function)
        harness.processElement(1, 10);

        //retrieve list of emitted records for assertions
        assertEquals(harness.extractOutputValues(), Collections.singletonList(1));
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class PassThroughProcessFunctionTest extends FlatSpec with Matchers {

  "PassThroughProcessFunction" should "forward values" in {

    //instantiate user-defined function
    val processFunction = new PassThroughProcessFunction

    // wrap user defined function into a the corresponding operator
    val harness = ProcessFunctionTestHarnesses.forProcessFunction(processFunction)

    //push (timestamped) elements into the operator (and hence user defined function)
    harness.processElement(1, 10)

    //retrieve list of emitted records for assertions
    harness.extractOutputValues() should contain (1)
  }
}
```
{% endtab %}
{% endtabs %}

 有关如何使用更多的例子`ProcessFunctionTestHarnesses`，以测试不同类型的`ProcessFunction`，如`KeyedProcessFunction`，`KeyedCoProcessFunction`，`BroadcastProcessFunction`等，建议用户看`ProcessFunctionTestHarnessesTest`。

## 测试Flink作业

### JUnit规则 `MiniClusterWithClientResource`

 Apache Flink提供了一个名为`MiniClusterWithClientResource`的JUnit规则，用于在本地嵌入式迷你集群上测试完成的作业。

要使用MiniClusterWithClientResource，需要一个附加的依赖项\(测试范围\)。

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-test-utils_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

 让我们采用与`MapFunction`前面各节相同的简单方法。

{% tabs %}
{% tab title="Java" %}
```java
public class IncrementMapFunction implements MapFunction<Long, Long> {

    @Override
    public Long map(Long record) throws Exception {
        return record + 1;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class IncrementMapFunction extends MapFunction[Long, Long] {

    override def map(record: Long): Long = {
        record + 1
    }
}
```
{% endtab %}
{% endtabs %}

现在可以在本地Flink集群中测试使用这个MapFunction的简单管道，如下所示。

{% tabs %}
{% tab title="Java" %}
```java
public class ExampleIntegrationTest {

     @ClassRule
     public static MiniClusterWithClientResource flinkCluster =
         new MiniClusterWithClientResource(
             new MiniClusterResourceConfiguration.Builder()
                 .setNumberSlotsPerTaskManager(2)
                 .setNumberTaskManagers(1)
                 .build());

    @Test
    public void testIncrementPipeline() throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // configure your test environment
        env.setParallelism(2);

        // values are collected in a static variable
        CollectSink.values.clear();

        // create a stream of custom elements and apply transformations
        env.fromElements(1L, 21L, 22L)
                .map(new IncrementMapFunction())
                .addSink(new CollectSink());

        // execute
        env.execute();

        // verify your results
        assertTrue(CollectSink.values.containsAll(2L, 22L, 23L));
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
class StreamingJobIntegrationTest extends FlatSpec with Matchers with BeforeAndAfter {

  val flinkCluster = new MiniClusterWithClientResource(new MiniClusterResourceConfiguration.Builder()
    .setNumberSlotsPerTaskManager(1)
    .setNumberTaskManagers(1)
    .build)

  before {
    flinkCluster.before()
  }

  after {
    flinkCluster.after()
  }


  "IncrementFlatMapFunction pipeline" should "incrementValues" in {

    val env = StreamExecutionEnvironment.getExecutionEnvironment

    // configure your test environment
    env.setParallelism(2)

    // values are collected in a static variable
    CollectSink.values.clear()

    // create a stream of custom elements and apply transformations
    env.fromElements(1, 21, 22)
       .map(new IncrementMapFunction())
       .addSink(new CollectSink())

    // execute
    env.execute()

    // verify your results
    CollectSink.values should contain allOf (2, 22, 23)
    }
}
// create a testing sink
class CollectSink extends SinkFunction[Long] {

  override def invoke(value: Long): Unit = {
    synchronized {
      CollectSink.values.add(value)
    }
  }
}

object CollectSink {
    // must be static
    val values: util.List[Long] = new util.ArrayList()
}
```
{% endtab %}
{% endtabs %}

关于`MiniClusterWithClientResource`集成测试的几点说明:

* 为了不从生产到测试复制整个管道代码，请将源和接收器插入生产代码中，并在测试中注入特殊的测试源和测试接收器。
* 这里使用CollectSink中的静态变量，因为在将所有操作符分布到集群之前，Flink会对它们进行序列化。通过静态变量与本地Flink迷你集群实例化的操作符通信是解决这个问题的一种方法。或者，您可以使用测试接收器将数据写入临时目录中的文件。
*  如果您的作业使用事件计时器计时器，则可以实现自定义_并行_源功能来发出水印。
* 建议始终以&gt; 1的并行度在本地测试管道，以识别仅在并行执行的管道中出现的错误。
* 优先选择@ClassRule而不是@Rule，这样多个测试就可以共享同一个Flink集群。这样做可以节省大量的时间，因为Flink集群的启动和关闭通常会控制实际测试的执行时间。
* 如果您的管道包含自定义状态处理，则可以通过启用检查点并在迷你集群中重新启动作业来测试其正确性。为此，您需要通过从管道中的\(只测试的\)用户定义函数抛出异常来触发失败。



