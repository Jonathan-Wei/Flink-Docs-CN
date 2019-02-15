# Elasticsearch

此连接器提供可以向[Elasticsearch](https://elastic.co/)索引请求文档操作的接收器 。要使用此连接器，请将以下依赖项之一添加到项目中，具体取决于Elasticsearch安装的版本：

| Maven依赖 | 支持 | Elasticsearch版本 |
| :--- | :--- | :--- |
| flink-connector-elasticsearch\_2.11 | 1.0.0 | 1.x |
| flink-connector-elasticsearch2\_2.11 | 1.0.0 | 2.x |
| flink-connector-elasticsearch5\_2.11 | 1.3.0 | 5.x |
| flink-connector-elasticsearch6\_2.11 | 1.6.0 | 6及更高版本 |

{% hint style="info" %}
请注意，流连接器目前不是二进制发行版的一部分。在[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking.html)查看如何与之关联以进行集群执行。
{% endhint %}

## 安装Elasticsearch

可以在[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)找到有关设置Elasticsearch集群的说明 。确保设置并记住群集名称。必须在创建`ElasticsearchSink`针对群集的请求文档操作时设置此选项。

## Elasticsearch Sink

ElasticsearchSink使用TransportClient\(在6.x之前\)或RestHighLevelClient\(从6.x开始\)与Elasticsearch集群通信。

下面的例子展示了如何配置和创建接收器:

{% tabs %}
{% tab title="Java,Elasticsearch 1.x" %}
```java
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSink;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;

import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
import org.elasticsearch.common.transport.TransportAddress;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

DataStream<String> input = ...;

Map<String, String> config = new HashMap<>();
config.put("cluster.name", "my-cluster-name");
// This instructs the sink to emit after every element, otherwise they would be buffered
config.put("bulk.flush.max.actions", "1");

List<TransportAddress> transportAddresses = new ArrayList<String>();
transportAddresses.add(new InetSocketTransportAddress("127.0.0.1", 9300));
transportAddresses.add(new InetSocketTransportAddress("10.2.3.1", 9300));

input.addSink(new ElasticsearchSink<>(config, transportAddresses, new ElasticsearchSinkFunction<String>() {
    public IndexRequest createIndexRequest(String element) {
        Map<String, String> json = new HashMap<>();
        json.put("data", element);
    
        return Requests.indexRequest()
                .index("my-index")
                .type("my-type")
                .source(json);
    }
    
    @Override
    public void process(String element, RuntimeContext ctx, RequestIndexer indexer) {
        indexer.add(createIndexRequest(element));
    }
}));
```
{% endtab %}

{% tab title="Java,Elasticsearch 2.x/5.x" %}
```java
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.apache.flink.streaming.connectors.elasticsearch5.ElasticsearchSink;

import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

DataStream<String> input = ...;

Map<String, String> config = new HashMap<>();
config.put("cluster.name", "my-cluster-name");
// This instructs the sink to emit after every element, otherwise they would be buffered
config.put("bulk.flush.max.actions", "1");

List<InetSocketAddress> transportAddresses = new ArrayList<>();
transportAddresses.add(new InetSocketAddress(InetAddress.getByName("127.0.0.1"), 9300));
transportAddresses.add(new InetSocketAddress(InetAddress.getByName("10.2.3.1"), 9300));

input.addSink(new ElasticsearchSink<>(config, transportAddresses, new ElasticsearchSinkFunction<String>() {
    public IndexRequest createIndexRequest(String element) {
        Map<String, String> json = new HashMap<>();
        json.put("data", element);
    
        return Requests.indexRequest()
      
```
{% endtab %}

{% tab title="Java,Elasticsearch 6.x" %}
```java
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink;

import org.apache.http.HttpHost;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

DataStream<String> input = ...;

List<HttpHost> httpHosts = new ArrayList<>();
httpHosts.add(new HttpHost("127.0.0.1", 9200, "http"));
httpHosts.add(new HttpHost("10.2.3.1", 9200, "http"));

// use a ElasticsearchSink.Builder to create an ElasticsearchSink
ElasticsearchSink.Builder<String> esSinkBuilder = new ElasticsearchSink.Builder<>(
    httpHosts,
    new ElasticsearchSinkFunction<String>() {
        public IndexRequest createIndexRequest(String element) {
            Map<String, String> json = new HashMap<>();
            json.put("data", element);
        
            return Requests.indexRequest()
                    .index("my-index")
                    .type("my-type")
                    .source(json);
        }
        
        @Override
        public void process(String element, RuntimeContext ctx, RequestIndexer indexer) {
            indexer.add(createIndexRequest(element));
        }
    }
);

// configuration for the bulk requests; this instructs the sink to emit after every element, otherwise they would be buffered
esSinkBuilder.setBulkFlushMaxActions(1);

// provide a RestClientFactory for custom configuration on the internally created REST client
esSinkBuilder.setRestClientFactory(
  restClientBuilder -> {
    restClientBuilder.setDefaultHeaders(...)
    restClientBuilder.setMaxRetryTimeoutMillis(...)
    restClientBuilder.setPathPrefix(...)
    restClientBuilder.setHttpClientConfigCallback(...)
  }
);

// finally, build and add the sink to the job's pipeline
input.addSink(esSinkBuilder.build());
```
{% endtab %}

{% tab title="Scala,Elasticsearch 1.x" %}
```scala
import org.apache.flink.api.common.functions.RuntimeContext
import org.apache.flink.streaming.api.datastream.DataStream
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSink
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer

import org.elasticsearch.action.index.IndexRequest
import org.elasticsearch.client.Requests
import org.elasticsearch.common.transport.InetSocketTransportAddress
import org.elasticsearch.common.transport.TransportAddress

import java.net.InetAddress
import java.util.ArrayList
import java.util.HashMap
import java.util.List
import java.util.Map

val input: DataStream[String] = ...

val config = new java.util.HashMap[String, String]
config.put("cluster.name", "my-cluster-name")
// This instructs the sink to emit after every element, otherwise they would be buffered
config.put("bulk.flush.max.actions", "1")

val transportAddresses = new java.util.ArrayList[TransportAddress]
transportAddresses.add(new InetSocketTransportAddress("127.0.0.1", 9300))
transportAddresses.add(new InetSocketTransportAddress("10.2.3.1", 9300))

input.addSink(new ElasticsearchSink(config, transportAddresses, new ElasticsearchSinkFunction[String] {
  def createIndexRequest(element: String): IndexRequest = {
    val json = new java.util.HashMap[String, String]
    json.put("data", element)
    
    return Requests.indexRequest()
            .index("my-index")
            .type("my-type")
            .source(json)
  }
}))
```
{% endtab %}

{% tab title="Scala,Elasticsearch 2.x/5.x" %}
```scala
import org.apache.flink.api.common.functions.RuntimeContext
import org.apache.flink.streaming.api.datastream.DataStream
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer
import org.apache.flink.streaming.connectors.elasticsearch5.ElasticsearchSink

import org.elasticsearch.action.index.IndexRequest
import org.elasticsearch.client.Requests

import java.net.InetAddress
import java.net.InetSocketAddress
import java.util.ArrayList
import java.util.HashMap
import java.util.List
import java.util.Map

val input: DataStream[String] = ...

val config = new java.util.HashMap[String, String]
config.put("cluster.name", "my-cluster-name")
// This instructs the sink to emit after every element, otherwise they would be buffered
config.put("bulk.flush.max.actions", "1")

val transportAddresses = new java.util.ArrayList[InetSocketAddress]
transportAddresses.add(new InetSocketAddress(InetAddress.getByName("127.0.0.1"), 9300))
transportAddresses.add(new InetSocketAddress(InetAddress.getByName("10.2.3.1"), 9300))

input.addSink(new ElasticsearchSink(config, transportAddresses, new ElasticsearchSinkFunction[String] {
  def createIndexRequest(element: String): IndexRequest = {
    val json = new java.util.HashMap[String, String]
    json.put("data", element)
    
    return Requests.indexRequest()
            .index("my-index")
            .type("my-type")
            .source(json)
  }
}))
```
{% endtab %}

{% tab title="Scala,Elasticsearch 6.x" %}
```scala
import org.apache.flink.api.common.functions.RuntimeContext
import org.apache.flink.streaming.api.datastream.DataStream
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink

import org.apache.http.HttpHost
import org.elasticsearch.action.index.IndexRequest
import org.elasticsearch.client.Requests

import java.util.ArrayList
import java.util.List

val input: DataStream[String] = ...

val httpHosts = new java.util.ArrayList[HttpHost]
httpHosts.add(new HttpHost("127.0.0.1", 9200, "http"))
httpHosts.add(new HttpHost("10.2.3.1", 9200, "http"))

val esSinkBuilder = new ElasticsearchSink.Builer[String](
  httpHosts,
  new ElasticsearchSinkFunction[String] {
    def createIndexRequest(element: String): IndexRequest = {
      val json = new java.util.HashMap[String, String]
      json.put("data", element)
      
      return Requests.indexRequest()
              .index("my-index")
              .type("my-type")
              .source(json)
    }
  }
)

// configuration for the bulk requests; this instructs the sink to emit after every element, otherwise they would be buffered
esSinkBuilder.setBulkFlushMaxActions(1)

// provide a RestClientFactory for custom configuration on the internally created REST client
esSinkBuilder.setRestClientFactory(
  restClientBuilder -> {
    restClientBuilder.setDefaultHeaders(...)
    restClientBuilder.setMaxRetryTimeoutMillis(...)
    restClientBuilder.setPathPrefix(...)
    restClientBuilder.setHttpClientConfigCallback(...)
  }
)

// finally, build and add the sink to the job's pipeline
input.addSink(esSinkBuilder.build)
```
{% endtab %}
{% endtabs %}

对于仍使用现已弃用的`TransportClient`与Elasticsearch集群通信的Elasticsearch版本（即，版本等于或低于5.x），请注意如何使用字符串映射来配置ElasticsearchSink。创建内部使用的TransportClient时，将直接转发此配置映射。配置键在[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)的Elasticsearch文档中进行了说明。特别重要的是`cluster.name`参数，该参数必须与群集的名称相对应。

对于Elasticsearch 6.x及更高版本，在内部，`RestHighLevelClient`用于群集通信。默认情况下，连接器使用REST客户端的默认配置。要为REST客户端进行自定义配置，用户可以在设置构建接收器的`ElasticsearchClient.Builder`时提供`RestClientFactory`实现。

另请注意，该示例仅演示了为每个传入元素执行单个索引请求。通常，`ElasticsearchSinkFunction`可用于执行不同类型的多个请求（例如，`DeleteRequest`，`UpdateRequest`等）。

在内部，Flink Elasticsearch Sink的每个并行实例都使用`BulkProcessor`将动作请求发送到群集。这将在将元素批量发送到集群之前缓冲元素。 `BulkProcessor`一次执行一个批量请求，即正在进行的缓冲操作不会有两个并发刷新。

### Elasticsearch Sinks和Fault Tolerance

在启用Flink的检查点后，Flink Elasticsearch接收器保证至少一次将操作请求交付给Elasticsearch集群。它通过在检查点时等待BulkProcessor中的所有挂起的操作请求来实现这一点。这有效地确保了在触发检查点之前的所有请求都已被Elasticsearch成功地确认，然后才能继续处理发送到sink的更多记录。

有关检查点和容错的更多详细信息，请参见[容错文档](https://ci.apache.org/projects/flink/flink-docs-master/internals/stream_checkpointing.html)。

要使用容错Elasticsearch Sink，需要在执行环境中启用拓扑的检查点：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // checkpoint every 5000 msecs
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
env.enableCheckpointing(5000) // checkpoint every 5000 msecs
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**注意**：如果用户希望禁用刷新，可以通过在创建的**ElasticsearchSink**上调用**disableFlushOnCheckpoint\(\)**来实现 。请注意，这实际上意味着接收器不再提供任何强大的传送保证，即使启用了拓扑的检查点也是如此。
{% endhint %}

### 使用嵌入式节点进行通信（仅适用于Elasticsearch 1.x）

对于Elasticsearch版本1.x，还支持使用嵌入式节点进行通信。有关使用Elasticsearch与嵌入式节点和TransportClient进行通信的区别，请参见[此处](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/client.html)。

下面是如何使用嵌入式节点创建`ElasticsearchSink`而不是`TransportClient`：

{% tabs %}
{% tab title="Java" %}
```java
DataStream<String> input = ...;

Map<String, String> config = new HashMap<>;
// This instructs the sink to emit after every element, otherwise they would be buffered
config.put("bulk.flush.max.actions", "1");
config.put("cluster.name", "my-cluster-name");

input.addSink(new ElasticsearchSink<>(config, new ElasticsearchSinkFunction<String>() {
    public IndexRequest createIndexRequest(String element) {
        Map<String, String> json = new HashMap<>();
        json.put("data", element);
    
        return Requests.indexRequest()
                .index("my-index")
                .type("my-type")
                .source(json);
    }
    
    @Override
    public void process(String element, RuntimeContext ctx, RequestIndexer indexer) {
        indexer.add(createIndexRequest(element));
    }
}));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[String] = ...

val config = new java.util.HashMap[String, String]
config.put("bulk.flush.max.actions", "1")
config.put("cluster.name", "my-cluster-name")

input.addSink(new ElasticsearchSink(config, new ElasticsearchSinkFunction[String] {
  def createIndexRequest(element: String): IndexRequest = {
    val json = new java.util.HashMap[String, String]
    json.put("data", element)
    
    return Requests.indexRequest()
            .index("my-index")
            .type("my-type")
            .source(json)
  }
}))
```
{% endtab %}
{% endtabs %}

不同之处在于，现在我们不需要提供Elasticsearch节点的地址列表。

### 处理失败的Elasticsearch请求

Elasticsearch操作请求可能由于各种原因而失败，包括临时饱和的节点队列容量或索引的文档格式不正确。Flink Elasticsearch接收器允许用户通过简单地实现ActionRequestFailureHandler并将其提供给构造函数来指定如何处理请求失败。

以下是一个例子：

{% tabs %}
{% tab title="Java" %}
```java
DataStream<String> input = ...;

input.addSink(new ElasticsearchSink<>(
    config, transportAddresses,
    new ElasticsearchSinkFunction<String>() {...},
    new ActionRequestFailureHandler() {
        @Override
        void onFailure(ActionRequest action,
                Throwable failure,
                int restStatusCode,
                RequestIndexer indexer) throw Throwable {

            if (ExceptionUtils.containsThrowable(failure, EsRejectedExecutionException.class)) {
                // full queue; re-add document for indexing
                indexer.add(action);
            } else if (ExceptionUtils.containsThrowable(failure, ElasticsearchParseException.class)) {
                // malformed document; simply drop request without failing sink
            } else {
                // for all other failures, fail the sink
                // here the failure is simply rethrown, but users can also choose to throw custom exceptions
                throw failure;
            }
        }
}));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[String] = ...

input.addSink(new ElasticsearchSink(
    config, transportAddresses,
    new ElasticsearchSinkFunction[String] {...},
    new ActionRequestFailureHandler {
        @throws(classOf[Throwable])
        override def onFailure(ActionRequest action,
                Throwable failure,
                int restStatusCode,
                RequestIndexer indexer) {

            if (ExceptionUtils.containsThrowable(failure, EsRejectedExecutionException.class)) {
                // full queue; re-add document for indexing
                indexer.add(action)
            } else if (ExceptionUtils.containsThrowable(failure, ElasticsearchParseException.class)) {
                // malformed document; simply drop request without failing sink
            } else {
                // for all other failures, fail the sink
                // here the failure is simply rethrown, but users can also choose to throw custom exceptions
                throw failure
            }
        }
}))
```
{% endtab %}
{% endtabs %}

上面的示例将允许接收器重新添加由于队列容量饱和而失败的请求，并在不导致接收器失败的情况下删除具有格式错误文档的请求。对于所有其他失败，接收器都会失败。如果`ActionRequestFailureHandler`没有提供给构造函数，接收器将会因为任何类型的错误而失败。

请注意，`onFailure`仅在`BulkProcessor`内部完成所有退避重试尝试后才会发生故障 。默认情况下，`BulkProcessor`使用指数`backoff`重试最多8次尝试。有关内部行为及其`BulkProcessor`配置方式的更多信息，请参阅以下部分。

默认情况下，如果未提供失败处理程序，则接收器将使用`NoOpFailureHandler`对所有类型的异常都失败的接收器 。连接器还提供一种`RetryRejectedExecutionFailureHandler`实现，该实现始终重新添加由于队列容量饱和而失败的请求。

{% hint style="danger" %}
**重要提示:**在出现故障时将请求重新添加回内部**BulkProcessor**将导致更长的检查点，因为sink还需要等待在检查点时刷新重新添加的请求。例如，当使用`RetryRejectedExecutionFailureHandler`时，检查点将需要等待，直到Elasticsearch节点队列有足够的能力处理所有挂起的请求。这也意味着如果重新添加的请求从未成功，检查点将永远不会完成。
{% endhint %}

{% hint style="warning" %}
**对于Elasticsearch 1.x的故障处理**：对于Elasticsearch 1.x，匹配失败的类型是不可行的，因为不能通过旧版本的Java客户机api检索确切的类型\(因此，类型将是一般的异常，仅在失败消息中有所不同\)。在这种情况下，建议匹配提供的REST状态代码。
{% endhint %}

### 配置内部批量处理器

通过在提供的Map&lt;String,String&gt;中设置以下值，可以进一步为内部BulkProcessor的行为配置缓冲操作请求的刷新方式:

* **bulk.flush.max.actions**：刷新前缓冲的最大操作量。
* **bulk.flush.max.size.mb**：刷新前缓冲区的最大数据大小（以兆字节为单位）。
* **bulk.flush.interval.ms**：无论缓冲操作的数量或大小如何都要时间间隔刷新。

对于2.x及更高版本，还支持配置重试临时请求错误的方式：

* **bulk.flush.backoff.enable**：如果一个或多个刷新操作由于临时`EsRejectedExecutionException`异常而失败，则是否使用`backoff`延迟对刷新执行重试。
* **bulk.flush.backoff.type**：`backoff`延迟的类型，`CONSTANT`或者`EXPONENTIAL`
* **bulk.flush.backoff.delay**：`backoff`的延迟量。对于恒定的退避，这只是每次重试之间的延迟。对于指数退避，这是初始基本延迟。
* **bulk.flush.backoff.retries**：要尝试的`backoff`重试次数。

有关Elasticsearch的更多信息，请[访问此处](https://elastic.co/)。

## 将Elasticsearch Connector打包到Uber-Jar中

为了执行Flink程序，建议构建一个包含所有依赖项的所谓uber-jar（可执行jar）（有关详细信息，请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-master/dev/linking.html)）。

或者，您可以将连接器的jar文件放入Flink的`lib/`文件夹中，以使其在系统范围内可用，即对于所有正在运行的作业。

