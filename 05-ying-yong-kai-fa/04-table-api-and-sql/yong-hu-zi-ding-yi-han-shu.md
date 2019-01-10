# 用户自定义函数

## 注册用户定义的函数

## 标量函数

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

![](../../.gitbook/assets/udagg-mechanism.png)

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

## 运行集成UDF

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

