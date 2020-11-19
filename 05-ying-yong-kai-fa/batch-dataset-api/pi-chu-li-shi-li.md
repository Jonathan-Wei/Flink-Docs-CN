# 批处理示例

以下示例程序展示了Flink从简单的单词计数到图形算法的不同应用。这些代码示例展示了[Flink的DataSet API](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/batch/index.html)的用法。

以下和更多示例的完整源代码可在Flink源存储库的[flink-examples-batch](https://github.com/apache/flink/blob/master/flink-examples/flink-examples-batch)模块中找到。

## 运行一个例子

为了运行Flink示例，我们假设您有一个正在运行的Flink实例。导航中的“快速入门”和“设置”选项卡描述了启动Flink的各种方法。

最简单的方法是运行`./bin/start-cluster.sh`，默认情况下，启动带有一个JobManager和一个TaskManager的本地集群。

每个Flink二进制发行版都包含一个`examples`目录，其中包含此页面上每个示例的jar文件。

要运行WordCount示例，请发出以下命令：

```text
./bin/flink run ./examples/batch/WordCount.jar
```

其他示例可以以类似方式启动。

请注意，许多示例使用内置数据运行时都没有传递任何参数。要使用实际数据运行WordCount，必须将路径传递给数据：

```text
./bin/flink run ./examples/batch/WordCount.jar --input /path/to/some/text/data --output /path/to/result
```

请注意，非本地文件系统需要模式前缀，例如`hdfs://`。

## Word Count

WordCount是大数据处理系统的“ Hello World”。它计算文本集合中单词的频率。该算法分两个步骤工作：首先，将文本拆分为单个单词。其次，对单词进行分组和计数。

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<String> text = env.readTextFile("/path/to/file");

DataSet<Tuple2<String, Integer>> counts =
        // split up the lines in pairs (2-tuples) containing: (word,1)
        text.flatMap(new Tokenizer())
        // group by the tuple field "0" and sum up tuple field "1"
        .groupBy(0)
        .sum(1);

counts.writeAsCsv(outputPath, "\n", " ");

// User-defined functions
public static class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {

    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
        // normalize and split the line
        String[] tokens = value.toLowerCase().split("\\W+");

        // emit the pairs
        for (String token : tokens) {
            if (token.length() > 0) {
                out.collect(new Tuple2<String, Integer>(token, 1));
            }   
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment

// get input data
val text = env.readTextFile("/path/to/file")

val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
  .map { (_, 1) }
  .groupBy(0)
  .sum(1)

counts.writeAsCsv(outputPath, "\n", " ")
```
{% endtab %}
{% endtabs %}

WordCount示例使用输入参数实现了上述算法:——input ——output 。作为测试数据，任何文本文件都可以。

## PageRank

PageRank算法计算链接定义的图中页面的“重要性”，这些链接从一个页面指向另一个页面。这是一种迭代图算法，这意味着它反复应用相同的计算。在每次迭代中，每个页面都会将其当前等级分配给所有邻居，并将其新等级计算为从邻居收到的等级的总和。PageRank算法已由Google搜索引擎广泛使用，该引擎使用网页的重要性对搜索查询的结果进行排名。

在此简单示例中，使用[批量迭代](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/batch/iterations.html)和固定数量的迭代来实现PageRank 。

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// read the pages and initial ranks by parsing a CSV file
DataSet<Tuple2<Long, Double>> pagesWithRanks = env.readCsvFile(pagesInputPath)
						   .types(Long.class, Double.class)

// the links are encoded as an adjacency list: (page-id, Array(neighbor-ids))
DataSet<Tuple2<Long, Long[]>> pageLinkLists = getLinksDataSet(env);

// set iterative data set
IterativeDataSet<Tuple2<Long, Double>> iteration = pagesWithRanks.iterate(maxIterations);

DataSet<Tuple2<Long, Double>> newRanks = iteration
        // join pages with outgoing edges and distribute rank
        .join(pageLinkLists).where(0).equalTo(0).flatMap(new JoinVertexWithEdgesMatch())
        // collect and sum ranks
        .groupBy(0).sum(1)
        // apply dampening factor
        .map(new Dampener(DAMPENING_FACTOR, numPages));

DataSet<Tuple2<Long, Double>> finalPageRanks = iteration.closeWith(
        newRanks,
        newRanks.join(iteration).where(0).equalTo(0)
        // termination condition
        .filter(new EpsilonFilter()));

finalPageRanks.writeAsCsv(outputPath, "\n", " ");

// User-defined functions

public static final class JoinVertexWithEdgesMatch
                    implements FlatJoinFunction<Tuple2<Long, Double>, Tuple2<Long, Long[]>,
                                            Tuple2<Long, Double>> {

    @Override
    public void join(<Tuple2<Long, Double> page, Tuple2<Long, Long[]> adj,
                        Collector<Tuple2<Long, Double>> out) {
        Long[] neighbors = adj.f1;
        double rank = page.f1;
        double rankToDistribute = rank / ((double) neigbors.length);

        for (int i = 0; i < neighbors.length; i++) {
            out.collect(new Tuple2<Long, Double>(neighbors[i], rankToDistribute));
        }
    }
}

public static final class Dampener implements MapFunction<Tuple2<Long,Double>, Tuple2<Long,Double>> {
    private final double dampening, randomJump;

    public Dampener(double dampening, double numVertices) {
        this.dampening = dampening;
        this.randomJump = (1 - dampening) / numVertices;
    }

    @Override
    public Tuple2<Long, Double> map(Tuple2<Long, Double> value) {
        value.f1 = (value.f1 * dampening) + randomJump;
        return value;
    }
}

public static final class EpsilonFilter
                implements FilterFunction<Tuple2<Tuple2<Long, Double>, Tuple2<Long, Double>>> {

    @Override
    public boolean filter(Tuple2<Tuple2<Long, Double>, Tuple2<Long, Double>> value) {
        return Math.abs(value.f0.f1 - value.f1.f1) > EPSILON;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
// User-defined types
case class Link(sourceId: Long, targetId: Long)
case class Page(pageId: Long, rank: Double)
case class AdjacencyList(sourceId: Long, targetIds: Array[Long])

// set up execution environment
val env = ExecutionEnvironment.getExecutionEnvironment

// read the pages and initial ranks by parsing a CSV file
val pages = env.readCsvFile[Page](pagesInputPath)

// the links are encoded as an adjacency list: (page-id, Array(neighbor-ids))
val links = env.readCsvFile[Link](linksInputPath)

// assign initial ranks to pages
val pagesWithRanks = pages.map(p => Page(p, 1.0 / numPages))

// build adjacency list from link input
val adjacencyLists = links
  // initialize lists
  .map(e => AdjacencyList(e.sourceId, Array(e.targetId)))
  // concatenate lists
  .groupBy("sourceId").reduce {
  (l1, l2) => AdjacencyList(l1.sourceId, l1.targetIds ++ l2.targetIds)
  }

// start iteration
val finalRanks = pagesWithRanks.iterateWithTermination(maxIterations) {
  currentRanks =>
    val newRanks = currentRanks
      // distribute ranks to target pages
      .join(adjacencyLists).where("pageId").equalTo("sourceId") {
        (page, adjacent, out: Collector[Page]) =>
        for (targetId <- adjacent.targetIds) {
          out.collect(Page(targetId, page.rank / adjacent.targetIds.length))
        }
      }
      // collect ranks and sum them up
      .groupBy("pageId").aggregate(SUM, "rank")
      // apply dampening factor
      .map { p =>
        Page(p.pageId, (p.rank * DAMPENING_FACTOR) + ((1 - DAMPENING_FACTOR) / numPages))
      }

    // terminate if no rank update was significant
    val termination = currentRanks.join(newRanks).where("pageId").equalTo("pageId") {
      (current, next, out: Collector[Int]) =>
        // check for significant update
        if (math.abs(current.rank - next.rank) > EPSILON) out.collect(1)
    }

    (newRanks, termination)
}

val result = finalRanks

// emit result
result.writeAsCsv(outputPath, "\n", " ")
```
{% endtab %}
{% endtabs %}

所述[的PageRank程序](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/graph/PageRankBasic.scala)实现上述实施例。它需要以下参数才能运行：`--pages <path> --links <path> --output <path> --numPages <n> --iterations <n>`。

输入文件是纯文本文件，并且必须采用以下格式：

* 以（long）ID表示的页面，以换行符分隔。
  * 例如，`"1\n2\n12\n42\n63\n"`给出五个ID为1、2、12、42和63的页面。
* 链接表示为成对的页面ID，用空格字符分隔。链接由换行符分隔：
  * 例如，`"1 2\n2 12\n1 12\n42 63\n"`给出四个（定向）链接（1）-&gt;（2），（2）-&gt;（12），（1）-&gt;（12）和（42）-&gt;（63）。

对于这种简单的实现，要求每个页面至少具有一个入站链接和一个出站链接（页面可以指向自身）。

## 连通分支（Connected Components）

“Connected Components”算法通过为同一连通部分中的所有顶点分配相同的组件ID来识别更大图中的连通部分。与PageRank相似，Connected Components是一种迭代算法。在每个步骤中，每个顶点都将其当前组件ID传播到其所有邻居。如果顶点小于自己的组件ID，则该顶点接受邻居的组件ID。

此实现使用[增量迭代](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/batch/iterations.html)：未更改其组件ID的顶点不参与下一步。这会产生更好的性能，因为后面的迭代通常只处理几个离群点。

{% tabs %}
{% tab title="Java" %}
```java
// read vertex and edge data
DataSet<Long> vertices = getVertexDataSet(env);
DataSet<Tuple2<Long, Long>> edges = getEdgeDataSet(env).flatMap(new UndirectEdge());

// assign the initial component IDs (equal to the vertex ID)
DataSet<Tuple2<Long, Long>> verticesWithInitialId = vertices.map(new DuplicateValue<Long>());

// open a delta iteration
DeltaIteration<Tuple2<Long, Long>, Tuple2<Long, Long>> iteration =
        verticesWithInitialId.iterateDelta(verticesWithInitialId, maxIterations, 0);

// apply the step logic:
DataSet<Tuple2<Long, Long>> changes = iteration.getWorkset()
        // join with the edges
        .join(edges).where(0).equalTo(0).with(new NeighborWithComponentIDJoin())
        // select the minimum neighbor component ID
        .groupBy(0).aggregate(Aggregations.MIN, 1)
        // update if the component ID of the candidate is smaller
        .join(iteration.getSolutionSet()).where(0).equalTo(0)
        .flatMap(new ComponentIdFilter());

// close the delta iteration (delta and new workset are identical)
DataSet<Tuple2<Long, Long>> result = iteration.closeWith(changes, changes);

// emit result
result.writeAsCsv(outputPath, "\n", " ");

// User-defined functions

public static final class DuplicateValue<T> implements MapFunction<T, Tuple2<T, T>> {

    @Override
    public Tuple2<T, T> map(T vertex) {
        return new Tuple2<T, T>(vertex, vertex);
    }
}

public static final class UndirectEdge
                    implements FlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Long>> {
    Tuple2<Long, Long> invertedEdge = new Tuple2<Long, Long>();

    @Override
    public void flatMap(Tuple2<Long, Long> edge, Collector<Tuple2<Long, Long>> out) {
        invertedEdge.f0 = edge.f1;
        invertedEdge.f1 = edge.f0;
        out.collect(edge);
        out.collect(invertedEdge);
    }
}

public static final class NeighborWithComponentIDJoin
                implements JoinFunction<Tuple2<Long, Long>, Tuple2<Long, Long>, Tuple2<Long, Long>> {

    @Override
    public Tuple2<Long, Long> join(Tuple2<Long, Long> vertexWithComponent, Tuple2<Long, Long> edge) {
        return new Tuple2<Long, Long>(edge.f1, vertexWithComponent.f1);
    }
}

public static final class ComponentIdFilter
                    implements FlatMapFunction<Tuple2<Tuple2<Long, Long>, Tuple2<Long, Long>>,
                                            Tuple2<Long, Long>> {

    @Override
    public void flatMap(Tuple2<Tuple2<Long, Long>, Tuple2<Long, Long>> value,
                        Collector<Tuple2<Long, Long>> out) {
        if (value.f0.f1 < value.f1.f1) {
            out.collect(value.f0);
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
// set up execution environment
val env = ExecutionEnvironment.getExecutionEnvironment

// read vertex and edge data
// assign the initial components (equal to the vertex id)
val vertices = getVerticesDataSet(env).map { id => (id, id) }

// undirected edges by emitting for each input edge the input edges itself and an inverted
// version
val edges = getEdgesDataSet(env).flatMap { edge => Seq(edge, (edge._2, edge._1)) }

// open a delta iteration
val verticesWithComponents = vertices.iterateDelta(vertices, maxIterations, Array(0)) {
  (s, ws) =>

    // apply the step logic: join with the edges
    val allNeighbors = ws.join(edges).where(0).equalTo(0) { (vertex, edge) =>
      (edge._2, vertex._2)
    }

    // select the minimum neighbor
    val minNeighbors = allNeighbors.groupBy(0).min(1)

    // update if the component of the candidate is smaller
    val updatedComponents = minNeighbors.join(s).where(0).equalTo(0) {
      (newVertex, oldVertex, out: Collector[(Long, Long)]) =>
        if (newVertex._2 < oldVertex._2) out.collect(newVertex)
    }

    // delta and new workset are identical
    (updatedComponents, updatedComponents)
}

verticesWithComponents.writeAsCsv(outputPath, "\n", " ")
```
{% endtab %}
{% endtabs %}

上面的示例实现了ConnectedComponents算法程序。它需要以下参数来运行:——vertices ——edges ——output ——iterations 。

输入文件是纯文本文件，并且必须采用以下格式：

* 顶点表示为ID，并由换行符分隔。
  * 例如，`"1\n2\n12\n42\n63\n"`给出具有（1），（2），（12），（42）和（63）的五个顶点。
* 边被表示为成对的顶点ID，用空格字符分隔。边缘由换行符分隔：
  * 例如，`"1 2\n2 12\n1 12\n42 63\n"`给出四个（无向）链接（1）-（2），（2）-（12），（1）-（12）和（42）-（63）。

