# LOAD语句

LOAD语句用于加载内置或用户定义的模块

## 运行LOAD语句

{% tabs %}
{% tab title="Java" %}
LOAD语句可以通过TableEnvironment的executeSql\(\)方法执行。对于成功的LOAD操作，executeSql\(\)方法返回' OK ';否则，它将抛出异常。

下面的例子展示了如何在TableEnvironment中运行LOAD语句。

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

// load a hive module
tEnv.executeSql("LOAD MODULE hive WITH ('hive-version' = '3.1.2')");
tEnv.executeSql("SHOW MODULES").print();
// +-------------+
// | module name |
// +-------------+
// |        core |
// |        hive |
// +-------------+
```
{% endtab %}

{% tab title="Scala" %}
LOAD语句可以通过TableEnvironment的executeSql\(\)方法执行。对于成功的LOAD操作，executeSql\(\)方法返回' OK ';否则，它将抛出异常。

下面的例子展示了如何在TableEnvironment中运行LOAD语句。

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
val tEnv = StreamTableEnvironment.create(env)

// load a hive module
tEnv.executeSql("LOAD MODULE hive WITH ('hive-version' = '3.1.2')")
tEnv.executeSql("SHOW MODULES").print()
// +-------------+
// | module name |
// +-------------+
// |        core |
// |        hive |
// +-------------+
```
{% endtab %}

{% tab title="Python" %}
LOAD语句可以通过TableEnvironment的executeSql\(\)方法执行。对于成功的LOAD操作，executeSql\(\)方法返回' OK ';否则，它将抛出异常。

下面的例子展示了如何在TableEnvironment中运行LOAD语句。

```python
settings = EnvironmentSettings.new_instance()...
table_env = StreamTableEnvironment.create(env, settings)

# load a hive module
table_env.execute_sql("LOAD MODULE hive WITH ('hive-version' = '3.1.2')")
table_env.execute_sql("SHOW MODULES").print()
# +-------------+
# | module name |
# +-------------+
# |        core |
# |        hive |
# +-------------+
```
{% endtab %}

{% tab title="SQL CLI" %}
LOAD语句可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sqlclient/)执行。

以下示例显示如何在SQL CLI中运行LOAD语句。

```sql
Flink SQL> LOAD MODULE hive WITH ('hive-version' = '3.1.2');
[INFO] Load module succeeded!

Flink SQL> SHOW MODULES;
+-------------+
| module name |
+-------------+
|        core |
|        hive |
+-------------+
```
{% endtab %}
{% endtabs %}

## LOAD MODULE

以下语法概述了可用的语法：

```sql
LOAD MODULE module_name [WITH ('key1' = 'val1', 'key2' = 'val2', ...)]
```

{% hint style="warning" %}
 `module_name`是一个简单的标识符。区分大小写，并且应与模块工厂中定义的模块类型相同，因为它用于执行模块发现。属性（'key1'='val1'，'key2'='val2'，…）是一个映射，它包含一组键值对（除了键“type”）并传递给发现服务以实例化相应的模块。
{% endhint %}

