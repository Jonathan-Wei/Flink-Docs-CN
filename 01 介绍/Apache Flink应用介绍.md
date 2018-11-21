# 应用
Apache Flink是一个用于对无界和有界数据流进行有状态计算的框架。Flink在不同的抽象级别提供多个API，并为常见用例提供专用库。

在这里，我们介绍Flink易于使用和富有表现力的API和库。

---

## 流媒体应用程序构建Block
可以由流处理框架构建和执行的应用程序类型由框架控制流，状态和时间的程度来定义。在下文中，我们描述了流处理应用程序的这些构建块，并解释了Flink处理它们的方法。

---

### 流
显然，流是流处理的一个基本方面。但是，流可能具有不同的特性，这些特性会影响流的处理方式和处理方式。Flink是一个通用的处理框架，可以处理任何类型的流。

 - 有界的和无界的流:流可以是无界的或有界的，即，固定大小的数据集。Flink具有处理无界流的复杂特性，但也有专门的操作符来有效地处理有界流。
 - 实时流和记录流:所有数据都作为流生成。有两种处理数据的方法。当流生成或将其持久化到存储系统(例如文件系统或对象存储)时实时处理，并在稍后处理。Flink应用程序可以处理记录或实时流。

---

### 状态（state）
每个重要的流应用程序都是有状态的，即只有对单个事件应用转换的应用程序才需要状态。运行基本业务逻辑的任何应用程序都需要记住事件或中间结果，以便在以后的时间点访问它们，例如在收到下一个事件时或在特定持续时间之后。

![function-state](https://github.com/Jonathan-Wei/Flink-Docs-CN/blob/master/01%20%E4%BB%8B%E7%BB%8D/images/function-state.png)

应用程序状态是Apache Flink的一等公民。可以通过查看Flink在状态处理环境中提供的所有功能来查看。
 - 多状态基元（Multiple State Primitives）：Flink为不同的数据结构提供状态基元，例如原子值，列表或映射。开发人员可以根据函数的访问模式选择最有效的状态原语。
 - 可插拔状态后端（Pluggable State Backends）：应用程序状态由可插拔状态后端管理和检查点。Flink具有不同的状态后端，可以在内存或RocksDB中存储状态，RocksDB是一种高效的嵌入式磁盘数据存储。也可以插入自定义状态后端。
 - 完全一次的状态一致性（Exactly-once state consistency）：Flink的检查点和恢复算法可确保在发生故障时应用程序状态的一致性。因此，故障是透明处理的，不会影响应用程序的正确性。
 - 非常大的状态（Very Large State）：由于其异步和增量检查点算法，Flink能够维持几TB的应用程序状态。
 - 可扩展的应用程序（Scalable Applications）：Flink通过将状态重新分配给更多或更少的工作人员来支持有状态应用程序的扩展。

---

### 时间（Time）
时间是流媒体应用的另一个重要组成部分 大多数事件流都具有固有的时间语义，因为每个事件都是在特定时间点生成的。此外，许多常见的流计算基于时间，例如窗口聚合，会话化，模式检测和基于时间的连接。流处理的一个重要方面是应用程序如何测量时间，即事件时间和处理时间的差异。

Flink提供了一组丰富的与时间相关的功能。

 - Event-time Mode：使用事件时间语义处理流的应用程序根据事件的时间戳计算结果。因此，无论是否处理记录的或实时的事件，事件时间处理都允许准确和一致的结果。
 - 支持Watermark：Flink使用Watermark来推断事件时间应用中的时间。Watermark也是一种灵活的机制，可以权衡结果的延迟和完整性。
 - 延迟数据处理（Late Data Handling）：当使用水印在事件时间模式下处理流时，可能会发生在所有相关事件到达之前已完成计算。这类事件被称为迟发事件。Flink具有多种处理延迟事件的选项，例如通过侧输出重新路由它们以及更新以前完成的结果。
 - 处理时间模式（Processing-time Mode）：除了事件时间模式之外，Flink还支持处理时间语义，该处理时间语义执行由处理机器的挂钟时间触发的计算。处理时间模式适用于具有严格的低延迟要求的某些应用，这些要求可以容忍近似结果。

---

## 分层API
Flink提供三层API。每个API在简洁性和表达性之间提供不同的权衡，并针对不同的用例。
我们简要介绍每个API，讨论它的应用程序，并展示一个代码示例。

![api-stack](https://github.com/Jonathan-Wei/Flink-Docs-CN/blob/master/01%20%E4%BB%8B%E7%BB%8D/images/api-stack.png)

---

#### ProcessFunctions
ProcessFunctions是Flink提供的最具表现力的功能接口。Flink提供ProcessFunctions来处理来自窗口中分组的一个或两个输入流或事件的单个事件。ProcessFunctions提供对时间和状态的细粒度控制。ProcessFunction可以任意修改其状态并注册将在未来触发回调函数的定时器。因此，ProcessFunctions可以实现许多有状态事件驱动应用程序所需的复杂的每事件业务逻辑。

以下示例显示了KeyedProcessFunction对一个KeyedStream进行匹配START以及END事件进行操作的示例。当一个START被接收的事件，则该函数在记住其状态时间戳和寄存器在四个小时的计时器。如果END在计时器触发之前收到事件，则该函数计算事件END和START事件之间的持续时间，清除状态并返回值。否则，计时器只会触发并清除状态。

```java
/**
 * Matches keyed START and END events and computes the difference between 
 * both elements' timestamps. The first String field is the key attribute, 
 * the second String attribute marks START and END events.
 */
public static class StartEndDuration
    extends KeyedProcessFunction<String, Tuple2<String, String>, Tuple2<String, Long>> {

  private ValueState<Long> startTime;

  @Override
  public void open(Configuration conf) {
    // obtain state handle
    startTime = getRuntimeContext()
      .getState(new ValueStateDescriptor<Long>("startTime", Long.class));
  }

  /** Called for each processed event. */
  @Override
  public void processElement(
      Tuple2<String, String> in,
      Context ctx,
      Collector<Tuple2<String, Long>> out) throws Exception {

    switch (in.f1) {
      case "START":
        // set the start time if we receive a start event.
        startTime.update(ctx.timestamp());
        // register a timer in four hours from the start event.
        ctx.timerService()
          .registerEventTimeTimer(ctx.timestamp() + 4 * 60 * 60 * 1000);
        break;
      case "END":
        // emit the duration between start and end event
        Long sTime = startTime.value();
        if (sTime != null) {
          out.collect(Tuple2.of(in.f0, ctx.timestamp() - sTime));
          // clear the state
          startTime.clear();
        }
      default:
        // do nothing
    }
  }

  /** Called when a timer fires. */
  @Override
  public void onTimer(
      long timestamp,
      OnTimerContext ctx,
      Collector<Tuple2<String, Long>> out) {

    // Timeout interval exceeded. Cleaning up the state.
    startTime.clear();
  }
}
```

这个例子说明了KeyedProcessFunction的表达能力，但也强调了它是一个相当冗长的接口

---

### DataStream API
DataStream API通过查询外部数据存储提供了许多常见的流处理操作，如windowing，record-at-a-time transformations，以及通过查询外部数据存储来丰富事件。数据流API可用于Java和Scala和基于功能，如map()，reduce()和aggregate()。可以通过扩展接口或Java或Scala lambda函数来定义函数。

以下示例显示如何对点击流进行会话并计算每个会话的点击次数。

```java
// a stream of website clicks
DataStream<Click> clicks = ...

DataStream<Tuple2<String, Long>> result = clicks
  // project clicks to userId and add a 1 for counting
  .map(
    // define function by implementing the MapFunction interface.
    new MapFunction<Click, Tuple2<String, Long>>() {
      @Override
      public Tuple2<String, Long> map(Click click) {
        return Tuple2.of(click.userId, 1L);
      }
    })
  // key by userId (field 0)
  .keyBy(0)
  // define session window with 30 minute gap
  .window(EventTimeSessionWindows.withGap(Time.minutes(30L)))
  // count clicks per session. Define function as lambda function.
  .reduce((a, b) -> Tuple2.of(a.f0, a.f1 + b.f1));
```

---

### SQL和Table API
Flink具有两个关系API，Table API和SQL。这两个api都是批处理和流处理的统一api，即，查询在无界流、实时流或有界流上以相同的语义执行，并产生相同的结果。Table API和SQL利用Apache Calcite 进行解析、验证和查询优化。它们可以无缝地与DataStream和DataSet api集成，并支持用户定义的标量、聚合和表值函数。
Flink的关系api旨在简化数据分析、数据流水线和ETL应用程序的定义。
下面的示例显示了用于处理clickstream的SQL查询，并计算每个会话的单击次数。这是与DataStream API示例相同的用例。

``` sql
SELECT userId, COUNT(*)
FROM clicks
GROUP BY SESSION(clicktime, INTERVAL '30' MINUTE), userId
```

---

## Libraries
Flink为通用数据处理用例提供了几个库。这些库通常嵌入在API中，而不是完全独立的。因此，它们可以从API的所有特性中获益，并与其他库集成.
 - 复杂事件处理(CEP):模式检测是事件流处理的一个非常常见的用例。Flink的CEP库提供了一个API来指定事件的模式(考虑正则表达式或状态机)。CEP库与Flink的DataStream API集成，以便在DataStreams上评估模式。CEP库的应用程序包括网络入侵检测、业务流程监视和欺诈检测。
 - DataSet API: DataSet API是Flink的核心API，用于批处理应用程序。DataSet API的原语包括map、reduce、(外部)join、co-group和iterate。所有操作都由算法和数据结构支持，这些算法和数据结构对内存中的序列化数据进行操作，如果数据大小超过内存预算，则会溢出到磁盘。Flink的DataSet API的数据处理算法受到传统数据库操作符的启发，如混合哈希连接或外部合并排序。
 - Gelly: Gelly是一个用于可伸缩图形处理和分析的库。Gelly是在数据集API的基础上实现的，并与之集成。因此，它得益于可伸缩和健壮的操作符。Gelly提供了内置的算法，比如标签传播、三角形枚举和页面排名，但也提供了一个图形API，简化了自定义图形算法的实现。

