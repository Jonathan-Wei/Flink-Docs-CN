# 用户自定义函数

用户定义的函数是一个重要的特性，因为它们展示扩展了查询的表达能力。

## 注册用户定义的函数

在大多数情况下，必须先注册用户定义的函数，然后才能在查询中使用它。没有必要为Scala Table API注册函数。

`TableEnvironment`通过调用`registerFunction()`方法来注册函数。注册用户定义的函数时，将插入到`TableEnvironment`函数目录中，以便Table API或SQL解析器可以识别并正确转换。

请在下面的子会话中找到如何注册以及如何调用每种类型的用户定义函数\(`ScalarFunction`、`TableFunction`和`AggregateFunction`\)的详细示例。

## 标量函数

如果内置函数中不包含必需的标量函数，则可以为Table API和SQL定义自定义的用户定义的标量函数。用户定义的标量函数将零个，一个或多个标量值映射到新的标量值。

为了定义标量函数，必须扩展`org.apache.flink.Table.function`中的基类`ScalarFunction`和实现\(一个或多个\)评估方法。标量函数的行为由评估法决定。必须公开声明一个评估方法并命名为`eval`。计算方法的参数类型和返回类型也决定标量函数的参数和返回类型。还可以通过实现名为`eval`的多个方法来重载评估方法。评估方法也可以支持变量参数，比如`eval(String…str)`。

以下示例显示如何定义自己的哈希代码函数，在TableEnvironment中注册它，并在查询中调用它。请注意，可以在注册之前通过构造函数配置标量函数：

{% tabs %}
{% tab title="Java" %}
```java
public class HashCode extends ScalarFunction {
  private int factor = 12;
  
  public HashCode(int factor) {
      this.factor = factor;
  }
  
  public int eval(String s) {
      return s.hashCode() * factor;
  }
}

BatchTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// register the function
tableEnv.registerFunction("hashCode", new HashCode(10));

// use the function in Java Table API
myTable.select("string, string.hashCode(), hashCode(string)");

// use the function in SQL API
tableEnv.sqlQuery("SELECT string, hashCode(string) FROM MyTable");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// must be defined in static/object context
class HashCode(factor: Int) extends ScalarFunction {
  def eval(s: String): Int = {
    s.hashCode() * factor
  }
}

val tableEnv = TableEnvironment.getTableEnvironment(env)

// use the function in Scala Table API
val hashCode = new HashCode(10)
myTable.select('string, hashCode('string))

// register and use the function in SQL
tableEnv.registerFunction("hashCode", new HashCode(10))
tableEnv.sqlQuery("SELECT string, hashCode(string) FROM MyTable")
```
{% endtab %}
{% endtabs %}

默认情况下，评估方法的结果类型由Flink的类型提取工具决定。对于基本类型或简单pojo，这已经足够了，但是对于更复杂的、自定义的或复合类型，这可能是错误的。在这些情况下，可以通过覆盖`ScalarFunction#getResultType()`手动定义结果类型的类型信息。

下面的示例显示了一个高级示例，它采用内部时间戳表示，并将内部时间戳表示返回为长值。通过重写`ScalarFunction getResultType()`，我们定义返回的长值应该由代码生成解释为`Types.TIMESTAMP`。

{% tabs %}
{% tab title="Java" %}
```java

public static class TimestampModifier extends ScalarFunction {
  public long eval(long t) {
    return t % 1000;
  }

  public TypeInformation<?> getResultType(Class<?>[] signature) {
    return Types.SQL_TIMESTAMP;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
object TimestampModifier extends ScalarFunction {
  def eval(t: Long): Long = {
    t % 1000
  }

  override def getResultType(signature: Array[Class[_]]): TypeInformation[_] = {
    Types.TIMESTAMP
  }
}
```
{% endtab %}
{% endtabs %}

## 表函数

与用户定义的标量函数类似，用户定义的表函数将零个，一个或多个标量值作为输入参数。但是，与标量函数相比，它可以返回任意数量的行作为输出而不是单个值。返回的行可以包含一个或多个列。

为了定义表函数，必须扩展`org.apache.flink.table.function`中的基类`TableFunction`和实现\(一个或多个\)评估方法。表函数的行为由其计算方法决定。一个求值方法必须声明为`public`并命名为`eval`。可以通过实现名为eval的多个方法来重载TableFunction。评价方法的参数类型决定了表函数的所有有效参数。求值方法也可以支持变量参数，比如eval\(String…str\)。返回的表的类型由TableFunction的泛型类型决定。评估方法使用受保护的collect\(T\)方法发出输出行。

在表API中，表函数与`.join(Table)`或`. leftouterjoin(Table)`一起使用。关联`operator (cross)`将外部表\(操作符左侧的表\)中的每一行与表值函数\(操作符右侧的表\)生成的所有行关联起来。`leftOuterJoin`操作符将外部表\(操作符左侧的表\)中的每一行与表值函数\(操作符右侧的表\)生成的所有行连接起来，并保留表函数返回空表的外部行。在SQL中使用`LATERAL TABLE(<TableFunction>)`CROSS JOIN和LEFT JOIN以及ON TRUE连接条件（参见下面的示例）。

以下示例展示如何定义表值函数，在`TableEnvironment`中注册它，并在查询中调用它。请注意，您可以在注册之前通过构造函数配置表函数：

{% tabs %}
{% tab title="Java" %}
```java
// The generic type "Tuple2<String, Integer>" determines the schema of the returned table as (String, Integer).
public class Split extends TableFunction<Tuple2<String, Integer>> {
    private String separator = " ";
    
    public Split(String separator) {
        this.separator = separator;
    }
    
    public void eval(String str) {
        for (String s : str.split(separator)) {
            // use collect(...) to emit a row
            collect(new Tuple2<String, Integer>(s, s.length()));
        }
    }
}

BatchTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);
Table myTable = ...         // table schema: [a: String]

// Register the function.
tableEnv.registerFunction("split", new Split("#"));

// Use the table function in the Java Table API. "as" specifies the field names of the table.
myTable.join(new Table(tableEnv, "split(a) as (word, length)"))
    .select("a, word, length");
myTable.leftOuterJoin(new Table(tableEnv, "split(a) as (word, length)"))
    .select("a, word, length");

// Use the table function in SQL with LATERAL and TABLE keywords.
// CROSS JOIN a table function (equivalent to "join" in Table API).
tableEnv.sqlQuery("SELECT a, word, length FROM MyTable, LATERAL TABLE(split(a)) as T(word, length)");
// LEFT JOIN a table function (equivalent to "leftOuterJoin" in Table API).
tableEnv.sqlQuery("SELECT a, word, length FROM MyTable LEFT JOIN LATERAL TABLE(split(a)) as T(word, length) ON TRUE");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// The generic type "(String, Int)" determines the schema of the returned table as (String, Integer).
class Split(separator: String) extends TableFunction[(String, Int)] {
  def eval(str: String): Unit = {
    // use collect(...) to emit a row.
    str.split(separator).foreach(x => collect((x, x.length)))
  }
}

val tableEnv = TableEnvironment.getTableEnvironment(env)
val myTable = ...         // table schema: [a: String]

// Use the table function in the Scala Table API (Note: No registration required in Scala Table API).
val split = new Split("#")
// "as" specifies the field names of the generated table.
myTable.join(split('a) as ('word, 'length)).select('a, 'word, 'length)
myTable.leftOuterJoin(split('a) as ('word, 'length)).select('a, 'word, 'length)

// Register the table function to use it in SQL queries.
tableEnv.registerFunction("split", new Split("#"))

// Use the table function in SQL with LATERAL and TABLE keywords.
// CROSS JOIN a table function (equivalent to "join" in Table API)
tableEnv.sqlQuery("SELECT a, word, length FROM MyTable, LATERAL TABLE(split(a)) as T(word, length)")
// LEFT JOIN a table function (equivalent to "leftOuterJoin" in Table API)
tableEnv.sqlQuery("SELECT a, word, length FROM MyTable LEFT JOIN LATERAL TABLE(split(a)) as T(word, length) ON TRUE")
```
{% endtab %}
{% endtabs %}

请注意，POJO类型没有确定性字段顺序。因此，您无法`AS`重命名由表函数返回的POJO字段。

默认情况下，表函数的结果类型由Flink的自动类型提取工具决定。这对于基本类型和简单pojo很有效，但是对于更复杂的、自定义的或复合类型可能是错误的。在这种情况下，可以通过覆盖`TableFunction#getResultType()`手动指定结果的类型，后者返回其类型信息。

下面的示例显示了一个表函数的示例，该函数返回一个需要显式类型信息的行类型。通过覆盖`TableFunction#getResultType()`，我们定义返回的表类型应该是`RowTypeInfo(String, Integer)`。

{% tabs %}
{% tab title="Java" %}
```java
public class CustomTypeSplit extends TableFunction<Row> {
    public void eval(String str) {
        for (String s : str.split(" ")) {
            Row row = new Row(2);
            row.setField(0, s);
            row.setField(1, s.length());
            collect(row);
        }
    }

    @Override
    public TypeInformation<Row> getResultType() {
        return Types.ROW(Types.STRING(), Types.INT());
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class CustomTypeSplit extends TableFunction[Row] {
  def eval(str: String): Unit = {
    str.split(" ").foreach({ s =>
      val row = new Row(2)
      row.setField(0, s)
      row.setField(1, s.length)
      collect(row)
    })
  }

  override def getResultType: TypeInformation[Row] = {
    Types.ROW(Types.STRING, Types.INT)
  }
}
```
{% endtab %}
{% endtabs %}

## 聚合函数

用户定义的聚合函数（UDAGG）将一个表（一个或多个具有一个或多个属性的行）聚合为标量值。

![](../../../.gitbook/assets/udagg-mechanism.png)

上图显示了一个聚合的例子。假设您有一个包含饮料数据的表。该表由三列\(id、name和price\)和5行组成。假设你需要找到桌子上所有饮料中价格最高的，即，执行max\(\)聚合。您需要检查这5行中的每一行，结果将是一个数字值。

用户定义的聚合函数是通过扩展`AggregateFunction`类实现的。`AggregateFunction`的工作原理如下。首先，需要一个累加器，它是保存聚合的中间结果的数据结构。一个空的累加器是通过调用`AggregateFunction`的`createAccumulator()`方法创建的。然后，对每个输入行调用函数的`collect()`方法来更新累加器。处理完所有行之后，调用函数的`getValue()`方法来计算并返回最终结果。

每个`AggregateFunction`都必须使用以下方法：

* `createAccumulator()`
* `accumulate()`
* `getValue()`

Flink的类型提取工具可能无法识别复杂的数据类型，例如，如果它们不是基本类型或简单的`POJO`。因此，与`ScalarFunction`和`TableFunction`类似，`AggregateFunction`提供了一些方法来指定结果类型的类型信息\(通过`AggregateFunction#getResultType()`\)和累加器的类型\(通过`AggregateFunction#getAccumulatorType()`\)。

除了上述方法之外，还有一些压缩的方法可以选择性地实现。虽然其中一些方法允许系统更高效地执行查询，但是其他方法对于某些用例是必需的。例如，如果聚合函数应该应用于会话组窗口的上下文中，则merge\(\)方法是强制性的\(当观察到“连接”它们的行时，需要连接两个会话窗口的累加器\)。

根据用例的不同，AggregateFunction需要以下方法:

* retract\(\):在有界`OVER`窗口上聚合是必需的。
* merge\(\):许多批处理聚合和会话窗口聚合都需要。
* resetAccumulator\(\):许多批处理聚合都需要。

`AggregateFunction`的所有方法都必须声明为`public`的，而不是`static`，并按照上面提到的名称命名。`createAccumulator`、`getValue`、`getResultType`和`getAccumulatorType`方法是在`AggregateFunction`抽象类中定义的，而其他方法是收缩的方法。为了定义聚合函数，必须扩展基类`org.apache.flink.table.functions.AggregateFunction`并实现一个\(或多个\)累计方法。可以使用不同的参数类型重载方法累计，并支持变量参数。

下面给出了`AggregateFunction`所有方法的详细文档。

{% tabs %}
{% tab title="Java" %}
```java
/**
  * Base class for aggregation functions. 
  *
  * @param <T>   the type of the aggregation result
  * @param <ACC> the type of the aggregation accumulator. The accumulator is used to keep the
  *             aggregated values which are needed to compute an aggregation result.
  *             AggregateFunction represents its state using accumulator, thereby the state of the
  *             AggregateFunction must be put into the accumulator.
  */
public abstract class AggregateFunction<T, ACC> extends UserDefinedFunction {

  /**
    * Creates and init the Accumulator for this [[AggregateFunction]].
    *
    * @return the accumulator with the initial value
    */
  public ACC createAccumulator(); // MANDATORY

  /** Processes the input values and update the provided accumulator instance. The method
    * accumulate can be overloaded with different custom types and arguments. An AggregateFunction
    * requires at least one accumulate() method.
    *
    * @param accumulator           the accumulator which contains the current aggregated results
    * @param [user defined inputs] the input value (usually obtained from a new arrived data).
    */
  public void accumulate(ACC accumulator, [user defined inputs]); // MANDATORY

  /**
    * Retracts the input values from the accumulator instance. The current design assumes the
    * inputs are the values that have been previously accumulated. The method retract can be
    * overloaded with different custom types and arguments. This function must be implemented for
    * datastream bounded over aggregate.
    *
    * @param accumulator           the accumulator which contains the current aggregated results
    * @param [user defined inputs] the input value (usually obtained from a new arrived data).
    */
  public void retract(ACC accumulator, [user defined inputs]); // OPTIONAL

  /**
    * Merges a group of accumulator instances into one accumulator instance. This function must be
    * implemented for datastream session window grouping aggregate and dataset grouping aggregate.
    *
    * @param accumulator  the accumulator which will keep the merged aggregate results. It should
    *                     be noted that the accumulator may contain the previous aggregated
    *                     results. Therefore user should not replace or clean this instance in the
    *                     custom merge method.
    * @param its          an [[java.lang.Iterable]] pointed to a group of accumulators that will be
    *                     merged.
    */
  public void merge(ACC accumulator, java.lang.Iterable<ACC> its); // OPTIONAL

  /**
    * Called every time when an aggregation result should be materialized.
    * The returned value could be either an early and incomplete result
    * (periodically emitted as data arrive) or the final result of the
    * aggregation.
    *
    * @param accumulator the accumulator which contains the current
    *                    aggregated results
    * @return the aggregation result
    */
  public T getValue(ACC accumulator); // MANDATORY

  /**
    * Resets the accumulator for this [[AggregateFunction]]. This function must be implemented for
    * dataset grouping aggregate.
    *
    * @param accumulator  the accumulator which needs to be reset
    */
  public void resetAccumulator(ACC accumulator); // OPTIONAL

  /**
    * Returns true if this AggregateFunction can only be applied in an OVER window.
    *
    * @return true if the AggregateFunction requires an OVER window, false otherwise.
    */
  public Boolean requiresOver = false; // PRE-DEFINED

  /**
    * Returns the TypeInformation of the AggregateFunction's result.
    *
    * @return The TypeInformation of the AggregateFunction's result or null if the result type
    *         should be automatically inferred.
    */
  public TypeInformation<T> getResultType = null; // PRE-DEFINED

  /**
    * Returns the TypeInformation of the AggregateFunction's accumulator.
    *
    * @return The TypeInformation of the AggregateFunction's accumulator or null if the
    *         accumulator type should be automatically inferred.
    */
  public TypeInformation<T> getAccumulatorType = null; // PRE-DEFINED
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
/**
  * Base class for aggregation functions. 
  *
  * @tparam T   the type of the aggregation result
  * @tparam ACC the type of the aggregation accumulator. The accumulator is used to keep the
  *             aggregated values which are needed to compute an aggregation result.
  *             AggregateFunction represents its state using accumulator, thereby the state of the
  *             AggregateFunction must be put into the accumulator.
  */
abstract class AggregateFunction[T, ACC] extends UserDefinedFunction {
  /**
    * Creates and init the Accumulator for this [[AggregateFunction]].
    *
    * @return the accumulator with the initial value
    */
  def createAccumulator(): ACC // MANDATORY

  /**
    * Processes the input values and update the provided accumulator instance. The method
    * accumulate can be overloaded with different custom types and arguments. An AggregateFunction
    * requires at least one accumulate() method.
    *
    * @param accumulator           the accumulator which contains the current aggregated results
    * @param [user defined inputs] the input value (usually obtained from a new arrived data).
    */
  def accumulate(accumulator: ACC, [user defined inputs]): Unit // MANDATORY

  /**
    * Retracts the input values from the accumulator instance. The current design assumes the
    * inputs are the values that have been previously accumulated. The method retract can be
    * overloaded with different custom types and arguments. This function must be implemented for
    * datastream bounded over aggregate.
    *
    * @param accumulator           the accumulator which contains the current aggregated results
    * @param [user defined inputs] the input value (usually obtained from a new arrived data).
    */
  def retract(accumulator: ACC, [user defined inputs]): Unit // OPTIONAL

  /**
    * Merges a group of accumulator instances into one accumulator instance. This function must be
    * implemented for datastream session window grouping aggregate and dataset grouping aggregate.
    *
    * @param accumulator  the accumulator which will keep the merged aggregate results. It should
    *                     be noted that the accumulator may contain the previous aggregated
    *                     results. Therefore user should not replace or clean this instance in the
    *                     custom merge method.
    * @param its          an [[java.lang.Iterable]] pointed to a group of accumulators that will be
    *                     merged.
    */
  def merge(accumulator: ACC, its: java.lang.Iterable[ACC]): Unit // OPTIONAL
  
  /**
    * Called every time when an aggregation result should be materialized.
    * The returned value could be either an early and incomplete result
    * (periodically emitted as data arrive) or the final result of the
    * aggregation.
    *
    * @param accumulator the accumulator which contains the current
    *                    aggregated results
    * @return the aggregation result
    */
  def getValue(accumulator: ACC): T // MANDATORY

  /**
    * Resets the accumulator for this [[AggregateFunction]]. This function must be implemented for
    * dataset grouping aggregate.
    *
    * @param accumulator  the accumulator which needs to be reset
    */
  def resetAccumulator(accumulator: ACC): Unit // OPTIONAL

  /**
    * Returns true if this AggregateFunction can only be applied in an OVER window.
    *
    * @return true if the AggregateFunction requires an OVER window, false otherwise.
    */
  def requiresOver: Boolean = false // PRE-DEFINED

  /**
    * Returns the TypeInformation of the AggregateFunction's result.
    *
    * @return The TypeInformation of the AggregateFunction's result or null if the result type
    *         should be automatically inferred.
    */
  def getResultType: TypeInformation[T] = null // PRE-DEFINED

  /**
    * Returns the TypeInformation of the AggregateFunction's accumulator.
    *
    * @return The TypeInformation of the AggregateFunction's accumulator or null if the
    *         accumulator type should be automatically inferred.
    */
  def getAccumulatorType: TypeInformation[ACC] = null // PRE-DEFINED
}
```
{% endtab %}
{% endtabs %}

以下示例说明如何操作

* 定义一个`AggregateFunction`计算给定列的加权平均值，
* 在`TableEnvironment`和中注册函数
* 在查询中使用该函数。

要计算加权平均值，累加器需要存储所有累积数据的加权和和计数。在我们的示例中，我们定义了一个类`WeightedAvgAccum`作为累加器。累加器由Flink的检查点机制自动备份，并在出现故障时恢复，以确保精确的一次语义。

`WeightedAvg AggregateFunction`的`collect()`方法有三个输入。第一个是`WeightedAvgAccum` `accumulator`，另外两个是用户定义的输入:输入值`ivalue`和输入`iweight`的权重。虽然`retract()`、`merge()`和r`esetAccumulator()`方法不是大多数聚合类型的强制方法，但是我们在下面提供了它们作为示例。请注意，我们在Scala示例中使用了Java基本类型并定义了`getResultType()`和`getAccumulatorType()`方法，因为Flink类型提取对于Scala类型不是很有效。

{% tabs %}
{% tab title="Java" %}
```java
/**
 * Accumulator for WeightedAvg.
 */
public static class WeightedAvgAccum {
    public long sum = 0;
    public int count = 0;
}

/**
 * Weighted Average user-defined aggregate function.
 */
public static class WeightedAvg extends AggregateFunction<Long, WeightedAvgAccum> {

    @Override
    public WeightedAvgAccum createAccumulator() {
        return new WeightedAvgAccum();
    }

    @Override
    public Long getValue(WeightedAvgAccum acc) {
        if (acc.count == 0) {
            return null;
        } else {
            return acc.sum / acc.count;
        }
    }

    public void accumulate(WeightedAvgAccum acc, long iValue, int iWeight) {
        acc.sum += iValue * iWeight;
        acc.count += iWeight;
    }

    public void retract(WeightedAvgAccum acc, long iValue, int iWeight) {
        acc.sum -= iValue * iWeight;
        acc.count -= iWeight;
    }
    
    public void merge(WeightedAvgAccum acc, Iterable<WeightedAvgAccum> it) {
        Iterator<WeightedAvgAccum> iter = it.iterator();
        while (iter.hasNext()) {
            WeightedAvgAccum a = iter.next();
            acc.count += a.count;
            acc.sum += a.sum;
        }
    }
    
    public void resetAccumulator(WeightedAvgAccum acc) {
        acc.count = 0;
        acc.sum = 0L;
    }
}

// register function
StreamTableEnvironment tEnv = ...
tEnv.registerFunction("wAvg", new WeightedAvg());

// use function
tEnv.sqlQuery("SELECT user, wAvg(points, level) AS avgPoints FROM userScores GROUP BY user");
```
{% endtab %}

{% tab title="Scala" %}
```scala
import java.lang.{Long => JLong, Integer => JInteger}
import org.apache.flink.api.java.tuple.{Tuple1 => JTuple1}
import org.apache.flink.api.java.typeutils.TupleTypeInfo
import org.apache.flink.table.api.Types
import org.apache.flink.table.functions.AggregateFunction

/**
 * Accumulator for WeightedAvg.
 */
class WeightedAvgAccum extends JTuple1[JLong, JInteger] {
  sum = 0L
  count = 0
}

/**
 * Weighted Average user-defined aggregate function.
 */
class WeightedAvg extends AggregateFunction[JLong, CountAccumulator] {

  override def createAccumulator(): WeightedAvgAccum = {
    new WeightedAvgAccum
  }
  
  override def getValue(acc: WeightedAvgAccum): JLong = {
    if (acc.count == 0) {
        null
    } else {
        acc.sum / acc.count
    }
  }
  
  def accumulate(acc: WeightedAvgAccum, iValue: JLong, iWeight: JInteger): Unit = {
    acc.sum += iValue * iWeight
    acc.count += iWeight
  }

  def retract(acc: WeightedAvgAccum, iValue: JLong, iWeight: JInteger): Unit = {
    acc.sum -= iValue * iWeight
    acc.count -= iWeight
  }
    
  def merge(acc: WeightedAvgAccum, it: java.lang.Iterable[WeightedAvgAccum]): Unit = {
    val iter = it.iterator()
    while (iter.hasNext) {
      val a = iter.next()
      acc.count += a.count
      acc.sum += a.sum
    }
  }

  def resetAccumulator(acc: WeightedAvgAccum): Unit = {
    acc.count = 0
    acc.sum = 0L
  }

  override def getAccumulatorType: TypeInformation[WeightedAvgAccum] = {
    new TupleTypeInfo(classOf[WeightedAvgAccum], Types.LONG, Types.INT)
  }

  override def getResultType: TypeInformation[JLong] = Types.LONG
}

// register function
val tEnv: StreamTableEnvironment = ???
tEnv.registerFunction("wAvg", new WeightedAvg())

// use function
tEnv.sqlQuery("SELECT user, wAvg(points, level) AS avgPoints FROM userScores GROUP BY user")
```
{% endtab %}
{% endtabs %}

## 实现UDF的最佳实践

Table API和SQL代码生成在内部尝试尽可能地使用原始值。用户定义的函数可以通过对象创建，转换和（un）装箱引入很多开销。因此，强烈建议将参数和结果类型声明为基本类型而不是它们的盒装类。`Types.DATE` 和 `Types.TIME` 可以表示为 `int.Types.TIMESTAMP` 或`long`.

我们建议用户定义的函数应该由Java而不是Scala编写，因为Scala类型对Flink的类型提取器构成了挑战。

## 运行集成UDF

有时，用户定义的函数可能需要获取全局运行时信息，或者在实际工作之前进行一些设置/清理工作。用户定义的函数提供`open()`和`close()`方法可以被覆盖，并提供与`RichFunction`DataSet或DataStream API中的方法类似的功能。

`open()`方法在求值方法之前调用一次。最后一次调用求值方法之后的`close()`方法。

`open()`方法提供一个`FunctionContext`，其中包含有关用户定义函数在其中执行的上下文的信息，例如度量组、分布式缓存文件或全局作业参数。

通过调用以下相应的方法可以获得以下信息`FunctionContext`：

| 方法 | 描述 |
| :--- | :--- |
| `getMetricGroup()` | 此并行子任务的度量标准组。 |
| `getCachedFile(name)` | 分布式缓存文件的本地临时文件副本。 |
| `getJobParameter(name, defaultValue)` | 与给定键关联的全局作业参数值。 |

下面的示例片段展示了如何在标量函数中使用`FunctionContext`来访问全局作业参数:

{% tabs %}
{% tab title="Java" %}
```java
public class HashCode extends ScalarFunction {

    private int factor = 0;

    @Override
    public void open(FunctionContext context) throws Exception {
        // access "hashcode_factor" parameter
        // "12" would be the default value if parameter does not exist
        factor = Integer.valueOf(context.getJobParameter("hashcode_factor", "12")); 
    }

    public int eval(String s) {
        return s.hashCode() * factor;
    }
}

ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// set job parameter
Configuration conf = new Configuration();
conf.setString("hashcode_factor", "31");
env.getConfig().setGlobalJobParameters(conf);

// register the function
tableEnv.registerFunction("hashCode", new HashCode());

// use the function in Java Table API
myTable.select("string, string.hashCode(), hashCode(string)");

// use the function in SQL
tableEnv.sqlQuery("SELECT string, HASHCODE(string) FROM MyTable");
```
{% endtab %}

{% tab title="Scala" %}
```scala
object hashCode extends ScalarFunction {

  var hashcode_factor = 12

  override def open(context: FunctionContext): Unit = {
    // access "hashcode_factor" parameter
    // "12" would be the default value if parameter does not exist
    hashcode_factor = context.getJobParameter("hashcode_factor", "12").toInt
  }

  def eval(s: String): Int = {
    s.hashCode() * hashcode_factor
  }
}

val tableEnv = TableEnvironment.getTableEnvironment(env)

// use the function in Scala Table API
myTable.select('string, hashCode('string))

// register and use the function in SQL
tableEnv.registerFunction("hashCode", hashCode)
tableEnv.sqlQuery("SELECT string, HASHCODE(string) FROM MyTable")
```
{% endtab %}
{% endtabs %}

