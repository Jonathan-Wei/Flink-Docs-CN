# 数据源\(Data Source\)

{% hint style="warning" %}
**注意：**这描述了Flink 1.11中作为[FLIP-27的](https://cwiki.apache.org/confluence/display/FLINK/FLIP-27%3A+Refactor+Source+Interface)一部分引入的新数据源API 。该新API当前处于**测试版**状态。

（从Flink 1.11开始）大多数现有的源连接器尚未使用此新API实现，而是使用基于[SourceFunction](https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/functions/source/SourceFunction.java)的以前的API[实现](https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/functions/source/SourceFunction.java)。
{% endhint %}

本页描述Flink的数据源API及其背后的概念和体系结构。 **如果您对Flink中的数据源的工作方式感兴趣，或者想要实现新的数据源，请阅读此文章。**

如果你正在寻找预定义的源连接器，请查阅[连接器文档](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/connectors/)。

## 数据源概念

 数据源具有三个核心组件：_Splits_，_SplitEnumerator_和_SourceReader_。

###  **核心组件**

###  **跨流和批量统一**

###  **有界文件源**

###  **无限流文件源**



###  **有界卡夫卡源**

## 数据源API

本节描述FLIP-27中引入的新Source API的主要接口，并为开发人员提供有关Source开发的技巧。

### **Source**

### SplitEnumerator

### SourceReader

### Use the Source





## 

![](../../.gitbook/assets/image%20%2847%29.png)

## Split Reader API

### SplitReader

 `SplitReader`API只有三个方法：

### SourceReaderBase

### SplitFetcherManager

![](../../.gitbook/assets/image%20%2849%29.png)

以下代码段实现了此线程模型。

```text
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

```text
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

显然，`SourceReader`实现还可以在`SplitFetcherManager`和`SourceReaderBase`之上轻松实现自己的线程模型。

## Event Time和WaterMarks



### **API**

### **Event 时间戳**

**Watermark 生成器**

![](../../.gitbook/assets/image%20%2848%29.png)

