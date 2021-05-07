# SET语句

 `SET` 语句用于修改配置或列出配置。

## 运行SET语句

{% tabs %}
{% tab title="SQL CLI" %}
`SET`语句可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sqlclient/)执行。

以下示例显示如何`SET`在SQL CLI中运行语句。

```sql
Flink SQL> SET table.planner = blink;
[INFO] Session property has been set.

Flink SQL> SET;
table.planner=blink;
```
{% endtab %}
{% endtabs %}

## 语法 

```sql
SET (key = value)?
```

如果未指定键和值，则仅打印所有属性。否则，将密钥设置为指定值。

