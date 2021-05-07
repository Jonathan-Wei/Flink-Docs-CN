# RESET语句

`RESET` 语句用于将配置重置为默认值。

## 运行RESET语句 

{% tabs %}
{% tab title="SQL CLI" %}
`RESET`语句可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sqlclient/)执行。

以下示例显示如何`RESET`在SQL CLI中运行语句。

```sql
Flink SQL> RESET table.planner;
[INFO] Session property has been reset.

Flink SQL> RESET;
[INFO] All session properties have been set to their default values.
```
{% endtab %}
{% endtabs %}

## 语法 

```sql
RESET (key)?
```

如果未指定任何键，它将所有属性重置为默认值。否则，将指定的密钥重置为默认值。

