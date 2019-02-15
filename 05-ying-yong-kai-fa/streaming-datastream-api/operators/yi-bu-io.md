# 异步I/O

本页介绍了Flink的API用于异步I/O和外部数据存储。对于不熟悉异步或事件驱动编程的用户来说，需要事先了解下关于Futures和事件驱动编程的知识。

注：有关异步I / O实用程序的设计和实现的详细信息，请参阅提议和设计文档 [FLIP-12：异步I / O设计和实现](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=65870673)。

## 异步I/O操作的需要

当与外部系统交互时（例如，当使用存储在数据库中的数据丰富流事件时），需要注意与外部系统的通信延迟不会支配流应用程序的总工作。

简单地访问外部数据库中的数据，例如在MapFunction中，通常意味着同步交互：请求被发送到数据库，MapFunction等待直到收到响应。在许多情况下，这种等待占据了函数的大部分时间。

与数据库的异步交互意味着单个并行函数实例可以同时处理多个请求，并同时接收响应。这样，等待时间可以与发送其他请求和接收响应重叠。至少，等待时间是在多个请求上分摊的。这在大多数情况下实现了更高的流吞吐量。

![](../../../.gitbook/assets/image%20%281%29.png)

{% hint style="info" %}
注意：在某些情况下，通过将MapFunction扩展到非常高的并行性来提高吞吐量是可行的，但通常会带来非常高的资源成本：拥有更多的并行度的MapFunction实例意味着更多的任务、线程、Flink内部网络连接、到数据库的网络连接、缓冲区和内部bookkeeping的开销
{% endhint %}

## 先决条件

如上一节所示，对数据库（或键/值存储）实现适当的异步I / O需要客户端访问支持异步请求的数据库。许多流行的数据库提供这样的客户端

在没有这样的客户端的情况下，可以通过创建多个客户端并使用线程池处理同步调用来尝试将同步客户端转变为有限的并发客户端。但是，这种方法通常比适当的异步客户端效率低。

## 异步I/O API

Flink的Async I / O API允许用户将异步请求客户端与数据流一起使用。API处理数据流的集成，以及结果顺序，事件时间，容错等。

假设有一个目标数据库的异步客户端，需要三个部分来实现对数据库的异步I / O流转换：

* AsyncFunction的一个实现，用来分派请求 
* 获取操作结果并将其传递给ResultFuture的回调 
* 将异步I/O操作作为转换应用于DataStream

以下代码示例说明了基本模式：

{% tabs %}
{% tab title="Java" %}
```java
// This example implements the asynchronous request and callback with Futures that have the
// interface of Java 8's futures (which is the same one followed by Flink's Future)

/**
 * An implementation of the 'AsyncFunction' that sends requests and sets the callback.
 */
class AsyncDatabaseRequest extends RichAsyncFunction<String, Tuple2<String, String>> {

    /** The database specific client that can issue concurrent requests with callbacks */
    private transient DatabaseClient client;

    @Override
    public void open(Configuration parameters) throws Exception {
        client = new DatabaseClient(host, post, credentials);
    }

    @Override
    public void close() throws Exception {
        client.close();
    }

    @Override
    public void asyncInvoke(String key, final ResultFuture<Tuple2<String, String>> resultFuture) throws Exception {

        // issue the asynchronous request, receive a future for result
        final Future<String> result = client.query(key);

        // set the callback to be executed once the request by the client is complete
        // the callback simply forwards the result to the result future
        CompletableFuture.supplyAsync(new Supplier<String>() {

            @Override
            public String get() {
                try {
                    return result.get();
                } catch (InterruptedException | ExecutionException e) {
                    // Normally handled explicitly.
                    return null;
                }
            }
        }).thenAccept( (String dbResult) -> {
            resultFuture.complete(Collections.singleton(new Tuple2<>(key, dbResult)));
        });
    }
}

// create the original stream
DataStream<String> stream = ...;

// apply the async I/O transformation
DataStream<Tuple2<String, String>> resultStream =
    AsyncDataStream.unorderedWait(stream, new AsyncDatabaseRequest(), 1000, TimeUnit.MILLISECONDS, 100);
```
{% endtab %}

{% tab title="Scala" %}
```scala
/**
 * An implementation of the 'AsyncFunction' that sends requests and sets the callback.
 */
class AsyncDatabaseRequest extends AsyncFunction[String, (String, String)] {

    /** The database specific client that can issue concurrent requests with callbacks */
    lazy val client: DatabaseClient = new DatabaseClient(host, post, credentials)

    /** The context used for the future callbacks */
    implicit lazy val executor: ExecutionContext = ExecutionContext.fromExecutor(Executors.directExecutor())


    override def asyncInvoke(str: String, resultFuture: ResultFuture[(String, String)]): Unit = {

        // issue the asynchronous request, receive a future for the result
        val resultFutureRequested: Future[String] = client.query(str)

        // set the callback to be executed once the request by the client is complete
        // the callback simply forwards the result to the result future
        resultFutureRequested.onSuccess {
            case result: String => resultFuture.complete(Iterable((str, result)))
        }
    }
}

// create the original stream
val stream: DataStream[String] = ...

// apply the async I/O transformation
val resultStream: DataStream[(String, String)] =
    AsyncDataStream.unorderedWait(stream, new AsyncDatabaseRequest(), 1000, TimeUnit.MILLISECONDS, 100)
```
{% endtab %}
{% endtabs %}

**重要提示**：`ResultFuture`在第一次通话时完成`ResultFuture.complete`。所有后续`complete`调用都将被忽略。

以下两个参数控制异步操作​​：

* **超时**：超时定义异步请求在被视为失败之前可能需要多长时间。此参数可防止死/失败请求。
* **容量**：此参数定义可以同时进行的异步请求数。尽管异步I / O方法通常会带来更好的吞吐量，但运营商仍然可能成为流应用程序的瓶颈。限制并发请求的数量可确保操作员不会累积不断增长的待处理请求积压，但一旦容量耗尽，它将触发背压。

### 超时处理

当异步I / O请求超时时，默认情况下会引发异常并重新启动作业。如果要处理超时，可以覆盖该`AsyncFunction#timeout`方法。

### 结果顺序

AsyncFunction发出的并发请求通常以某种未定义的顺序完成，该顺序基于哪个请求先完成。为了控制结果记录的发出顺序，Flink提供了两种模式:

* **无序**：异步请求完成后立即发出结果记录。在异步I / O运算符之后，流中记录的顺序与以前不同。当使用_处理时间_作为基本时间特性时，此模式具有最低延迟和最低开销。此模式使用`AsyncDataStream.unorderedWait(...)`。
* **有序**：在这种情况下，保留流顺序。结果记录的发出顺序与触发异步请求的顺序相同（运算符输入记录的顺序）。为此，运算符缓冲结果记录，直到其所有先前记录被发出（或超时）。这通常会在检查点中引入一些额外的延迟和一些开销，因为与无序模式相比，记录或结果在检查点状态下保持更长的时间。此模式使用`AsyncDataStream.orderedWait(...)`。

### 事件时间

当流应用程序与事件时间一起工作时，异步I/O操作符将正确处理水印。具备以下两种排序模式：

* **无序**：水印不会超过记录，反之亦然，这意味着水印会建立一个有序的边界。记录只在水印之间无序地发出。在某个水印之后出现的记录只会在该水印发出之后才会发出。然后，只有在发出水印之前的输入的所有结果记录之后，才会发出水印。

  这意味着，在存在水印的情况下，无序模式引入了与有序模式相同的延迟和管理开销。这种开销的大小取决于水印频率。

* **有序：**保留记录的水印顺序，就像保留记录之间的顺序一样。与_处理时间_相比，开销没有显著变化。

请记住，摄取时间\(_Ingestion Time_\)是事件时间的一种特殊情况，它根据源处理时间自动生成水印。

### 容错保证

异步I / O运算符提供完全一次的容错保证。它在检查点中存储正在进行的异步请求的记录，并在从故障中恢复时恢复/重新触发请求。

### 实施建议

对于具有用于回调的执行器（或scala中的ExecutionContext）的`Futures`的实现，我们建议使用`DirectExecutor`，因为回调通常只做最少的工作，并且`DirectExecutor`避免了额外的线程到线程切换开销。回调通常只将结果传递给`ResultFuture`，后者将其添加到输出缓冲区。包含记录发送和与检查点bookkeeping的交互的繁重逻辑在专用的线程池中发生。

`DirectExecutor`可以通过`org.apache.flink.runtime.concurrent.Executors.directExecutor()`或 `com.google.common.util.concurrent.MoreExecutors.directExecutor()`获得。

### 警告

**AsyncFunction不是多线程的**

例如，以下模式会导致阻塞asyncInvoke\(…\)函数，从而使异步行为无效:

* 使用 lookup/query方法调用阻塞的数据库客户机，直到收到返回的结果**为止**
* 阻塞/等待asyncInvoke\(…\)方法中的异步客户机返回的future-type对象



