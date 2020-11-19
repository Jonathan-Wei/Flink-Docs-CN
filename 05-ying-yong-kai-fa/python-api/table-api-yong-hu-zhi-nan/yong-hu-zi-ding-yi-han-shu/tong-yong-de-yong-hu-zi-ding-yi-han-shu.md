# 通用的用户自定义函数

用户定义函数是重要的功能，因为它们极大地扩展了Python Table API程序的表达能力。

**注意：要**执行Python UDF，需要安装PyFlink的Python版本（3.5、3.6或3.7）。客户端和集群端都需要它。

## 标量函数

它支持在Python表API程序中使用Python标量函数。为了定义Python标量函数，可以在`pyflink.table.udf`中扩展基类ScalarFunction和实现一个评估方法。Python标量函数的行为是由评估方法eval定义的。评估方法可以支持变量参数，比如`eval(*args)`。

下面的示例演示如何定义自己的Python哈希码函数，如何在TableEnvironment中注册并在查询中调用它。请注意，你可以在注册之前通过构造函数配置标量函数：

```python
class HashCode(ScalarFunction):
  def __init__(self):
    self.factor = 12

  def eval(self, s):
    return hash(s) * self.factor

# use StreamTableEnvironment since Python UDF is not supported in the old planner under batch mode
table_env = StreamTableEnvironment.create(env)

# register the Python function
table_env.register_function("hash_code", udf(HashCode(), DataTypes.BIGINT(), DataTypes.BIGINT()))

# use the Python function in Python Table API
my_table.select("string, bigint, bigint.hash_code(), hash_code(bigint)")

# use the Python function in SQL API
table_env.sql_query("SELECT string, bigint, hash_code(bigint) FROM MyTable")
```

{% hint style="info" %}
注意：如果不使用RocksDB作为状态后端，则还可以通过将**python.fn-execution.memory.managed**设置为**true**，来配置python worker以使用taskmanager的托管内存。然后，无需设置配置**taskmanager.memory.task.off-heap.size**。
{% endhint %}

还支持在Python Table API程序中使用Java / Scala标量函数。

```python
'''
Java code:

// The Java class must have a public no-argument constructor and can be founded in current Java classloader.
public class HashCode extends ScalarFunction {
  private int factor = 12;

  public int eval(String s) {
      return s.hashCode() * factor;
  }
}
'''

table_env = BatchTableEnvironment.create(env)

# register the Java function
table_env.register_java_function("hash_code", "my.java.function.HashCode")

# use the Java function in Python Table API
my_table.select("string.hash_code(), hash_code(string)")

# use the Java function in SQL API
table_env.sql_query("SELECT string, bigint, hash_code(string) FROM MyTable")
```

{% hint style="info" %}
注意：如果不使用RocksDB作为状态后端，则还可以通过将**python.fn-execution.memory.managed**设置为**true**，来配置python worker以使用taskmanager的托管内存。然后，无需设置配置**taskmanager.memory.task.off-heap.size**。
{% endhint %}

除了扩展基类，还有许多方法可以定义Python标量函数`ScalarFunction`。以下示例显示了定义Python标量函数的不同方法，该函数将两列bigint作为输入参数，并将其总和作为结果返回。

```python
# option 1: extending the base class `ScalarFunction`
class Add(ScalarFunction):
  def eval(self, i, j):
    return i + j

add = udf(Add(), [DataTypes.BIGINT(), DataTypes.BIGINT()], DataTypes.BIGINT())

# option 2: Python function
@udf(input_types=[DataTypes.BIGINT(), DataTypes.BIGINT()], result_type=DataTypes.BIGINT())
def add(i, j):
  return i + j

# option 3: lambda function
add = udf(lambda i, j: i + j, [DataTypes.BIGINT(), DataTypes.BIGINT()], DataTypes.BIGINT())

# option 4: callable function
class CallableAdd(object):
  def __call__(self, i, j):
    return i + j

add = udf(CallableAdd(), [DataTypes.BIGINT(), DataTypes.BIGINT()], DataTypes.BIGINT())

# option 5: partial function
def partial_add(i, j, k):
  return i + j + k

add = udf(functools.partial(partial_add, k=1), [DataTypes.BIGINT(), DataTypes.BIGINT()],
          DataTypes.BIGINT())

# register the Python function
table_env.register_function("add", add)
# use the function in Python Table API
my_table.select("add(a, b)")
```

##  表函数（Table Function）

与Python用户定义的标量函数相似，用户定义的表函数将零个，一个或多个标量值作为输入参数。但是，与标量函数相比，它可以返回任意数量的行作为输出，而不是单个值。Python UDTF的返回类型可以是Iterable，Iterator或generator类型。

以下示例显示了如何定义自己的Python多重发射函数，如何在TableEnvironment中注册它以及如何在查询中调用它。

```java
class Split(TableFunction):
    def eval(self, string):
        for s in string.split(" "):
            yield s, len(s)

env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)
my_table = ...  # type: Table, table schema: [a: String]

# configure the off-heap memory of current taskmanager to enable the python worker uses off-heap memory.
table_env.get_config().get_configuration().set_string("taskmanager.memory.task.off-heap.size", '80m')

# register the Python Table Function
table_env.register_function("split", udtf(Split(), DataTypes.STRING(), [DataTypes.STRING(), DataTypes.INT()]))

# use the Python Table Function in Python Table API
my_table.join_lateral("split(a) as (word, length)")
my_table.left_outer_join_lateral("split(a) as (word, length)")

# use the Python Table function in SQL API
table_env.sql_query("SELECT a, word, length FROM MyTable, LATERAL TABLE(split(a)) as T(word, length)")
table_env.sql_query("SELECT a, word, length FROM MyTable LEFT JOIN LATERAL TABLE(split(a)) as T(word, length) ON TRUE")
```

{% hint style="info" %}
注意：如果不使用RocksDB作为状态后端，则还可以通过将**python.fn-execution.memory.managed**设置为**true**，来配置python worker以使用taskmanager的托管内存。然后，无需设置配置**taskmanager.memory.task.off-heap.size**。
{% endhint %}

它还支持在Python Table API程序中使用Java / Scala表函数。

```java
'''
Java code:

// The generic type "Tuple2<String, Integer>" determines the schema of the returned table as (String, Integer).
// The java class must have a public no-argument constructor and can be founded in current java classloader.
public class Split extends TableFunction<Tuple2<String, Integer>> {
    private String separator = " ";
    
    public void eval(String str) {
        for (String s : str.split(separator)) {
            // use collect(...) to emit a row
            collect(new Tuple2<String, Integer>(s, s.length()));
        }
    }
}
'''

env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)
my_table = ...  # type: Table, table schema: [a: String]

# configure the off-heap memory of current taskmanager to enable the python worker uses off-heap memory.
table_env.get_config().get_configuration().set_string("taskmanager.memory.task.off-heap.size", '80m')

# Register the java function.
table_env.register_java_function("split", "my.java.function.Split")

# Use the table function in the Python Table API. "as" specifies the field names of the table.
my_table.join_lateral("split(a) as (word, length)").select("a, word, length")
my_table.left_outer_join_lateral("split(a) as (word, length)").select("a, word, length")

# Register the python function.

# Use the table function in SQL with LATERAL and TABLE keywords.
# CROSS JOIN a table function (equivalent to "join" in Table API).
table_env.sql_query("SELECT a, word, length FROM MyTable, LATERAL TABLE(split(a)) as T(word, length)")
# LEFT JOIN a table function (equivalent to "left_outer_join" in Table API).
table_env.sql_query("SELECT a, word, length FROM MyTable LEFT JOIN LATERAL TABLE(split(a)) as T(word, length) ON TRUE")
```

{% hint style="info" %}
注意：如果不使用RocksDB作为状态后端，则还可以通过将**python.fn-execution.memory.managed**设置为**true**，来配置python worker以使用taskmanager的托管内存。然后，无需设置配置**taskmanager.memory.task.off-heap.size**。
{% endhint %}

像Python标量函数一样，您可以使用上述五种方式来定义Python TableFunction。

{% hint style="info" %}
注意：唯一的区别是Python Table Functions的返回类型必须是可迭代的，迭代器或生成器。
{% endhint %}

```scala
# option 1: generator function
@udtf(input_types=DataTypes.BIGINT(), result_types=DataTypes.BIGINT())
def generator_func(x):
      yield 1
      yield 2

# option 2: return iterator
@udtf(input_types=DataTypes.BIGINT(), result_types=DataTypes.BIGINT())
def iterator_func(x):
      return range(5)

# option 3: return iterable
@udtf(input_types=DataTypes.BIGINT(), result_types=DataTypes.BIGINT())
def iterable_func(x):
      result = [1, 2, 3]
      return result

table_env.register_function("iterable_func", iterable_func)
table_env.register_function("iterator_func", iterator_func)
table_env.register_function("generator_func", generator_func)
```

