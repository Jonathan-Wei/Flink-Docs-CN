# 数据源\(Data Source\)

{% hint style="warning" %}
**注意：**这描述了Flink 1.11中作为[FLIP-27的](https://cwiki.apache.org/confluence/display/FLINK/FLIP-27%3A+Refactor+Source+Interface)一部分引入的新数据源API 。该新API当前处于**测试版**状态。

（从Flink 1.11开始）大多数现有的源连接器尚未使用此新API实现，而是使用基于[SourceFunction](https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/functions/source/SourceFunction.java)的以前的API[实现](https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/functions/source/SourceFunction.java)。
{% endhint %}

本页描述Flink的数据源API及其背后的概念和体系结构。 **如果您对Flink中的数据源的工作方式感兴趣，或者想要实现新的数据源，请阅读此文章。**

如果你正在寻找预定义的源连接器，请查阅[连接器文档](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/connectors/)。

## 数据源概念

###  **核心组件**

数据源具有三个核心组件：_Splits_，_SplitEnumerator_和_SourceReader_。

* **Splits**是是源使用的数据的一部分，如文件或日志分区。**Splits**是`Source`分配工作和并行数据读取的粒度。
* **SourceReader**请求对它们进行拆分和处理，例如通过读取拆分所表示的文件或日志分区。该`SourceReader`在`SourceOperators`中的任务管理器上并行运行，并产生并行的事件/记录流。
* **SplitEnumerator**生成`Split`并将它们分配给_`SourceReaders`_。它在作业管理器上作为单个实例运行，并负责维护待处理的`Split`的待办事项，并以平衡的方式将其分配给读者。

 [Source](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/connector/source/Source.java)类是将上述三个组件连接在一起的API入口点。

![](../../.gitbook/assets/image%20%2865%29.png)

###  **跨流和批量统一**

**Source** API以统一的方式支持无限流源和无限批处理源。

两种情况之间的差异是最小的：在有界/批处理情况下，枚举器生成一个固定的_Splits_集合，每个_Splits_必然是有限的。在无限流传输的情况下，这两个中的一个不为真（_Splits_不是有限的，否则枚举器会继续生成新的_Splits_）。

**例子**

下面是一些简化的概念示例，用于说明数据源组件在流和批处理情况下如何交互。

请注意，这并不能准确描述Kafka和文件源实现是如何工作的;为了便于说明，部分被简化了。

###  **有界文件源**

此`Source`具有要读取的目录的URI/路径，以及定义如何解析文件的格式。

* _**Splits**_是一个文件，或文件的区域（如果数据格式支持分裂文件）。
* _**SplitEnumerator**_列出给定目录路径下的所有文件。它将`Split`分配给下一个请求`Split`的读取器。分配所有`Split`后，它将使用_`NoMoreSplits`_响应请求。
* _**SourceReader**_请求分割并读取分配的`Split`\(文件或文件区域\)，并使用给定的格式解析它。如果它没有得到另一个`Split`，但是一个`NoMoreSplits`消息，它将结束。

###  **无限流文件源**

 除了_`SplitEnumerator`_永远不会用_`NoMoreSplits`进行_响应并定期在给定的URI / Path下列出内容以检查新文件之外，此`Source`的工作方式与上述相同。找到新文件后，它将为它们生成新的`Split`并将其分配给可用的`SourceReader`。

###  无**界Kafka Source**

该**`Source`**有一个Kafka Topic\(或Topic列表或Topic正则表达式\)和一个反序列化器来解析记录。

* 一个_Split_是Kafka Topic Partition。
* _SplitEnumerator_连接到代理，列出订阅`Topic`中涉及的所有`Topic`分区。枚举器可以选择性地重复此操作以发现新添加的`Topic`/`Partition`。
* _SourceReader_使用`KafkaConsumer`读取指定的`Split`\(`Topic Partition`\)，并使用提供的反序列化器反序列化记录。`Split`\(`Topic Partition`\)没有结束，因此读者永远不会到达数据的结尾。

###  **有界的Kafka Source**

 除每个`Split`（`Kafka Topic`）具有定义的结束偏移外，其他与上述相同。一旦_SourceReader_达到了Split的结束偏移量，它将完成该Split。一旦所有分配的Split完成，SourceReader便完成。

## 数据源API

本节描述FLIP-27中引入的新Source API的主要接口，并为开发人员提供有关Source开发的技巧。

### **Source**

Source ****API是一个工厂样式的接口，用于创建以下组件。

* _Split Enumerator_
* _Source Reader_
* _Split Serializer_
* _Enumerator Checkpoint Serializer_

除此之外，Source还提供了Source的[boundedness](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/connector/source/Boundedness.java)属性，因此Flink可以选择适当的模式来运行Flink作业。

Source实现应该是可序列化的，因为Source实例在运行时被序列化并上传到Flink集群。

### SplitEnumerator

SplitEnumerator应该是Source的“大脑”。SplitEnumerator的典型实现如下:

* `SourceReader` 登记处理
* `SourceReader` 故障处理
  * `addSplitsBack()`方法将在`SourceReader`失败时被调用。`SplitEnumerator`应该收回未被失败的`SourceReader`确认的`Split`赋值。
* `SourceEvent` 处理
  * `SourceEvents`是在`SplitEnumerator`和`SourceReader`之间发送的自定义事件。实现可以利用这种机制来执行复杂的协调。
* `Split`发现和分配
  * `SplitEnumerator`可以为`SourceReader`分配分片以响应各种事件，包括发现新的`Split`、新的`SourceReader`注册、`SourceReader`失败等。

`SplitEnumerator`可以在`SplitEnumeratorContext`的帮助下完成上述工作，`SplitEnumerator`在创建或恢复`SplitEnumerator`时提供给源。`SplitEnumeratorContext`允许`SplitEnumerator`检索读取器的必要信息并执行协调操作。`Source`实现将`SplitEnumeratorContext`传递给`SplitEnumerator`实例。

尽管`SplitEnumerator`实现可以以一种响应式的方式工作，只在它的方法被调用时采取协调动作，但有些`SplitEnumerator`实现可能希望主动采取动作。例如，`SplitEnumerator`可能希望定期运行`Split`发现，并将新的`Split`分配给`SourceReader`。这样的实现可能会发现`callAsync()`方法`SplitEnumeratorContext`非常方便。下面的代码片段展示了`SplitEnumerator`实现如何在不维护自己线程的情况下实现这一点。

```java
class MySplitEnumerator implements SplitEnumerator<MySplit> {
    private final long DISCOVER_INTERVAL = 60_000L;

    /**
     * A method to discover the splits.
     */
    private List<MySplit> discoverSplits() {...}
    
    @Override
    public void start() {
        ...
        enumContext.callAsync(this::discoverSplits, splits -> {
            Map<Integer, List<MockSourceSplit>> assignments = new HashMap<>();
            int parallelism = enumContext.currentParallelism();
            for (MockSourceSplit split : splits) {
                int owner = split.splitId().hashCode() % parallelism;
                assignments.computeIfAbsent(owner, new ArrayList<>()).add(split);
            }
            enumContext.assignSplits(new SplitsAssignment<>(assignments));
        }, 0L, DISCOVER_INTERVAL);
        ...
    }
    ...
}
```

### SourceReader

 [SourceReader](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/connector/source/SourceReader.java)是运行在任务管理器中的一个组件，用于消费`split`中的记录。

`SourceReader`公开了一个基于拉形式的消费接口。Flink任务循环调用`pollNext(ReaderOutput)`来轮询`SourceReader`的记录。`pollNext(ReaderOutput)`方法的返回值指示源读取器的状态。

* `MORE_AVAILABLE` -SourceReader可以立即使用更多记录。
* `NOTHING_AVAILABLE` -SourceReader目前没有更多记录，但是将来可能会有更多记录。
* `END_OF_INPUT`-SourceReader消费所有记录并到达数据结尾。这意味着可以关闭SourceReader。

为了提高性能，为`pollNext(ReaderOutput)`方法提供了一个`ReaderOutput`，因此如果有必要，`SourceReader`可以在一次`pollNext()`调用中发出多条记录。例如，有时候外部系统以块的粒度工作。一个块可以包含多个记录，但`Source`只能在块边界处检查点。在这种情况下，`SourceReader`可以一次将一个块中的所有记录发送给`ReaderOutput`。**然而，除非必要，`SourceReader`实现应避免在单个`pollNext(ReaderOutput)`调用中发出多条记录**。这是因为从`SourceReader`轮询的任务线程工作在事件循环中，不能阻塞。

`SourceReader`的所有状态都应该在`snapshotState()`调用时返回的`SourceSplits`中维护。这样做可以在需要时将`SourceSplits`重新分配给其他`SourceReader`。

在创建`SourceReader`时，向`Sourece`提供一个`SourceReaderContext`。预期`Source`将把上下文传递给`SourceReader`实例。`SourceReader`可以通过`SourceReaderContext`将`SourceEvent`发送给它的`SplitEnumerator`。`Source`的一个典型设计模式是让`SourceReader`将其本地信息报告给具有全局视图的`SplitEnumerator`, `SplitEnumerator`可以做出决策。

SourceReader API是一个低等级API，允许用户手动处理`Split`，并拥有自己的线程模型来获取和切换记录。为了方便`SourceReader`的实现，Flink提供了一个 [SourceReaderBase](https://github.com/apache/flink/blob/master/flink-connectors/flink-connector-base/src/main/java/org/apache/flink/connector/base/source/reader/SourceReaderBase.java)类，这大大减少了编写`SourceReader`所需的工作量。**强烈建议连接器开发人员利用`SourceReaderBase`，而不是从头编写`SourceReader`。** 有关更多详细信息，请检查[Split Reader API](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/sources.html#the-split-reader-api)部分。

### Use the Source

为了从`Source`创建`DataStream`，需要将`Source`传递给`StreamExecutionEnvironment`。例如,

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

Source mySource = new MySource(...);

DataStream<Integer> stream = env.fromSource(
        mySource,
        WatermarkStrategy.noWatermarks(),
        "MySourceName");
...
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

val mySource = new MySource(...)

val stream = env.fromSource(
      mySource,
      WatermarkStrategy.noWatermarks(),
      "MySourceName")
...
```
{% endtab %}
{% endtabs %}

## Split Reader API

核心SourceReader API是完全异步的，需要实现手动管理异步`Split`读取。然而，在实践中，大多数源执行阻塞操作，比如阻塞客户机上的`poll()`调用\(例如`KafkaConsumer`\)，或者阻塞分布式文件系统\(HDFS, S3，…\)上的I/O操作。为了与异步源API兼容，这些阻塞\(同步\)操作需要在单独的线程中进行，这些线程将数据移交给读取器的异步部分。

 [SplitReader](https://github.com/apache/flink/blob/master/flink-connectors/flink-connector-base/src/main/java/org/apache/flink/connector/base/source/reader/splitreader/SplitReader.java)是一个高级API，用于简单的同步读取/轮询`Source`实现，比如文件读取、Kafka等。

核心是`SourceReaderBase`类，它接受一个`SplitReader`，并创建运行`SplitReader`的获取线程，支持不同的消费线程模型。

### SplitReader

 `SplitReader`API只有三个方法：

* 一个阻塞获取方法，返回 [RecordsWithSplitIds](https://github.com/apache/flink/blob/master/flink-connectors/flink-connector-base/src/main/java/org/apache/flink/connector/base/source/reader/RecordsWithSplitIds.java)
* 一种非阻塞方法，用于处理拆分更改。
* 一种非阻塞唤醒方法，用于唤醒阻塞提取操作。

`SplitReader`只关注从外部系统读取记录，因此与`SourceReader`相比要简单得多。请查看该类的Java文档以了解更多细节。

### SourceReaderBase

SourceReader实现通常会做以下事情:

* 使用一个线程池，以阻塞的方式从外部系统的`Split`中获取线程。
* 处理内部获取线程和其他方法调用\(如`pollNext(ReaderOutput))`之间的同步。。
* 保持每个`Split Watermark`的对齐方式。
* 维护检查点的每个`Split`的状态。

为了减少编写新的`SourceReader`的工作量，Flink提供了一个 [SourceReaderBase](https://github.com/apache/flink/blob/master/flink-connectors/flink-connector-base/src/main/java/org/apache/flink/connector/base/source/reader/SourceReaderBase.java)类作为`SourceReader`的基实现。`SourceReaderBase`已经完成了上述所有工作。要编写一个新的`SourceReader`，只需让`SourceReader`实现继承`SourceReaderBase`，填入一些方法，然后实现一个高级的 [SplitReader](https://github.com/apache/flink/blob/master/flink-connectors/flink-connector-base/src/main/java/org/apache/flink/connector/base/source/reader/splitreader/SplitReader.java)。

### SplitFetcherManager

`SourceReaderBase`支持一些开箱就开的线程模型，这取决于它使用的`SplitFetcherManager`的行为。`SplitFetcherManager`帮助创建和维护一个`SplitFetchers`池，每个`SplitReader`都在进行抓取。它还决定了如何为每个`Split`提取器分配`Splits`。

举个例子，如下所示，`SplitFetcherManager`可能有固定数量的线程，每个线程从分配给`SourceReader`的一些`Split`中获取数据。

![](../../.gitbook/assets/image%20%2851%29.png)

以下代码段实现了此线程模型。

```java
/**
 * A SplitFetcherManager that has a fixed size of split fetchers and assign splits 
 * to the split fetchers based on the hash code of split IDs.
 */
public class FixedSizeSplitFetcherManager<E, SplitT extends SourceSplit> 
        extends SplitFetcherManager<E, SplitT> {
    private final int numFetchers;

    public FixedSizeSplitFetcherManager(
            int numFetchers,
            FutureNotifier futureNotifier,
            FutureCompletingBlockingQueue<RecordsWithSplitIds<E>> elementsQueue,
            Supplier<SplitReader<E, SplitT>> splitReaderSupplier) {
        super(futureNotifier, elementsQueue, splitReaderSupplier);
        this.numFetchers = numFetchers;
        // Create numFetchers split fetchers.
        for (int i = 0; i < numFetchers; i++) {
            startFetcher(createSplitFetcher());
        }
    }

    @Override
    public void addSplits(List<SplitT> splitsToAdd) {
        // Group splits by their owner fetchers.
        Map<Integer, List<SplitT>> splitsByFetcherIndex = new HashMap<>();
        splitsToAdd.forEach(split -> {
            int ownerFetcherIndex = split.hashCode() % numFetchers;
            splitsByFetcherIndex
                    .computeIfAbsent(ownerFetcherIndex, s -> new ArrayList<>())
                    .add(split);
        });
        // Assign the splits to their owner fetcher.
        splitsByFetcherIndex.forEach((fetcherIndex, splitsForFetcher) -> {
            fetchers.get(fetcherIndex).addSplits(splitsForFetcher);
        });
    }
}
```

`SourceReader`使用这种线程模型可以像下面创建：

```java
public class FixedFetcherSizeSourceReader<E, T, SplitT extends SourceSplit, SplitStateT>
        extends SourceReaderBase<E, T, SplitT, SplitStateT> {

    public FixedFetcherSizeSourceReader(
            FutureNotifier futureNotifier,
            FutureCompletingBlockingQueue<RecordsWithSplitIds<E>> elementsQueue,
            Supplier<SplitReader<E, SplitT>> splitFetcherSupplier,
            RecordEmitter<E, T, SplitStateT> recordEmitter,
            Configuration config,
            SourceReaderContext context) {
        super(
                futureNotifier,
                elementsQueue,
                new FixedSizeSplitFetcherManager<>(
                        config.getInteger(SourceConfig.NUM_FETCHERS),
                        futureNotifier,
                        elementsQueue,
                        splitFetcherSupplier),
                recordEmitter,
                config,
                context);
    }

    @Override
    protected void onSplitFinished(Collection<String> finishedSplitIds) {
        // Do something in the callback for the finished splits.
    }

    @Override
    protected SplitStateT initializedState(SplitT split) {
        ...
    }

    @Override
    protected SplitT toSplitType(String splitId, SplitStateT splitState) {
        ...
    }
}
```

显然，`SourceReader`的实现也可以在`SplitFetcherManager`和`SourceReaderBase`的基础上轻松地实现它们自己的线程模型。

## Event Time和WaterMarks

`Event Time`分配和`Watermark`生成作为`Source`的一部分。离开Source Readers的事件流具有事件时间戳，并且\(在流执行期间\)包含Watermark。有关`Event Time`和`Watermark`的介绍，请参阅 [及时流处理](https://ci.apache.org/projects/flink/flink-docs-release-1.12/concepts/timely-stream-processing.html)。

{% hint style="danger" %}
**注意：**基于旧版 [SourceFunction](https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/functions/source/SourceFunction.java)的应用程序通常通过`stream.assigntimestampsandwatermark (WatermarkStrategy)`在稍后的单独步骤中生成时间戳和`Watermark`。这个函数不应该与新的源一起使用，因为时间戳已经被分配，并且它将覆盖以前的可识别分离的Watermark。
{% endhint %}

### **API**

`WatermarkStrategy`在`DataStream API`中创建时传递给`Source`，并创建`TimestampAssigner`和 [WatermarkGenerator](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/common/eventtime/WatermarkGenerator.java).。

`TimestampAssigner`和 `WatermarkGenerator` 作为`ReaderOutput`\(或`SourceOutput`\)的一部分透明地运行，因此`Source`实现者不必实现任何时间戳提取和`Watermark`生成代码。

### **Event 时间戳**

事件时间戳分两个步骤分配:

1. `SourceReader`可以通过调用`SourceOutput`将源记录时间戳附加到事件上。收集\(事件、时间戳\)。这只与基于记录和有时间戳的数据源相关，如Kafka，Kinesis，Pulsar，或Pravega。不基于带有时间戳的记录的源\(如文件\)没有源记录时间戳。此步骤是源连接器实现的一部分，并且没有被使`Source`的应用程序参数化。
2. 由应用程序配置的`TimestampAssigner`分配最后的时间戳。`TimestampAssigner`将看到原始`Source`记录的时间戳和事件。 分配者可以使用_源记录时间戳_或访问事件的字段以获得最终事件时间戳。

这种两步方法允许用户引用源系统中的时间戳和事件数据中的时间戳作为事件时间戳。

注意:当使用一个没有`Source`记录时间戳的数据源\(如文件\)并选择源记录时间戳作为最终事件时间戳时， 事件将获得等于`LONG_MIN` _`（= -9,223,372,036,854,775,808）`_的默认时间戳。

**Watermark 生成器**

`Watermark`生成器仅在流执行期间激活。批量执行使`Watermark`生成器失效;下面所述的所有相关操作实际上都是无操作的。

数据源API支持每个`Split`单独运行`Watermark`生成器。这允许Flink单独观察每个`Split`的事件时间进度，这对于正确处理事件时间倾斜和防止空闲分区阻碍整个应用程序的事件时间进度非常重要。

![](../../.gitbook/assets/image%20%2849%29.png)

 使用_Split Reader API_实现源连接_器时_，将自动处理此问题。所有基于Split Reader API的实现都具有开箱即用的可感知拆分的`Watermark`。

为了实现底层SourceReader API来使用感知分割的`Watermark`生成，该实现必须将事件从不同的分割输出到不同的输出: _`Split-local SourceOutputs`_ 。`Split-local`输出可以通过[ReaderOutput](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/connector/source/ReaderOutput.java)的`createOutputForSplit(splitId)`和`releaseOutputForSplit(splitId)`方法创建和释放。有关详细信息，请参阅该类和方法的JavaDocs。

