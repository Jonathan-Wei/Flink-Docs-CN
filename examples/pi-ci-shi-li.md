# 批次示例

## 运行一个示例



```text
./bin/flink run ./examples/batch/WordCount.jar
```

```text
./bin/flink run ./examples/batch/WordCount.jar --input /path/to/some/text/data --output /path/to/result
```

## WordCount

WordCount是大数据处理系统的“Hello World”。它计算文本集合中单词的频率。该算法分两步进行：首先，文本将文本分成单个单词。其次，对单词进行分组和计数。

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

该[WordCount示例](https://github.com/apache/flink/blob/master//flink-examples/flink-examples-batch/src/main/scala/org/apache/flink/examples/scala/wordcount/WordCount.scala)实现上述算法的输入参数：`--input <path> --output <path>`。作为测试数据，任何文本文件都可以。

## Page Rank

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

## Connected Components

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

