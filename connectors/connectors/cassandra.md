# Cassandra

此连接器提供将数据写入[Apache Cassandra](https://cassandra.apache.org/)数据库的接收器。

要使用此连接器，请将以下依赖项添加到项目中：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-cassandra_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

{% hint style="info" %}
请注意，流连接器目前不是二进制发行版的一部分。在[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking.html)查看如何与之关联以进行集群执行。
{% endhint %}

## 安装Apache Cassandra

有多种方法可以在本地计算机上启动Cassandra实例：

1. 按照[Cassandra入门页面上](http://cassandra.apache.org/doc/latest/getting_started/index.html)的说明进行操作。
2. 从[官方Docker Repository](https://hub.docker.com/_/cassandra/)启动一个运行Cassandra的容器

## Cassandra Sinks

### 配置

Flink的Cassandra接收器是使用静态`CassandraSink.addSink(DataStream)`方法创建的。这个方法返回一个CassandraSinkBuilder，它提供了进一步配置接收器的方法，最后是\`build\(\)\`一个接收器实例。

可以使用以下配置方法：

1. _setQuery（字符串查询）_
   * 设置为接收器接收的每个记录执行的upsert查询。
   * 该查询在内部被视为CQL语句。
   * **DO**设置upsert查询以处理**Tuple**数据类型。
   * **DO NOT**设置查询以处理**POJO**数据类型。
2. _setClusterBuilder\(\)_
   * 将用于配置与cassandra的连接的集群构建器设置更为复杂的设置，例如一致性级别，重试策略等。
3. _setHost（String host \[，int port\]）_
   * setClusterBuilder\(\)的简单版本，包含连接到Cassandra实例的主机/端口信息
4. _setMapperOptions（MapperOptions选项）_
   * 设置用于配置DataStax ObjectMapper的映射器选项。
   * 仅在处理**POJO**数据类型时适用。
5. _setMaxConcurrentRequests\(int maxConcurrentRequests,Duration timeout\)_
   * 设置允许的最大并发请求数，以及获取执行许可的超时。
   * 仅在未配置**enableWriteAheadLog\(\)**时适用。
6. _enableWriteAheadLog\(\[CheckpointCommitter committer\]\)_
   * 一个**可选的**设置
   * 允许对非确定性算法进行一次性处理。
7. _setFailureHandler\(\[CassandraFailureHandler failureHandler\]\)_
   * 一个**可选的**设置
   * 设置自定义失败处理程序。
8. _build\(\)_
   * 完成配置并构造CassandraSink实例。

### 预写日志

检查点提交者在某些资源中存储有关已完成检查点的其他信息。此信息用于防止在发生故障时完整重播最后完成的检查点。您可以使用a `CassandraCommitter`将它们存储在cassandra的单独表中。请注意，Flink不会清理此表。

如果查询是幂等的（意味着它可以多次应用而不改变结果）并且启用了检查点，则Flink可以提供一次性保证。如果发生故障，将完全重播失败的检查点。

此外，对于非确定性程序，必须启用预写日志。对于这样的程序，重放检查点可能与先前的尝试完全不同，这可能使数据库处于不一致状态，因为可能已经写入了第一次尝试的部分。预写日志保证重放的检查点与第一次尝试相同。请注意，启用此功能会对延迟产生负面影响。

{% hint style="danger" %}
**注意**：预写日志功能目前是实验性的。在许多情况下，使用连接器而不启用它就足够了。请将问题报告给开发邮件列表。
{% endhint %}

### 检查点与容错

启用检查点后，Cassandra Sink保证至少一次向C \*实例传递操作请求。

有关[检查点文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/checkpointing.html)和[容错保证文档的](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/guarantees.html)更多详细信息

## 示例

Cassandra接收器当前支持Tuple和POJO数据类型，Flink自动检测使用哪种类型的输入。有关那些流数据类型的一般用例，请参阅[支持的数据类型](https://ci.apache.org/projects/flink/flink-docs-master/dev/api_concepts.html)。我们分别针对Pojo和Tuple数据类型展示了基于[SocketWindowWordCount的](https://github.com/apache/flink/blob/master/flink-examples/flink-examples-streaming/src/main/java/org/apache/flink/streaming/examples/socket/SocketWindowWordCount.java)两个实现。

在所有这些例子中，我们假设已经创建了相关的Keyspace `example`和Table `wordcount`。

{% tabs %}
{% tab title="CQL" %}
```sql
CREATE KEYSPACE IF NOT EXISTS example
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'};
CREATE TABLE IF NOT EXISTS example.wordcount (
    word text,
    count bigint,
    PRIMARY KEY(word)
    );
```
{% endtab %}
{% endtabs %}

### 用于流式元组数据类型的Cassandra Sink示例

在将结果与Java / Scala Tuple数据类型存储到Cassandra接收器时，需要设置CQL upsert语句（通过setQuery（'stmt'））将每条记录保存回数据库。将upsert查询缓存为`PreparedStatement`，每个Tuple元素都转换为语句的参数。

有关细节`PreparedStatement`和`BoundStatement`信息，请访问[DataStax Java驱动程序手册](https://docs.datastax.com/en/developer/java-driver/2.1/manual/statements/prepared/)

{% tabs %}
{% tab title="Java" %}
```java
// get the execution environment
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// get input data by connecting to the socket
DataStream<String> text = env.socketTextStream(hostname, port, "\n");

// parse the data, group it, window it, and aggregate the counts
DataStream<Tuple2<String, Long>> result = text
        .flatMap(new FlatMapFunction<String, Tuple2<String, Long>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Long>> out) {
                // normalize and split the line
                String[] words = value.toLowerCase().split("\\s");

                // emit the pairs
                for (String word : words) {
                    //Do not accept empty word, since word is defined as primary key in C* table
                    if (!word.isEmpty()) {
                        out.collect(new Tuple2<String, Long>(word, 1L));
                    }
                }
            }
        })
        .keyBy(0)
        .timeWindow(Time.seconds(5))
        .sum(1);

CassandraSink.addSink(result)
        .setQuery("INSERT INTO example.wordcount(word, count) values (?, ?);")
        .setHost("127.0.0.1")
        .build();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

// get input data by connecting to the socket
val text: DataStream[String] = env.socketTextStream(hostname, port, '\n')

// parse the data, group it, window it, and aggregate the counts
val result: DataStream[(String, Long)] = text
  // split up the lines in pairs (2-tuples) containing: (word,1)
  .flatMap(_.toLowerCase.split("\\s"))
  .filter(_.nonEmpty)
  .map((_, 1L))
  // group by the tuple field "0" and sum up tuple field "1"
  .keyBy(0)
  .timeWindow(Time.seconds(5))
  .sum(1)

CassandraSink.addSink(result)
  .setQuery("INSERT INTO example.wordcount(word, count) values (?, ?);")
  .setHost("127.0.0.1")
  .build()

result.print().setParallelism(1)
```
{% endtab %}
{% endtabs %}

### 用于流式传输POJO数据类型的Cassandra Sink示例

一个流式POJO数据类型并将相同的POJO实体存储回Cassandra的示例。此外，此POJO实现需要遵循[DataStax Java驱动程序手册](http://docs.datastax.com/en/developer/java-driver/2.1/manual/object_mapper/creating/)来注释类，因为此实体的每个字段都使用DataStax Java Driver `com.datastax.driver.mapping.Mapper`类映射到指定表的关联列。

可以通过放置在Pojo类中的字段声明上的注释来定义每个表列的映射。有关映射的详细信息，请参阅[有关映射类](http://docs.datastax.com/en/developer/java-driver/3.1/manual/object_mapper/creating/)和[CQL数据类型](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cql_data_types_c.html)[定义的](http://docs.datastax.com/en/developer/java-driver/3.1/manual/object_mapper/creating/)CQL文档

```java
// get the execution environment
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// get input data by connecting to the socket
DataStream<String> text = env.socketTextStream(hostname, port, "\n");

// parse the data, group it, window it, and aggregate the counts
DataStream<WordCount> result = text
        .flatMap(new FlatMapFunction<String, WordCount>() {
            public void flatMap(String value, Collector<WordCount> out) {
                // normalize and split the line
                String[] words = value.toLowerCase().split("\\s");

                // emit the pairs
                for (String word : words) {
                    if (!word.isEmpty()) {
                        //Do not accept empty word, since word is defined as primary key in C* table
                        out.collect(new WordCount(word, 1L));
                    }
                }
            }
        })
        .keyBy("word")
        .timeWindow(Time.seconds(5))

        .reduce(new ReduceFunction<WordCount>() {
            @Override
            public WordCount reduce(WordCount a, WordCount b) {
                return new WordCount(a.getWord(), a.getCount() + b.getCount());
            }
        });

CassandraSink.addSink(result)
        .setHost("127.0.0.1")
        .setMapperOptions(() -> new Mapper.Option[]{Mapper.Option.saveNullFields(true)})
        .build();


@Table(keyspace = "example", name = "wordcount")
public class WordCount {

    @Column(name = "word")
    private String word = "";

    @Column(name = "count")
    private long count = 0;

    public WordCount() {}

    public WordCount(String word, long count) {
        this.setWord(word);
        this.setCount(count);
    }

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public long getCount() {
        return count;
    }

    public void setCount(long count) {
        this.count = count;
    }

    @Override
    public String toString() {
        return getWord() + " : " + getCount();
    }
}
```

