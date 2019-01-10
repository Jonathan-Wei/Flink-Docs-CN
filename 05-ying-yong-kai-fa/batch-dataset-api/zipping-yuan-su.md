# Zipping元素

在某些算法中，可能需要为数据集元素分配唯一标识符。本文档展示了如何将[DataSetUtils](https://github.com/apache/flink/blob/master//flink-java/src/main/java/org/apache/flink/api/java/utils/DataSetUtils.java)用于此目的。

## 密集索引压缩

`zipWithIndex`为元素分配连续的标签，接收一个数据集作为输入并返回一个新的数据集\(唯一id，初始值\)2元组。这个过程需要两次传递，首先计数，然后标记元素，并且由于计数的同步，不能流水线化。替代的`zipWithUniqueId`以流水线方式工作，并且在唯一标签足够时是首选。 例如，以下代码:

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(2);
DataSet<String> in = env.fromElements("A", "B", "C", "D", "E", "F", "G", "H");

DataSet<Tuple2<Long, String>> result = DataSetUtils.zipWithIndex(in);

result.writeAsCsv(resultPath, "\n", ",");
env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.scala._

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
env.setParallelism(2)
val input: DataSet[String] = env.fromElements("A", "B", "C", "D", "E", "F", "G", "H")

val result: DataSet[(Long, String)] = input.zipWithIndex

result.writeAsCsv(resultPath, "\n", ",")
env.execute()
```
{% endtab %}

{% tab title="Python" %}
```python
from flink.plan.Environment import get_environment

env = get_environment()
env.set_parallelism(2)
input = env.from_elements("A", "B", "C", "D", "E", "F", "G", "H")

result = input.zip_with_index()

result.write_text(result_path)
env.execute()
```
{% endtab %}
{% endtabs %}

生成元组：\(0,G\), \(1,H\), \(2,A\), \(3,B\), \(4,C\), \(5,D\), \(6,E\), \(7,F\)

## 使用唯一标识符进行压缩

在许多情况下，可能不需要分配连续标签。 `zipWithUniqueId`以流水线方式工作，加快标签分配过程。此方法接收数据集作为输入，并返回`(unique id, initial value)`2元组的新数据集。例如，以下代码：

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(2);
DataSet<String> in = env.fromElements("A", "B", "C", "D", "E", "F", "G", "H");

DataSet<Tuple2<Long, String>> result = DataSetUtils.zipWithUniqueId(in);

result.writeAsCsv(resultPath, "\n", ",");
env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.scala._

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
env.setParallelism(2)
val input: DataSet[String] = env.fromElements("A", "B", "C", "D", "E", "F", "G", "H")

val result: DataSet[(Long, String)] = input.zipWithUniqueId

result.writeAsCsv(resultPath, "\n", ",")
env.execute()
```
{% endtab %}
{% endtabs %}

生成元组：\(0,G\), \(1,A\), \(2,H\), \(3,B\), \(5,C\), \(7,D\), \(9,E\), \(11,F\)  


