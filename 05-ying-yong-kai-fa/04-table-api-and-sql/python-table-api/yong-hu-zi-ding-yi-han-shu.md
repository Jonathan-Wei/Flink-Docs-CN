# 用户自定义函数

用户定义函数是重要的功能，因为它们极大地扩展了Python Table API程序的表达能力。

## 标量函数

它支持在Python表API程序中使用Python标量函数。为了定义Python标量函数，可以在`pyflink.table.udf`中扩展基类ScalarFunction和实现一个评估方法。Python标量函数的行为是由评估方法eval定义的。评估方法可以支持变量参数，比如`eval(*args)`。

下面的示例演示如何定义自己的Python散列代码函数，在TableEnvironment中注册它，并在查询中调用它。注意，你可以通过构造函数配置你的标量函数之前，它被注册:

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

