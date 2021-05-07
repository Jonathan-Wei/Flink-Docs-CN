# UNLOAD语句

UNLOAD语句用于卸载内置或用户定义的模块。

## 运行UNLOAD语句

{% tabs %}
{% tab title="Java" %}
LOAD语句可以通过TableEnvironment的executeSql\(\)方法执行。对于成功的LOAD操作，executeSql\(\)方法返回' OK ';否则，它将抛出异常。

下面的例子展示了如何在TableEnvironment中运行LOAD语句。

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

// unload a core module
tEnv.executeSql("UNLOAD MODULE core");
tEnv.executeSql("SHOW MODULES").print();
// Empty set
```
{% endtab %}

{% tab title="SQL CLI" %}
LOAD语句可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sqlclient/)执行。

以下示例显示如何在SQL CLI中运行LOAD语句。

```sql
Flink SQL> UNLOAD MODULE core;
[INFO] Unload module succeeded!

Flink SQL> SHOW MODULES;
Empty set
```
{% endtab %}
{% endtabs %}

## UNLOAD MODULE

以下语法概述了可用的语法：

```text
UNLOAD MODULE module_name
```

