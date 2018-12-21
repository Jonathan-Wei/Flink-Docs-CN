# Scala API拓展

为了在Scala和Java API之间保持相当大的一致性，在批处理和流式传输的标准API中省略了一些允许Scala具有高级表达能力的功能。

如果您想_享受完整的Scala体验_，可以选择选择加入通过隐式转换增强Scala API的扩展。

要使用所有可用的扩展，您只需`import`为DataSet API 添加一个简单的扩展

```text
import org.apache.flink.api.scala.extensions._
```

或DataStream API

```text
import org.apache.flink.streaming.api.scala.extensions._
```

或者，您可以导入单独的扩展a-la-carte来只使用您喜欢的扩展。

## 接受部分功能

通常，DataSet和DataStream API都不接受匿名模式匹配函数来解构元组，案例类或集合，如下所示：

```scala
val data: DataSet[(Int, String, Double)] = // [...]
data.map {
  case (id, name, temperature) => // [...]
  // The previous line causes the following compilation error:
  // "The argument types of an anonymous function must be fully known. (SLS 8.5)"
}
```

此扩展在DataSet和DataStream Scala API中引入了新方法，这些方法在扩展API中具有一对一的对应关系。这些委托方法确实支持匿名模式匹配功能。

### **DataSet API**

<table>
  <thead>
    <tr>
      <th style="text-align:left">方法</th>
      <th style="text-align:left">原型</th>
      <th style="text-align:left">示例</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>mapWith</b>
      </td>
      <td style="text-align:left"><b>map (DataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>mapWith <b>{</b>
        </p>
        <p> <b>case</b> (<b>_</b>, value) <b>=></b> value<b>.</b>toString</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mapPartitionWith</b>
      </td>
      <td style="text-align:left"><b>mapPartition (DataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>mapPartitionWith <b>{</b>
        </p>
        <p> <b>case</b> head <b>#::</b>  <b>_</b>  <b>=></b> head</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>flatMapWith</b>
      </td>
      <td style="text-align:left"><b>flatMap (DataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>flatMapWith <b>{</b>
        </p>
        <p> <b>case</b> (<b>_</b>, name, visitTimes) <b>=></b> visitTimes<b>.</b>map(name <b>-></b>  <b>_</b>)</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>filterWith</b>
      </td>
      <td style="text-align:left"><b>filter (DataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>filterWith <b>{</b>
        </p>
        <p> <b>case</b>  <b>Train</b>(<b>_</b>, isOnTime) <b>=></b> isOnTime</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>reduceWith</b>
      </td>
      <td style="text-align:left"><b>reduce (DataSet, GroupedDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>reduceWith <b>{</b>
        </p>
        <p> <b>case</b> ((<b>_</b>, amount1), (<b>_</b>, amount2)) <b>=></b> amount1 <b>+</b> amount2</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>reduceGroupWith</b>
      </td>
      <td style="text-align:left"><b>reduceGroup (GroupedDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>reduceGroupWith <b>{</b>
        </p>
        <p> <b>case</b> id <b>#::</b> value <b>#::</b>  <b>_</b>  <b>=></b> id <b>-></b> value</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>groupingBy</b>
      </td>
      <td style="text-align:left"><b>groupBy (DataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>groupingBy <b>{</b>
        </p>
        <p> <b>case</b> (id, <b>_</b>, <b>_</b>) <b>=></b> id</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>sortGroupWith</b>
      </td>
      <td style="text-align:left"><b>sortGroup (GroupedDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>grouped<b>.</b>sortGroupWith(<b>Order.ASCENDING</b>) <b>{</b>
        </p>
        <p> <b>case</b>  <b>House</b>(<b>_</b>, value) <b>=></b> value</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>combineGroupWith</b>
      </td>
      <td style="text-align:left"><b>combineGroup (GroupedDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>grouped<b>.</b>combineGroupWith <b>{</b>
        </p>
        <p> <b>case</b> header <b>#::</b> amounts <b>=></b> amounts<b>.</b>sum</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>projecting</b>
      </td>
      <td style="text-align:left"><b>apply (JoinDataSet, CrossDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data1<b>.</b>join(data2)<b>.</b>
        </p>
        <p>whereClause(<b>case</b> (pk, <b>_</b>) <b>=></b> pk)<b>.</b>
        </p>
        <p>isEqualTo(<b>case</b> (<b>_</b>, fk) <b>=></b> fk)<b>.</b>
        </p>
        <p>projecting <b>{</b>
        </p>
        <p> <b>case</b> ((pk, tx), (products, fk)) <b>=></b> tx <b>-></b> products</p>
        <p> <b>}</b>
        </p>
        <p>data1<b>.</b>cross(data2)<b>.</b>projecting <b>{</b>
        </p>
        <p> <b>case</b> ((a, <b>_</b>), (<b>_</b>, b) <b>=></b> a <b>-></b> b</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>projecting</b>
      </td>
      <td style="text-align:left"><b>apply (CoGroupDataSet)</b>
      </td>
      <td style="text-align:left">
        <p>data1<b>.</b>coGroup(data2)<b>.</b>
        </p>
        <p>whereClause(<b>case</b> (pk, <b>_</b>) <b>=></b> pk)<b>.</b>
        </p>
        <p>isEqualTo(<b>case</b> (<b>_</b>, fk) <b>=></b> fk)<b>.</b>
        </p>
        <p>projecting <b>{</b>
        </p>
        <p> <b>case</b> (head1 <b>#::</b>  <b>_</b>, head2 <b>#::</b>  <b>_</b>) <b>=></b> head1 <b>-></b> head2</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
  </tbody>
</table>### DataStream API

<table>
  <thead>
    <tr>
      <th style="text-align:left">方法</th>
      <th style="text-align:left">原型</th>
      <th style="text-align:left">示例</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>mapWith</b>
      </td>
      <td style="text-align:left"><b>map (DataStream</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>mapWith <b>{</b>
        </p>
        <p> <b>case</b> (<b>_</b>, value) <b>=></b> value<b>.</b>toString</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mapPartitionWith</b>
      </td>
      <td style="text-align:left"><b>mapPartition (DataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>mapPartitionWith <b>{</b>
        </p>
        <p> <b>case</b> head <b>#::</b>  <b>_</b>  <b>=></b> head</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>flatMapWith</b>
      </td>
      <td style="text-align:left"><b>flatMap (DataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>flatMapWith <b>{</b>
        </p>
        <p> <b>case</b> (<b>_</b>, name, visits) <b>=></b> visits<b>.</b>map(name <b>-></b>  <b>_</b>)</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>filterWith</b>
      </td>
      <td style="text-align:left"><b>filter (DataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>filterWith <b>{</b>
        </p>
        <p> <b>case</b>  <b>Train</b>(<b>_</b>, isOnTime) <b>=></b> isOnTime</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>keyingBy</b>
      </td>
      <td style="text-align:left"><b>keyBy (DataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>keyingBy <b>{</b>
        </p>
        <p> <b>case</b> (id, <b>_</b>, <b>_</b>) <b>=></b> id</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mapWith</b>
      </td>
      <td style="text-align:left"><b>map (ConnectedDataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>mapWith(</p>
        <p>map1 <b>=</b>  <b>case</b> (<b>_</b>, value) <b>=></b> value<b>.</b>toString,</p>
        <p>map2 <b>=</b>  <b>case</b> (<b>_</b>, <b>_</b>, value, <b>_</b>) <b>=></b> value <b>+</b> 1</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>flatMapWith</b>
      </td>
      <td style="text-align:left"><b>flatMap (ConnectedDataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>flatMapWith(</p>
        <p>flatMap1 <b>=</b>  <b>case</b> (<b>_</b>, json) <b>=></b> parse(json),</p>
        <p>flatMap2 <b>=</b>  <b>case</b> (<b>_</b>, <b>_</b>, json, <b>_</b>) <b>=></b> parse(json)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>keyingBy</b>
      </td>
      <td style="text-align:left"><b>keyBy (ConnectedDataStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>keyingBy(</p>
        <p>key1 <b>=</b>  <b>case</b> (<b>_</b>, timestamp) <b>=></b> timestamp,</p>
        <p>key2 <b>=</b>  <b>case</b> (id, <b>_</b>, <b>_</b>) <b>=></b> id</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>reduceWith</b>
      </td>
      <td style="text-align:left"><b>reduce (KeyedStream, WindowedStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>reduceWith <b>{</b>
        </p>
        <p> <b>case</b> ((<b>_</b>, sum1), (<b>_</b>, sum2) <b>=></b> sum1 <b>+</b> sum2</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>foldWith</b>
      </td>
      <td style="text-align:left"><b>fold (KeyedStream, WindowedStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>foldWith(<b>User</b>(bought <b>=</b> 0)) <b>{</b>
        </p>
        <p> <b>case</b> (<b>User</b>(b), (<b>_</b>, items)) <b>=></b>  <b>User</b>(b <b>+</b> items<b>.</b>size)</p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>applyWith</b>
      </td>
      <td style="text-align:left"><b>apply (WindowedStream)</b>
      </td>
      <td style="text-align:left">
        <p>data<b>.</b>applyWith(0)(</p>
        <p>foldFunction <b>=</b>  <b>case</b> (sum, amount) <b>=></b> sum <b>+</b> amount</p>
        <p>windowFunction <b>=</b>  <b>case</b> (k, w, sum) <b>=></b> // [...]</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>projecting</b>
      </td>
      <td style="text-align:left"><b>apply (JoinedStream)</b>
      </td>
      <td style="text-align:left">
        <p>data1<b>.</b>join(data2)<b>.</b>
        </p>
        <p>whereClause(<b>case</b> (pk, <b>_</b>) <b>=></b> pk)<b>.</b>
        </p>
        <p>isEqualTo(<b>case</b> (<b>_</b>, fk) <b>=></b> fk)<b>.</b>
        </p>
        <p>projecting <b>{</b>
        </p>
        <p> <b>case</b> ((pk, tx), (products, fk)) <b>=></b> tx <b>-></b> products</p>
        <p> <b>}</b>
        </p>
      </td>
    </tr>
  </tbody>
</table>有关每种方法的语义的更多信息，请参阅 [DataSet](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html)和[DataStream](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html) API文档。

要完全使用此扩展，您可以添加以下`import`：

```java
import org.apache.flink.api.scala.extensions.acceptPartialFunctions
```

对于DataSet扩展和

```java
import org.apache.flink.streaming.api.scala.extensions.acceptPartialFunctions
```

以下代码段显示了如何一起使用这些扩展方法的最小示例（使用DataSet API）：

```scala
object Main {
  import org.apache.flink.api.scala.extensions._
  case class Point(x: Double, y: Double)
  def main(args: Array[String]): Unit = {
    val env = ExecutionEnvironment.getExecutionEnvironment
    val ds = env.fromElements(Point(1, 2), Point(3, 4), Point(5, 6))
    ds.filterWith {
      case Point(x, _) => x > 1
    }.reduceWith {
      case (Point(x1, y1), (Point(x2, y2))) => Point(x1 + y1, x2 + y2)
    }.mapWith {
      case Point(x, y) => (x, y)
    }.flatMapWith {
      case (x, y) => Seq("x" -> x, "y" -> y)
    }.groupingBy {
      case (id, value) => id
    }
  }
}
```

