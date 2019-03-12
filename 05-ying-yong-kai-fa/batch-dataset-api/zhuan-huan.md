# 转换

本文档深入介绍了DataSet上的可用转换。有关Flink Java API的一般介绍，请参阅[编程指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html)。

有关在具有密集索引的数据集中压缩元素，请参阅[Zip元素指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/zip_elements_guide.html)。

## Map

Map转换在DataSet的每个元素上应用用户定义的map函数。它实现了一对一的映射，也就是说，函数必须返回一个元素。

以下代码将Integer对的DataSet转换为Integers的DataSet：

{% tabs %}
{% tab title="Java" %}
```java
// MapFunction that adds two integer values
public class IntAdder implements MapFunction<Tuple2<Integer, Integer>, Integer> {
  @Override
  public Integer map(Tuple2<Integer, Integer> in) {
    return in.f0 + in.f1;
  }
}

// [...]
DataSet<Tuple2<Integer, Integer>> intPairs = // [...]
DataSet<Integer> intSums = intPairs.map(new IntAdder());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val intPairs: DataSet[(Int, Int)] = // [...]
val intSums = intPairs.map { pair => pair._1 + pair._2 }
```
{% endtab %}

{% tab title="Python" %}
```python
words = lines.flat_map(lambda x,c: [line.split() for line in x])
```
{% endtab %}
{% endtabs %}

## FlatMap

FlatMap转换在DataSet的每个元素上应用用户定义的`FlatMap`函数。`map`函数的这种变体可以为每个输入元素返回任意多个结果元素（包括none）。

以下代码将文本行的DataSet转换为单词的DataSet：

{% tabs %}
{% tab title="Java" %}
```java
public class PartitionCounter implements MapPartitionFunction<String, Long> {

  public void mapPartition(Iterable<String> values, Collector<Long> out) {
    long c = 0;
    for (String s : values) {
      c++;
    }
    out.collect(c);
  }
}

// [...]
DataSet<String> textLines = // [...]
DataSet<Long> counts = textLines.mapPartition(new PartitionCounter());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val textLines: DataSet[String] = // [...]
// Some is required because the return value must be a Collection.
// There is an implicit conversion from Option to a Collection.
val counts = texLines.mapPartition { in => Some(in.size) }
```
{% endtab %}

{% tab title="Python" %}
```python
counts = lines.map_partition(lambda x,c: [sum(1 for _ in x)])
```
{% endtab %}
{% endtabs %}

## MapPartition

`MapPartition`在单个函数调用中转换并行分区。`map-partition`函数将分区作为`Iterable`获取，并且可以生成任意数量的结果值。每个分区中的元素数量取决于并行度和先前的操作。

以下代码将文本行的DataSet转换为每个分区的计数数据集：

{% tabs %}
{% tab title="Java" %}
```java
public class PartitionCounter implements MapPartitionFunction<String, Long> {

  public void mapPartition(Iterable<String> values, Collector<Long> out) {
    long c = 0;
    for (String s : values) {
      c++;
    }
    out.collect(c);
  }
}

// [...]
DataSet<String> textLines = // [...]
DataSet<Long> counts = textLines.mapPartition(new PartitionCounter());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val textLines: DataSet[String] = // [...]
// Some is required because the return value must be a Collection.
// There is an implicit conversion from Option to a Collection.
val counts = texLines.mapPartition { in => Some(in.size) }
```
{% endtab %}

{% tab title="Python" %}
```python
counts = lines.map_partition(lambda x,c: [sum(1 for _ in x)])
```
{% endtab %}
{% endtabs %}

## Filter

`Filter`转换在DataSet的每个元素上应用用户定义的`filter`函数，并仅保留函数返回的元素`true`。

以下代码从DataSet中删除所有小于零的整数：

{% tabs %}
{% tab title="Java" %}
```java
// FilterFunction that filters out all Integers smaller than zero.
public class NaturalNumberFilter implements FilterFunction<Integer> {
  @Override
  public boolean filter(Integer number) {
    return number >= 0;
  }
}

// [...]
DataSet<Integer> intNumbers = // [...]
DataSet<Integer> naturalNumbers = intNumbers.filter(new NaturalNumberFilter());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val intNumbers: DataSet[Int] = // [...]
val naturalNumbers = intNumbers.filter { _ > 0 }
```
{% endtab %}

{% tab title="Python" %}
```python
 naturalNumbers = intNumbers.filter(lambda x: x > 0)
```
{% endtab %}
{% endtabs %}

**重要信息：**系统假定该函数不会修改应用谓词的元素。违反此假设可能会导致错误的结果。

## Tuple DataSet投影

`Project`转换删除或移动元组数据集的Tuple字段。该`project(int...)`方法选择应由其索引保留的元组字段，并在输出元组中定义它们的顺序。

不需要定义用户功能。

以下代码显示了在DataSet上应用`project`转换的不同方法：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<Integer, Double, String>> in = // [...]
// converts Tuple3<Integer, Double, String> into Tuple2<String, Integer>
DataSet<Tuple2<String, Integer>> out = in.project(2,0);
```

**使用类型提示进行投影**

请注意，Java编译器无法推断`project`操作符的返回类型。如果您对操作符的结果调用另一个操作符，则可能会导致问题，`project`例如：

```java
DataSet<Tuple5<String,String,String,String,String>> ds = ....
DataSet<Tuple1<String>> ds2 = ds.project(0).distinct(0);
```

通过提示返回类型的`project`操作符可以克服此问题，如下所示：

```java
DataSet<Tuple1<String>> ds2 = ds.<Tuple1<String>>project(0).distinct(0);
```
{% endtab %}

{% tab title="Scala" %}
```text
Not supported.
```
{% endtab %}

{% tab title="Python" %}
```text
out = in.project(2,0);
```
{% endtab %}
{% endtabs %}

## 分组数据集转换

reduce操作可以对分组数据集进行操作。指定用于分组的密钥可以通过多种方式完成：

* 关键表达
* Key选择器函数
* 一个或多个字段位置键（仅限元组数据集）
* Case Class字段（仅限案例类）

请查看reduce示例以了解如何指定分组键。

## Reduce分组数据集

应用于分组数据集的Reduce转换使用用户定义的Reduce函数将每个组缩减为单个元素。对于每组输入元素，reduce函数依次将成对的元素组合到一个元素中，直到每个组只剩下一个元素为止。

注意，对于`ReduceFunction`，返回对象的键控字段应该与输入值匹配。这是因为reduce是隐式组合的，当传递给reduce操作符时，从组合操作符发出的对象再次按键分组。

### 对按键表达式分组的数据集进行Reduce

键表达式指定数据集中每个元素的一个或多个字段。每个键表达式要么是公共字段的名称，要么是`getter`方法的名称。点可以被用来向下钻取物体。键表达式`“*”`选择所有字段。下面的代码展示了如何使用关键表达式对POJO数据集进行分组，并使用reduce函数对其进行缩减。

{% tabs %}
{% tab title="Java" %}
```java
// some ordinary POJO
public class WC {
  public String word;
  public int count;
  // [...]
}

// ReduceFunction that sums Integer attributes of a POJO
public class WordCounter implements ReduceFunction<WC> {
  @Override
  public WC reduce(WC in1, WC in2) {
    return new WC(in1.word, in1.count + in2.count);
  }
}

// [...]
DataSet<WC> words = // [...]
DataSet<WC> wordCounts = words
                         // DataSet grouping on field "word"
                         .groupBy("word")
                         // apply ReduceFunction on grouped DataSet
                         .reduce(new WordCounter());
```
{% endtab %}

{% tab title="Scala" %}
```scala
// some ordinary POJO
class WC(val word: String, val count: Int) {
  def this() {
    this(null, -1)
  }
  // [...]
}

val words: DataSet[WC] = // [...]
val wordCounts = words.groupBy("word").reduce {
  (w1, w2) => new WC(w1.word, w1.count + w2.count)
}
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

### 对**KeySelector**函数分组的数据集进行Reduce

键选择器函数从数据集的每个元素中提取键值。提取的键值用于对数据集进行分组。下面的代码展示了如何使用键选择器函数对POJO数据集进行分组，并使用reduce函数对其进行缩减。

{% tabs %}
{% tab title="Java" %}
```java
// some ordinary POJO
public class WC {
  public String word;
  public int count;
  // [...]
}

// ReduceFunction that sums Integer attributes of a POJO
public class WordCounter implements ReduceFunction<WC> {
  @Override
  public WC reduce(WC in1, WC in2) {
    return new WC(in1.word, in1.count + in2.count);
  }
}

// [...]
DataSet<WC> words = // [...]
DataSet<WC> wordCounts = words
                         // DataSet grouping on field "word"
                         .groupBy(new SelectWord())
                         // apply ReduceFunction on grouped DataSet
                         .reduce(new WordCounter());

public class SelectWord implements KeySelector<WC, String> {
  @Override
  public String getKey(Word w) {
    return w.word;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
// some ordinary POJO
class WC(val word: String, val count: Int) {
  def this() {
    this(null, -1)
  }
  // [...]
}

val words: DataSet[WC] = // [...]
val wordCounts = words.groupBy { _.word } reduce {
  (w1, w2) => new WC(w1.word, w1.count + w2.count)
}
```
{% endtab %}

{% tab title="Python" %}
```python
class WordCounter(ReduceFunction):
    def reduce(self, in1, in2):
        return (in1[0], in1[1] + in2[1])

words = // [...]
wordCounts = words \
    .group_by(lambda x: x[0]) \
    .reduce(WordCounter()) 
```
{% endtab %}
{% endtabs %}

### 按字段位置键分组的数据集上的Reduce\(仅限元组数据集\)

字段位置键指定用作分组键的元组数据集的一个或多个字段。下面的代码展示了如何使用字段位置键并应用reduce函数

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<String, Integer, Double>> tuples = // [...]
DataSet<Tuple3<String, Integer, Double>> reducedTuples = tuples
                                         // group DataSet on first and second field of Tuple
                                         .groupBy(0, 1)
                                         // apply ReduceFunction on grouped DataSet
                                         .reduce(new MyTupleReducer());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val tuples = DataSet[(String, Int, Double)] = // [...]
// group on the first and second Tuple field
val reducedTuples = tuples.groupBy(0, 1).reduce { ... }
```
{% endtab %}

{% tab title="Python" %}
```python
reducedTuples = tuples.group_by(0, 1).reduce( ... )
```
{% endtab %}
{% endtabs %}

### 按Case类字段分组的数据集上的Reduce

使用Case Classes时，还可以使用字段名称指定分组键：

{% tabs %}
{% tab title="Java" %}
```java
Not supported.
```
{% endtab %}

{% tab title="Scala" %}
```scala
case class MyClass(val a: String, b: Int, c: Double)
val tuples = DataSet[MyClass] = // [...]
// group on the first and second field
val reducedTuples = tuples.groupBy("a", "b").reduce { ... }
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

## 分组数据集上的GroupReduce

应用于分组数据集的GroupReduce转换为每个组调用用户定义的group-reduce函数。这与Reduce的区别在于，用户定义的函数一次获得整个组。该函数使用可遍历组的所有元素来调用，可以返回任意数量的结果元素。

### **由字段位置键分组的DataSet上的GroupReduce（仅限元组数据集）**

下面的代码展示了如何从按整数分组的数据集中删除重复字符串。

{% tabs %}
{% tab title="Java" %}
```java
public class DistinctReduce
         implements GroupReduceFunction<Tuple2<Integer, String>, Tuple2<Integer, String>> {

  @Override
  public void reduce(Iterable<Tuple2<Integer, String>> in, Collector<Tuple2<Integer, String>> out) {

    Set<String> uniqStrings = new HashSet<String>();
    Integer key = null;

    // add all strings of the group to the set
    for (Tuple2<Integer, String> t : in) {
      key = t.f0;
      uniqStrings.add(t.f1);
    }

    // emit all unique strings.
    for (String s : uniqStrings) {
      out.collect(new Tuple2<Integer, String>(key, s));
    }
  }
}

// [...]
DataSet<Tuple2<Integer, String>> input = // [...]
DataSet<Tuple2<Integer, String>> output = input
                           .groupBy(0)            // group DataSet by the first tuple field
                           .reduceGroup(new DistinctReduce());  // apply GroupReduceFunction
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String)] = // [...]
val output = input.groupBy(0).reduceGroup {
      (in, out: Collector[(Int, String)]) =>
        in.toSet foreach (out.collect)
    }
```
{% endtab %}

{% tab title="Python" %}
```python
class DistinctReduce(GroupReduceFunction):
   def reduce(self, iterator, collector):
     dic = dict()
     for value in iterator:
       dic[value[1]] = 1
     for key in dic.keys():
       collector.collect(key)

 output = data.group_by(0).reduce_group(DistinctReduce())
```
{% endtab %}
{% endtabs %}

### 对按键表达式、键选择器函数或Case类字段分组的数据集进行GroupReduce

类似于_Reduce_转换中的[键表达式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-dataset-grouped-by-key-expression)， [键选择器函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-dataset-grouped-by-keyselector-function)和[案例类字段的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-dataset-grouped-by-case-class-fields)工作。

### **已排序组的GroupReduce**

group-reduce函数使用迭代器访问组的元素。可选的是，Iterable可以按指定的顺序分发组的元素。在许多情况下，这有助于降低用户定义的group-reduce函数的复杂性，并提高其效率。

下面的代码展示了另一个示例，该示例如何删除按整数分组并按字符串排序的数据集中的重复字符串。

{% tabs %}
{% tab title="Java" %}
```java
// GroupReduceFunction that removes consecutive identical elements
public class DistinctReduce
         implements GroupReduceFunction<Tuple2<Integer, String>, Tuple2<Integer, String>> {

  @Override
  public void reduce(Iterable<Tuple2<Integer, String>> in, Collector<Tuple2<Integer, String>> out) {
    Integer key = null;
    String comp = null;

    for (Tuple2<Integer, String> t : in) {
      key = t.f0;
      String next = t.f1;

      // check if strings are different
      if (com == null || !next.equals(comp)) {
        out.collect(new Tuple2<Integer, String>(key, next));
        comp = next;
      }
    }
  }
}

// [...]
DataSet<Tuple2<Integer, String>> input = // [...]
DataSet<Double> output = input
                         .groupBy(0)                         // group DataSet by first field
                         .sortGroup(1, Order.ASCENDING)      // sort groups on second tuple field
                         .reduceGroup(new DistinctReduce());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String)] = // [...]
val output = input.groupBy(0).sortGroup(1, Order.ASCENDING).reduceGroup {
      (in, out: Collector[(Int, String)]) =>
        var prev: (Int, String) = null
        for (t <- in) {
          if (prev == null || prev != t)
            out.collect(t)
            prev = t
        }
    }
```
{% endtab %}

{% tab title="Python" %}
```python
 class DistinctReduce(GroupReduceFunction):
   def reduce(self, iterator, collector):
     dic = dict()
     for value in iterator:
       dic[value[1]] = 1
     for key in dic.keys():
       collector.collect(key)

 output = data.group_by(0).sort_group(1, Order.ASCENDING).reduce_group(DistinctReduce())
```
{% endtab %}
{% endtabs %}

注意:如果在reduce操作之前使用基于排序的操作符执行策略建立分组，那么GroupSort通常是免费的。

### **可组合的GroupReduceFunction**

与reduce函数相比，group-reduce函数不是隐式组合的。为了使group-reduce函数可组合，它必须实现GroupCombineFunction接口。

重要提示:`GroupCombineFunction`接口的通用输入和输出类型必须等于`GroupReduceFunction`的通用输入类型，如下例所示:

{% tabs %}
{% tab title="Java" %}
```java
// Combinable GroupReduceFunction that computes a sum.
public class MyCombinableGroupReducer implements
  GroupReduceFunction<Tuple2<String, Integer>, String>,
  GroupCombineFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>
{
  @Override
  public void reduce(Iterable<Tuple2<String, Integer>> in,
                     Collector<String> out) {

    String key = null;
    int sum = 0;

    for (Tuple2<String, Integer> curr : in) {
      key = curr.f0;
      sum += curr.f1;
    }
    // concat key and sum and emit
    out.collect(key + "-" + sum);
  }

  @Override
  public void combine(Iterable<Tuple2<String, Integer>> in,
                      Collector<Tuple2<String, Integer>> out) {
    String key = null;
    int sum = 0;

    for (Tuple2<String, Integer> curr : in) {
      key = curr.f0;
      sum += curr.f1;
    }
    // emit tuple with key and sum
    out.collect(new Tuple2<>(key, sum));
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Combinable GroupReduceFunction that computes two sums.
class MyCombinableGroupReducer
  extends GroupReduceFunction[(String, Int), String]
  with GroupCombineFunction[(String, Int), (String, Int)]
{
  override def reduce(
    in: java.lang.Iterable[(String, Int)],
    out: Collector[String]): Unit =
  {
    val r: (String, Int) =
      in.iterator.asScala.reduce( (a,b) => (a._1, a._2 + b._2) )
    // concat key and sum and emit
    out.collect (r._1 + "-" + r._2)
  }

  override def combine(
    in: java.lang.Iterable[(String, Int)],
    out: Collector[(String, Int)]): Unit =
  {
    val r: (String, Int) =
      in.iterator.asScala.reduce( (a,b) => (a._1, a._2 + b._2) )
    // emit tuple with key and sum
    out.collect(r)
  }
}
```
{% endtab %}

{% tab title="Python" %}
```python
 class GroupReduce(GroupReduceFunction):
   def reduce(self, iterator, collector):
     key, int_sum = iterator.next()
     for value in iterator:
       int_sum += value[1]
     collector.collect(key + "-" + int_sum))

   def combine(self, iterator, collector):
     key, int_sum = iterator.next()
     for value in iterator:
       int_sum += value[1]
     collector.collect((key, int_sum))

data.reduce_group(GroupReduce(), combinable=True)
```
{% endtab %}
{% endtabs %}

## 分组**数据**集上的GroupCombine

GroupCombine转换是可组合GroupReduceFunction中的组合步骤的通用形式。从某种意义上说，它允许将输入类型组合`I`到任意输出类型`O`。相反，GroupReduce中的组合步骤仅允许从输入类型`I`到输出类型的组合`I`。这是因为GroupReduceFunction中的reduce步骤需要输入类型`I`。

在某些应用程序中，在执行其他转换\(例如减少数据大小\)之前，最好将数据集合并为中间格式。这可以通过一个很少成本的CombineGroup转换来实现。

**注意：**分组数据集上的GroupCombine在内存中使用贪婪策略执行，该策略可能不会一次处理所有数据，而是以多个步骤处理。它也可以在各个分区上执行，而无需像GroupReduce转换那样进行数据交换。这可能会导致部分结果。

以下示例演示了如何将CombineGroup转换用于备用WordCount实现。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<String> input = [..] // The words received as input

DataSet<Tuple2<String, Integer>> combinedWords = input
  .groupBy(0) // group identical words
  .combineGroup(new GroupCombineFunction<String, Tuple2<String, Integer>() {

    public void combine(Iterable<String> words, Collector<Tuple2<String, Integer>>) { // combine
        String key = null;
        int count = 0;

        for (String word : words) {
            key = word;
            count++;
        }
        // emit tuple with word and count
        out.collect(new Tuple2(key, count));
    }
});

DataSet<Tuple2<String, Integer>> output = combinedWords
  .groupBy(0)                              // group by words again
  .reduceGroup(new GroupReduceFunction() { // group reduce with full data exchange

    public void reduce(Iterable<Tuple2<String, Integer>>, Collector<Tuple2<String, Integer>>) {
        String key = null;
        int count = 0;

        for (Tuple2<String, Integer> word : words) {
            key = word;
            count++;
        }
        // emit tuple with word and count
        out.collect(new Tuple2(key, count));
    }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[String] = [..] // The words received as input

val combinedWords: DataSet[(String, Int)] = input
  .groupBy(0)
  .combineGroup {
    (words, out: Collector[(String, Int)]) =>
        var key: String = null
        var count = 0

        for (word <- words) {
            key = word
            count += 1
        }
        out.collect((key, count))
}

val output: DataSet[(String, Int)] = combinedWords
  .groupBy(0)
  .reduceGroup {
    (words, out: Collector[(String, Int)]) =>
        var key: String = null
        var sum = 0

        for ((word, sum) <- words) {
            key = word
            sum += count
        }
        out.collect((key, sum))
}
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

上面的备选WordCount实现演示了GroupCombine如何在执行GroupReduce转换之前组合单词。上面的例子只是一个概念证明。注意，合并步骤如何更改数据集的类型，通常在执行GroupReduce之前需要进行额外的映射转换。

## 对分组的元组数据集进行聚合

有一些常用的聚合操作经常使用。Aggregate转换提供以下内置聚合函数：

* Sum,
* Min, 
* Max.

聚合转换只能应用于元组数据集，并且仅支持字段位置键进行分组。

以下代码展示如何对按字段位置键分组的DataSet应用聚合转换：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input
                                   .groupBy(1)        // group DataSet on second field
                                   .aggregate(SUM, 0) // compute sum of the first field
                                   .and(MIN, 2);      // compute minimum of the third field
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Double)] = // [...]
val output = input.groupBy(1).aggregate(SUM, 0).and(MIN, 2)
```
{% endtab %}

{% tab title="Python" %}
```python
from flink.functions.Aggregation import Sum, Min

input = # [...]
output = input.group_by(1).aggregate(Sum, 0).and_agg(Min, 2)
```
{% endtab %}
{% endtabs %}

要在DataSet上应用多个聚合，必须在第一个聚合之后使用`.and()`函数，这意味着`.aggregate(SUM, 0).and(MIN, 2)`生成字段0的总和和原始DataSet的字段2的最小值。与此相反，`.aggregate(SUM, 0).aggregate(MIN, 2)`将在聚合上应用聚合。在给定的示例中，在计算由字段1分组的字段0的总和之后，它将产生字段2的最小值。  


**注意：**将来会扩展聚合函数集。

## 分组元组数据集上的MinBy / MaxBy

MinBy \(MaxBy\)转换为每组元组选择一个元组。选择的元组是一个或多个指定字段的值为最小\(最大值\)的元组。用于比较的字段必须是有效的键字段，即。,具有可比性。如果多个元组具有最小\(最大\)字段值，则返回这些元组的任意一个元组。

下面的代码展示了如何从 `DataSet<Tuple3<Integer, String, Double>>`中为每组具有相同字符串值的元组选择具有`Integer`和`Double`字段最小值的元组:

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input
                                   .groupBy(1)   // group DataSet on second field
                                   .minBy(0, 2); // select tuple with minimum values for first and third field.
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Double)] = // [...]
val output: DataSet[(Int, String, Double)] = input
                                   .groupBy(1)  // group DataSet on second field
                                   .minBy(0, 2) // select tuple with minimum values for first and third field.
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

## 完整数据集上的Reduce

Reduce转换将用户定义的reduce函数应用于DataSet的所有元素。reduce函数随后将元素对组合成一个元素，直到只剩下一个元素。

以下代码暂时如何对Integer DataSet的所有元素求和：

{% tabs %}
{% tab title="Java" %}
```java
// ReduceFunction that sums Integers
public class IntSummer implements ReduceFunction<Integer> {
  @Override
  public Integer reduce(Integer num1, Integer num2) {
    return num1 + num2;
  }
}

// [...]
DataSet<Integer> intNumbers = // [...]
DataSet<Integer> sum = intNumbers.reduce(new IntSummer());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val intNumbers = env.fromElements(1,2,3)
val sum = intNumbers.reduce (_ + _)
```
{% endtab %}

{% tab title="Python" %}
```python
intNumbers = env.from_elements(1,2,3)
 sum = intNumbers.reduce(lambda x,y: x + y)
```
{% endtab %}
{% endtabs %}

使用Reduce转换减少完整的DataSet意味着最终的Reduce操作不能并行完成。但是，reduce函数可以自动组合，因此Reduce转换不会限制大多数用例的可伸缩性。

## 完整数据集上的GroupReduce

GroupReduce转换在DataSet的所有元素上应用用户定义的group-reduce函数。group-reduce可以迭代DataSet的所有元素并返回任意数量的结果元素。

以下示例展示如何在完整DataSet上应用GroupReduce转换：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Integer> input = // [...]
// apply a (preferably combinable) GroupReduceFunction to a DataSet
DataSet<Double> output = input.reduceGroup(new MyGroupReducer());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[Int] = // [...]
val output = input.reduceGroup(new MyGroupReducer())
```
{% endtab %}

{% tab title="Python" %}
```text
 output = data.reduce_group(MyGroupReducer())
```
{% endtab %}
{% endtabs %}

**注意：**如果group-reduce函数不可组合，则无法并行完成对完整DataSet的GroupReduce转换。因此，这可能是计算密集型操作。请参阅上面的“可组合GroupReduceFunctions”一节，了解如何实现可组合的group-reduce函数。

## 完整数据集上的GroupCombine

完整DataSet上的GroupCombine与分组DataSet上的GroupCombine类似。数据在所有节点上分区，然后以贪婪的方式组合（即，只有一次合并到存储器中的数据）。

## 完整元组数据集上的聚合

有一些常用的聚合操作经常使用。Aggregate转换提供以下内置聚合函数：

* Sum,
* Min,
* Max.

聚合转换只能应用于元组数据集。

以下代码展示如何在完整DataSet上应用聚合转换：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<Integer, Double>> input = // [...]
DataSet<Tuple2<Integer, Double>> output = input
                                     .aggregate(SUM, 0)    // compute sum of the first field
                                     .and(MIN, 1);    // compute minimum of the second field
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Double)] = // [...]
val output = input.aggregate(SUM, 0).and(MIN, 2)
```
{% endtab %}

{% tab title="Python" %}
```python
from flink.functions.Aggregation import Sum, Min

input = # [...]
output = input.aggregate(Sum, 0).and_agg(Min, 2)
```
{% endtab %}
{% endtabs %}

**注意：**扩展支持的聚合功能集在我们的路线图中。

## 完整元组数据集上的MinBy / MaxBy

MinBy \(MaxBy\)转换从元组数据集中选择单个元组。选择的元组是一个或多个指定字段的值为最小\(最大值\)的元组。用于比较的字段必须是有效的键字段，即,具有可比性。如果多个元组具有最小\(最大\)字段值，则返回这些元组的任意一个元组。

下面的代码展示了如何从`DataSet<Tuple3<Integer, String, Double>>`中选择具有整数和双字段最大值的tuple:

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input
                                   .maxBy(0, 2); // select tuple with maximum values for first and third field.
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Double)] = // [...]
val output: DataSet[(Int, String, Double)] = input                          
                                   .maxBy(0, 2) // select tuple with maximum values for first and third field.
```
{% endtab %}

{% tab title="Python" %}
```python
Not supported.
```
{% endtab %}
{% endtabs %}

## Distinct

Distinct转换计算源DataSet的不同元素的DataSet。以下代码从DataSet中删除所有重复的元素：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<Integer, Double>> input = // [...]
DataSet<Tuple2<Integer, Double>> output = input.distinct();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Double)] = // [...]
val output = input.distinct()
```
{% endtab %}

{% tab title="Python" %}
```python
Not supported.
```
{% endtab %}
{% endtabs %}

也可以改变数据集中元素的区别是如何决定的，使用:

* 一个或多个字段位置键（仅限元组数据集），
* 键选择器函数
* 键表达式

### 与字段位置键不同

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<Integer, Double, String>> input = // [...]
DataSet<Tuple2<Integer, Double, String>> output = input.distinct(0,2);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, Double, String)] = // [...]
val output = input.distinct(0,2)
```
{% endtab %}

{% tab title="Python" %}
```
Not supported.
```
{% endtab %}
{% endtabs %}

### **与KeySelector功能不同**

{% tabs %}
{% tab title="Java" %}
```java
private static class AbsSelector implements KeySelector<Integer, Integer> {
private static final long serialVersionUID = 1L;
	@Override
	public Integer getKey(Integer t) {
    	return Math.abs(t);
	}
}
DataSet<Integer> input = // [...]
DataSet<Integer> output = input.distinct(new AbsSelector());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[Int] = // [...]
val output = input.distinct {x => Math.abs(x)}
```
{% endtab %}

{% tab title="Python" %}
```python
Not supported.
```
{% endtab %}
{% endtabs %}

### **与键表达式不同**

{% tabs %}
{% tab title="Java" %}
```java
// some ordinary POJO
public class CustomType {
  public String aName;
  public int aNumber;
  // [...]
}

DataSet<CustomType> input = // [...]
DataSet<CustomType> output = input.distinct("aName", "aNumber");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// some ordinary POJO
case class CustomType(aName : String, aNumber : Int) { }

val input: DataSet[CustomType] = // [...]
val output = input.distinct("aName", "aNumber")
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

也可以通过通配符指示使用所有字段：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<CustomType> input = // [...]
DataSet<CustomType> output = input.distinct("*");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// some ordinary POJO
val input: DataSet[CustomType] = // [...]
val output = input.distinct("_")
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

## Join

Join转换将两个DataSet连接到一个DataSet中。两个DataSet的元素连接在一个或多个可以使用的键上

* Key表达式
* 键选择器函数
* 一个或多个字段位置键（仅限元组数据集）。
* Case类字段

有几种不同的方法来执行Join变换，如下所示。

### **默认关联（关联Tuple2）**

默认的**关联**转换会生成一个包含两个字段的新Tuple DataSet。每个元组保存第一个元组字段中第一个输入DataSet的关联元素和第二个字段中第二个输入DataSet的匹配元素。

以下代码显示使用字段位置键的默认关联转换：

{% tabs %}
{% tab title="Java" %}
```java
public static class User { public String name; public int zip; }
public static class Store { public Manager mgr; public int zip; }
DataSet<User> input1 = // [...]
DataSet<Store> input2 = // [...]
// result dataset is typed as Tuple2
DataSet<Tuple2<User, Store>>
            result = input1.join(input2)
                           .where("zip")       // key of the first input (users)
                           .equalTo("zip");    // key of the second input (stores)
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input1: DataSet[(Int, String)] = // [...]
val input2: DataSet[(Double, Int)] = // [...]
val result = input1.join(input2).where(0).equalTo(1)
```
{% endtab %}

{% tab title="Python" %}
```python
 result = input1.join(input2).where(0).equal_to(1)
```
{% endtab %}
{% endtabs %}

### **关联Join函数**

**关联**转换还可以调用用户定义的连接函数来处理连接元组。关联函数接收第一个输入DataSet的一个元素和第二个输入DataSet的一个元素，并返回一个元素。

以下代码使用键选择器函数执行DataSet与自定义java对象和Tuple DataSet的关联，并显示如何使用用户定义的关联函数：

{% tabs %}
{% tab title="Java" %}
```java
// some POJO
public class Rating {
  public String name;
  public String category;
  public int points;
}

// Join function that joins a custom POJO with a Tuple
public class PointWeighter
         implements JoinFunction<Rating, Tuple2<String, Double>, Tuple2<String, Double>> {

  @Override
  public Tuple2<String, Double> join(Rating rating, Tuple2<String, Double> weight) {
    // multiply the points and rating and construct a new output tuple
    return new Tuple2<String, Double>(rating.name, rating.points * weight.f1);
  }
}

DataSet<Rating> ratings = // [...]
DataSet<Tuple2<String, Double>> weights = // [...]
DataSet<Tuple2<String, Double>>
            weightedRatings =
            ratings.join(weights)

                   // key of the first input
                   .where("category")

                   // key of the second input
                   .equalTo("f0")

                   // applying the JoinFunction on joining pairs
                   .with(new PointWeighter());
```
{% endtab %}

{% tab title="Scala" %}
```scala
case class Rating(name: String, category: String, points: Int)

val ratings: DataSet[Ratings] = // [...]
val weights: DataSet[(String, Double)] = // [...]

val weightedRatings = ratings.join(weights).where("category").equalTo(0) {
  (rating, weight) => (rating.name, rating.points * weight._2)
}
```
{% endtab %}

{% tab title="Python" %}
```python
 class PointWeighter(JoinFunction):
   def join(self, rating, weight):
     return (rating[0], rating[1] * weight[1])
       if value1[3]:

 weightedRatings =
   ratings.join(weights).where(0).equal_to(0). \
   with(new PointWeighter());
```
{% endtab %}
{% endtabs %}

### 关联Flat-Join函数

与Map和FlatMap类似，FlatJoin的行为与Join相同，但它可以返回\(collect\)、0、1或多个元素，而不是返回一个元素。

{% tabs %}
{% tab title="Java" %}
```java
public class PointWeighter
         implements FlatJoinFunction<Rating, Tuple2<String, Double>, Tuple2<String, Double>> {
  @Override
  public void join(Rating rating, Tuple2<String, Double> weight,
	  Collector<Tuple2<String, Double>> out) {
	if (weight.f1 > 0.1) {
		out.collect(new Tuple2<String, Double>(rating.name, rating.points * weight.f1));
	}
  }
}

DataSet<Tuple2<String, Double>>
            weightedRatings =
            ratings.join(weights) // [...]
```
{% endtab %}

{% tab title="Scala" %}
```scala
case class Rating(name: String, category: String, points: Int)

val ratings: DataSet[Ratings] = // [...]
val weights: DataSet[(String, Double)] = // [...]

val weightedRatings = ratings.join(weights).where("category").equalTo(0) {
  (rating, weight, out: Collector[(String, Double)]) =>
    if (weight._2 > 0.1) out.collect(rating.name, rating.points * weight._2)
}
```
{% endtab %}

{% tab title="Python" %}
```python
Not supported.
```
{% endtab %}
{% endtabs %}

### **关联投影（仅限Java / Python）**

**关联转换**可以使用如下所示的投影构造结果元组：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple3<Integer, Byte, String>> input1 = // [...]
DataSet<Tuple2<Integer, Double>> input2 = // [...]
DataSet<Tuple4<Integer, String, Double, Byte>>
            result =
            input1.join(input2)
                  // key definition on first DataSet using a field position key
                  .where(0)
                  // key definition of second DataSet using a field position key
                  .equalTo(0)
                  // select and reorder fields of matching tuples
                  .projectFirst(0,2).projectSecond(1).projectFirst(1);
```

`projectFirst(int...)`并`projectSecond(int...)`选择应组合成输出元组的第一个和第二个关联输入的字段。索引的顺序定义了输出元组中字段的顺序。关联投影也适用于非元组数据集。在这种情况下，`projectFirst()`或者`projectSecond()`必须在没有参数的情况下调用，以将关联元素添加到输出元组。
{% endtab %}

{% tab title="Scala" %}
```scala
Not supported.
```
{% endtab %}

{% tab title="Python" %}
```python
result = input1.join(input2).where(0).equal_to(0) \
  .project_first(0,2).project_second(1).project_first(1);
```

`projectFirst(int...)`并`projectSecond(int...)`选择应组合成输出元组的第一个和第二个关联输入的字段。索引的顺序定义了输出元组中字段的顺序。关联投影也适用于非元组数据集。在这种情况下，`projectFirst()`或者`projectSecond()`必须在没有参数的情况下调用，以将关联元素添加到输出元组。
{% endtab %}
{% endtabs %}

### **关联DataSet Size提示**

为了引导优化器选择正确的执行策略，可以关联的DataSet的大小提示，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<Integer, String>> input1 = // [...]
DataSet<Tuple2<Integer, String>> input2 = // [...]

DataSet<Tuple2<Tuple2<Integer, String>, Tuple2<Integer, String>>>
            result1 =
            // hint that the second DataSet is very small
            input1.joinWithTiny(input2)
                  .where(0)
                  .equalTo(0);

DataSet<Tuple2<Tuple2<Integer, String>, Tuple2<Integer, String>>>
            result2 =
            // hint that the second DataSet is very large
            input1.joinWithHuge(input2)
                  .where(0)
                  .equalTo(0);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input1: DataSet[(Int, String)] = // [...]
val input2: DataSet[(Int, String)] = // [...]

// hint that the second DataSet is very small
val result1 = input1.joinWithTiny(input2).where(0).equalTo(0)

// hint that the second DataSet is very large
val result1 = input1.joinWithHuge(input2).where(0).equalTo(0)
```
{% endtab %}

{% tab title="Python" %}
```python
 #hint that the second DataSet is very small
 result1 = input1.join_with_tiny(input2).where(0).equal_to(0)

 #hint that the second DataSet is very large
 result1 = input1.join_with_huge(input2).where(0).equal_to(0)
```
{% endtab %}
{% endtabs %}

### **关联算法提示**

Flink运行时可以以各种方式执行连接。在不同的情况下，每种可能的方法都优于其他方法。系统试图自动选择一种合理的方式，但允许您手动选择一种策略，以防您希望强制执行特定的连接方式。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<SomeType> input1 = // [...]
DataSet<AnotherType> input2 = // [...]

DataSet<Tuple2<SomeType, AnotherType> result =
      input1.join(input2, JoinHint.BROADCAST_HASH_FIRST)
            .where("id").equalTo("key");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input1: DataSet[SomeType] = // [...]
val input2: DataSet[AnotherType] = // [...]

// hint that the second DataSet is very small
val result1 = input1.join(input2, JoinHint.BROADCAST_HASH_FIRST).where("id").equalTo("key")
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

以下提示可用：

* `OPTIMIZER_CHOOSES`：相当于不提供任何提示，将选择留给系统。
* `BROADCAST_HASH_FIRST`：广播第一个输入并从中构建哈希表，由第二个输入探测。如果第一个输入非常小，这是一个很好的策略。
* `BROADCAST_HASH_SECOND`：广播第二个输入并从中构建哈希表，由第一个输入探测。如果第二个输入非常小，这是一个好策略。
* `REPARTITION_HASH_FIRST`：系统分区（shuffle）每个输入（除非输入已经分区）并从第一个输入构建哈希表。如果第一个输入小于第二个输入，则此策略很好，但两个输入仍然很大。 _注意：_如果不能进行大小估算，并且不能重新使用预先存在的分区和排序顺序，则这是系统使用的默认回退策略。
* `REPARTITION_HASH_SECOND`：系统分区（shuffle）每个输入（除非输入已经分区）并从第二个输入构建哈希表。如果第二个输入小于第一个输入，则此策略很好，但两个输入仍然很大。
* `REPARTITION_SORT_MERGE`：系统分区（shuffle）每个输入（除非输入已经分区）并对每个输入进行排序（除非它已经排序）。输入通过已排序输入的流合并来连接。如果已经对一个或两个输入进行了排序，则此策略很好。

## OuterJoin

OuterJoin转换对两个数据集执行左关联、右关联或完全外关联。外部关联类似于常规\(内部\)关联，并创建键上相等的所有元素对。此外，如果在另一侧没有找到匹配的键，则“外部”端\(在full的情况下为左、右或两者\)的记录将被保留。将匹配的元素对\(或一个元素和另一个输入的空值\)提供给`JoinFunction`，以将元素对转换为单个元素，或使用`FlatJoinFunction`将元素对转换为任意多个\(包括none\)元素。

两个数据集的元素关联在一个或多个键上，可以使用这些键指定

* 键的表达式
* 键选择器函数
* 一个或多个字段位置键（仅限元组数据集）。
* Case类字段

**OuterJoins仅支持Java和Scala DataSet API。**

### **具有关联函数的**OuterJoin

OuterJoin转换调用用户定义的关联函数来处理连接元组。关联函数接收第一个输入数据集的一个元素和第二个输入数据集的一个元素，并准确地返回一个元素。根据外部关联的类型\(left, right, full\)，关联函数的两个输入元素之一可以是null。

以下代码使用键选择器函数执行DataSet与自定义java对象和Tuple DataSet的左外关联，并**展示**如何使用用户定义的连接函数：

{% tabs %}
{% tab title="Java" %}
```java
// some POJO
public class Rating {
  public String name;
  public String category;
  public int points;
}

// Join function that joins a custom POJO with a Tuple
public class PointAssigner
         implements JoinFunction<Tuple2<String, String>, Rating, Tuple2<String, Integer>> {

  @Override
  public Tuple2<String, Integer> join(Tuple2<String, String> movie, Rating rating) {
    // Assigns the rating points to the movie.
    // NOTE: rating might be null
    return new Tuple2<String, Double>(movie.f0, rating == null ? -1 : rating.points;
  }
}

DataSet<Tuple2<String, String>> movies = // [...]
DataSet<Rating> ratings = // [...]
DataSet<Tuple2<String, Integer>>
            moviesWithPoints =
            movies.leftOuterJoin(ratings)

                   // key of the first input
                   .where("f0")

                   // key of the second input
                   .equalTo("name")

                   // applying the JoinFunction on joining pairs
                   .with(new PointAssigner());
```
{% endtab %}

{% tab title="Scala" %}
```scala
case class Rating(name: String, category: String, points: Int)

val movies: DataSet[(String, String)] = // [...]
val ratings: DataSet[Ratings] = // [...]

val moviesWithPoints = movies.leftOuterJoin(ratings).where(0).equalTo("name") {
  (movie, rating) => (movie._1, if (rating == null) -1 else rating.points)
}
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

### **具有Flat-Join函数的**OuterJoin

类似于Map和FlatMap，具有**Flat-Join函数**的OuterJoin与具有关联函数的OuterJoin的行为方式相同，但它不返回一个元素，而是返回（收集），零个，一个或多个元素。

{% tabs %}
{% tab title="Java" %}
```java
public class PointAssigner
         implements FlatJoinFunction<Tuple2<String, String>, Rating, Tuple2<String, Integer>> {
  @Override
  public void join(Tuple2<String, String> movie, Rating rating
    Collector<Tuple2<String, Integer>> out) {
  if (rating == null ) {
    out.collect(new Tuple2<String, Integer>(movie.f0, -1));
  } else if (rating.points < 10) {
    out.collect(new Tuple2<String, Integer>(movie.f0, rating.points));
  } else {
    // do not emit
  }
}

DataSet<Tuple2<String, Integer>>
            moviesWithPoints =
            movies.leftOuterJoin(ratings) // [...]
```
{% endtab %}

{% tab title="Scala" %}
```text
Not supported.
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

### **加入算法提示**

Flink运行时可以以各种方式执行外部连接。在不同的情况下，每种可能的方法都优于其他方法。系统试图自动选择一种合理的方式，但允许您手动选择策略，以防您强制执行外部联接的特定方式。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<SomeType> input1 = // [...]
DataSet<AnotherType> input2 = // [...]

DataSet<Tuple2<SomeType, AnotherType> result1 =
      input1.leftOuterJoin(input2, JoinHint.REPARTITION_SORT_MERGE)
            .where("id").equalTo("key");

DataSet<Tuple2<SomeType, AnotherType> result2 =
      input1.rightOuterJoin(input2, JoinHint.BROADCAST_HASH_FIRST)
            .where("id").equalTo("key");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input1: DataSet[SomeType] = // [...]
val input2: DataSet[AnotherType] = // [...]

// hint that the second DataSet is very small
val result1 = input1.leftOuterJoin(input2, JoinHint.REPARTITION_SORT_MERGE).where("id").equalTo("key")

val result2 = input1.rightOuterJoin(input2, JoinHint.BROADCAST_HASH_FIRST).where("id").equalTo("key")
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

以下提示可用。

* `OPTIMIZER_CHOOSES`：相当于不提供任何提示，将选择留给系统。
* `BROADCAST_HASH_FIRST`：广播第一个输入并从中构建哈希表，由第二个输入探测。如果第一个输入非常小，这是一个很好的策略。
* `BROADCAST_HASH_SECOND`：广播第二个输入并从中构建哈希表，由第一个输入探测。如果第二个输入非常小，这是一个好策略。
* `REPARTITION_HASH_FIRST`：系统分区（shuffle）每个输入（除非输入已经分区）并从第一个输入构建哈希表。如果第一个输入小于第二个输入，则此策略很好，但两个输入仍然很大。
* `REPARTITION_HASH_SECOND`：系统分区（shuffle）每个输入（除非输入已经分区）并从第二个输入构建哈希表。如果第二个输入小于第一个输入，则此策略很好，但两个输入仍然很大。
* `REPARTITION_SORT_MERGE`：系统分区（shuffle）每个输入（除非输入已经分区）并对每个输入进行排序（除非它已经排序）。输入通过已排序输入的流合并来连接。如果已经对一个或两个输入进行了排序，则此策略很好。

**注意：**并非所有外部联接类型都支持所有执行策略。

* `LeftOuterJoin` 支持：
  * `OPTIMIZER_CHOOSES`
  * `BROADCAST_HASH_SECOND`
  * `REPARTITION_HASH_SECOND`
  * `REPARTITION_SORT_MERGE`
* `RightOuterJoin` 支持：
  * `OPTIMIZER_CHOOSES`
  * `BROADCAST_HASH_FIRST`
  * `REPARTITION_HASH_FIRST`
  * `REPARTITION_SORT_MERGE`
* `FullOuterJoin` 支持：
  * `OPTIMIZER_CHOOSES`
  * `REPARTITION_SORT_MERGE`

## Cross

交叉转换将两个数据集组合成一个数据集。它构建两个输入数据集元素的所有成对组合，即它建立了一个笛卡尔积。交叉转换要么在每对元素上调用用户定义的交叉函数，要么输出Tuple2。两种模式如下所示。

{% hint style="danger" %}
**注意：**交叉可能是一种计算密集型操作，甚至可以挑战大型计算集群!
{% endhint %}

**与用户定义的函数交叉**

交叉转换可以调用用户定义的交叉函数。交叉函数接收第一个输入的一个元素和第二个输入的一个元素，并恰好返回一个结果元素。

下面的代码展示了如何使用交叉函数对两个数据集应用交叉转换:

{% tabs %}
{% tab title="Java" %}
```java
public class Coord {
  public int id;
  public int x;
  public int y;
}

// CrossFunction computes the Euclidean distance between two Coord objects.
public class EuclideanDistComputer
         implements CrossFunction<Coord, Coord, Tuple3<Integer, Integer, Double>> {

  @Override
  public Tuple3<Integer, Integer, Double> cross(Coord c1, Coord c2) {
    // compute Euclidean distance of coordinates
    double dist = sqrt(pow(c1.x - c2.x, 2) + pow(c1.y - c2.y, 2));
    return new Tuple3<Integer, Integer, Double>(c1.id, c2.id, dist);
  }
}

DataSet<Coord> coords1 = // [...]
DataSet<Coord> coords2 = // [...]
DataSet<Tuple3<Integer, Integer, Double>>
            distances =
            coords1.cross(coords2)
                   // apply CrossFunction
                   .with(new EuclideanDistComputer());
```

**与投影交叉**

交叉转换还可以使用如下所示的投影构造结果元组:

```java
DataSet<Tuple3<Integer, Byte, String>> input1 = // [...]
DataSet<Tuple2<Integer, Double>> input2 = // [...]
DataSet<Tuple4<Integer, Byte, Integer, Double>
            result =
            input1.cross(input2)
                        // select and reorder fields of matching tuples
                  .projectSecond(0).projectFirst(1,0).projectSecond(1);
```

交叉投影中的字段选择与关联结果的投影方式相同。
{% endtab %}

{% tab title="Scala" %}
```scala
case class Coord(id: Int, x: Int, y: Int)

val coords1: DataSet[Coord] = // [...]
val coords2: DataSet[Coord] = // [...]

val distances = coords1.cross(coords2) {
  (c1, c2) =>
    val dist = sqrt(pow(c1.x - c2.x, 2) + pow(c1.y - c2.y, 2))
    (c1.id, c2.id, dist)
}
```
{% endtab %}

{% tab title="Python" %}
```python
 class Euclid(CrossFunction):
   def cross(self, c1, c2):
     return (c1[0], c2[0], sqrt(pow(c1[1] - c2.[1], 2) + pow(c1[2] - c2[2], 2)))

 distances = coords1.cross(coords2).using(Euclid())
```

**与投影交叉**

交叉变换还可以使用如下所示的投影构造结果元组：

```python
result = input1.cross(input2).projectFirst(1,0).projectSecond(0,1);
```

交叉投影中的字段选择与关联结果的投影方式相同。
{% endtab %}
{% endtabs %}

### 交叉数据集大小提示

为了引导优化器选择正确的执行策略，可以提示要交叉的DataSet的大小，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<Integer, String>> input1 = // [...]
DataSet<Tuple2<Integer, String>> input2 = // [...]

DataSet<Tuple4<Integer, String, Integer, String>>
            udfResult =
                  // hint that the second DataSet is very small
            input1.crossWithTiny(input2)
                  // apply any Cross function (or projection)
                  .with(new MyCrosser());

DataSet<Tuple3<Integer, Integer, String>>
            projectResult =
                  // hint that the second DataSet is very large
            input1.crossWithHuge(input2)
                  // apply a projection (or any Cross function)
                  .projectFirst(0,1).projectSecond(1);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input1: DataSet[(Int, String)] = // [...]
val input2: DataSet[(Int, String)] = // [...]

// hint that the second DataSet is very small
val result1 = input1.crossWithTiny(input2)

// hint that the second DataSet is very large
val result1 = input1.crossWithHuge(input2)
```
{% endtab %}

{% tab title="Python" %}
```python
#hint that the second DataSet is very small
 result1 = input1.cross_with_tiny(input2)

 #hint that the second DataSet is very large
 result1 = input1.cross_with_huge(input2)
```
{% endtab %}
{% endtabs %}

## CoGroup

`CoGroup`转换联合处理两个数据集的组。这两个数据集按一个已定义的键进行分组，共享相同键的这两个数据集的组一起交给用户定义的`CoGroup`函数。如果对于特定的键，只有一个数据集具有组，则使用此组和空组调用`CoGroup`函数。一个同组函数可以分别遍历两个组的元素并返回任意数量的结果元素。

与Reduce，GroupReduce和Join类似，可以使用不同的键选择方法定义键。

### **DataSet上的CoGroup**

{% tabs %}
{% tab title="Java" %}
```java
// Some CoGroupFunction definition
class MyCoGrouper
         implements CoGroupFunction<Tuple2<String, Integer>, Tuple2<String, Double>, Double> {

  @Override
  public void coGroup(Iterable<Tuple2<String, Integer>> iVals,
                      Iterable<Tuple2<String, Double>> dVals,
                      Collector<Double> out) {

    Set<Integer> ints = new HashSet<Integer>();

    // add all Integer values in group to set
    for (Tuple2<String, Integer>> val : iVals) {
      ints.add(val.f1);
    }

    // multiply each Double value with each unique Integer values of group
    for (Tuple2<String, Double> val : dVals) {
      for (Integer i : ints) {
        out.collect(val.f1 * i);
      }
    }
  }
}

// [...]
DataSet<Tuple2<String, Integer>> iVals = // [...]
DataSet<Tuple2<String, Double>> dVals = // [...]
DataSet<Double> output = iVals.coGroup(dVals)
                         // group first DataSet on first tuple field
                         .where(0)
                         // group second DataSet on first tuple field
                         .equalTo(0)
                         // apply CoGroup function on each pair of groups
                         .with(new MyCoGrouper());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val iVals: DataSet[(String, Int)] = // [...]
val dVals: DataSet[(String, Double)] = // [...]

val output = iVals.coGroup(dVals).where(0).equalTo(0) {
  (iVals, dVals, out: Collector[Double]) =>
    val ints = iVals map { _._2 } toSet

    for (dVal <- dVals) {
      for (i <- ints) {
        out.collect(dVal._2 * i)
      }
    }
}

```
{% endtab %}

{% tab title="Python" %}
```python
class CoGroup(CoGroupFunction):
   def co_group(self, ivals, dvals, collector):
     ints = dict()
     # add all Integer values in group to set
     for value in ivals:
       ints[value[1]] = 1
     # multiply each Double value with each unique Integer values of group
     for value in dvals:
       for i in ints.keys():
         collector.collect(value[1] * i)


 output = ivals.co_group(dvals).where(0).equal_to(0).using(CoGroup())
```
{% endtab %}
{% endtabs %}

## Union

生成两个DataSet的并集，这两个DataSet必须属于同一类型。可以使用多个联合调用实现两个以上DataSet的并集，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<String, Integer>> vals1 = // [...]
DataSet<Tuple2<String, Integer>> vals2 = // [...]
DataSet<Tuple2<String, Integer>> vals3 = // [...]
DataSet<Tuple2<String, Integer>> unioned = vals1.union(vals2).union(vals3);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val vals1: DataSet[(String, Int)] = // [...]
val vals2: DataSet[(String, Int)] = // [...]
val vals3: DataSet[(String, Int)] = // [...]

val unioned = vals1.union(vals2).union(vals3)
```
{% endtab %}

{% tab title="Python" %}
```python
unioned = vals1.union(vals2).union(vals3)
```
{% endtab %}
{% endtabs %}

## Rebalance

均匀地重新平衡数据集的并行分区，以消除数据倾斜。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<String> in = // [...]
// rebalance DataSet and apply a Map transformation.
DataSet<Tuple2<String, String>> out = in.rebalance()
                                        .map(new Mapper());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val in: DataSet[String] = // [...]
// rebalance DataSet and apply a Map transformation.
val out = in.rebalance().map { ... }
```
{% endtab %}

{% tab title="Python" %}
```
Not supported.
```
{% endtab %}
{% endtabs %}

## Hash-Partition

对给定键上的DataSet进行Hash分区。键可以指定为位置键，表达式键和键选择器功能（请参阅[Reduce示例](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-grouped-dataset)以了解如何指定键）。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<String, Integer>> in = // [...]
// hash-partition DataSet by String value and apply a MapPartition transformation.
DataSet<Tuple2<String, String>> out = in.partitionByHash(0)
                                        .mapPartition(new PartitionMapper());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val in: DataSet[(String, Int)] = // [...]
// hash-partition DataSet by String value and apply a MapPartition transformation.
val out = in.partitionByHash(0).mapPartition { ... }
```
{% endtab %}

{% tab title="Python" %}
```
Not supported.
```
{% endtab %}
{% endtabs %}

## Range-Partition

对给定键的DataSet进行范围分区。键可以指定为位置键，表达式键和键选择器功能（请参阅[Reduce示例](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-grouped-dataset)以了解如何指定键）。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<String, Integer>> in = // [...]
// range-partition DataSet by String value and apply a MapPartition transformation.
DataSet<Tuple2<String, String>> out = in.partitionByRange(0)
                                        .mapPartition(new PartitionMapper());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val in: DataSet[(String, Int)] = // [...]
// range-partition DataSet by String value and apply a MapPartition transformation.
val out = in.partitionByRange(0).mapPartition { ... }
```
{% endtab %}

{% tab title="Python" %}
```text
Not supported.
```
{% endtab %}
{% endtabs %}

## Sort Partition

本地按指定顺序对指定字段上的DataSet的所有分区进行排序。可以将字段指定为字段表达式或字段位置（有关如何指定键，请参阅[Reduce示例](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-grouped-dataset)）。可以通过链接`sortPartition()`调用在多个字段上对分区进行排序。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<String, Integer>> in = // [...]
// Locally sort partitions in ascending order on the second String field and
// in descending order on the first String field.
// Apply a MapPartition transformation on the sorted partitions.
DataSet<Tuple2<String, String>> out = in.sortPartition(1, Order.ASCENDING)
                                        .sortPartition(0, Order.DESCENDING)
                                        .mapPartition(new PartitionMapper());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val in: DataSet[(String, Int)] = // [...]
// Locally sort partitions in ascending order on the second String field and
// in descending order on the first String field.
// Apply a MapPartition transformation on the sorted partitions.
val out = in.sortPartition(1, Order.ASCENDING)
            .sortPartition(0, Order.DESCENDING)
            .mapPartition { ... }
```
{% endtab %}

{% tab title="Python" %}
```
Not supported.
```
{% endtab %}
{% endtabs %}

## First-n

返回DataSet的前n个（任意）元素。First-n可以应用于常规DataSet，分组DataSet或分组排序DataSet。可以将分组键指定为键选择器功能或字段位置键（有关如何指定键，请参阅[Reduce示例](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#reduce-on-grouped-dataset)）。

{% tabs %}
{% tab title="Java" %}
```java
DataSet<Tuple2<String, Integer>> in = // [...]
// Return the first five (arbitrary) elements of the DataSet
DataSet<Tuple2<String, Integer>> out1 = in.first(5);

// Return the first two (arbitrary) elements of each String group
DataSet<Tuple2<String, Integer>> out2 = in.groupBy(0)
                                          .first(2);

// Return the first three elements of each String group ordered by the Integer field
DataSet<Tuple2<String, Integer>> out3 = in.groupBy(0)
                                          .sortGroup(1, Order.ASCENDING)
                                          .first(3);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val in: DataSet[(String, Int)] = // [...]
// Return the first five (arbitrary) elements of the DataSet
val out1 = in.first(5)

// Return the first two (arbitrary) elements of each String group
val out2 = in.groupBy(0).first(2)

// Return the first three elements of each String group ordered by the Integer field
val out3 = in.groupBy(0).sortGroup(1, Order.ASCENDING).first(3)
```
{% endtab %}

{% tab title="Python" %}
```python
Not supported.
```
{% endtab %}
{% endtabs %}

