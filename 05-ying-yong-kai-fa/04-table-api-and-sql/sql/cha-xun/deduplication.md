# Deduplication

重复数据删除将删除在一组列上重复的行，仅保留第一个或最后一个。在某些情况下，上游ETL作业不是端到端精确的一次。如果发生故障转移，这可能会导致接收器中的记录重复。然而，重复的记录会影响到下游的分析工作的正确性-例如`SUM`，`COUNT`-所以前进一步的分析需要重复数据删除。

Flink用于`ROW_NUMBER()`删除重复项，就像Top-N查询一样。从理论上讲，重复数据删除是Top-N的一种特殊情况，其中N为1，并按处理时间或事件时间排序。

下面显示了重复数据删除语句的语法：

```sql
SELECT [column_list]
FROM (
   SELECT [column_list],
     ROW_NUMBER() OVER ([PARTITION BY col1[, col2...]]
       ORDER BY time_attr [asc|desc]) AS rownum
   FROM table_name)
WHERE rownum = 1
```

**参数格式：**

* `ROW_NUMBER()`：从第一行开始，为每行分配一个唯一的顺序号。
* `PARTITION BY col1[, col2...]`：指定分区列，即重复数据删除键。
* `ORDER BY time_attr [asc|desc]`：指定排序列，它必须是[time属性](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/)。当前Flink支持[处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/#processing-time)和[事件时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/#event-time)。按ASC排序意味着保留第一行，按DESC排序意味着保留最后一行。
* `WHERE rownum = 1`：`rownum = 1`Flink识别此查询为重复数据删除所必需。

{% hint style="info" %}
注意：必须严格遵循上述模式，否则优化器将无法翻译查询。
{% endhint %}

以下示例显示如何在流表上使用重复数据删除指定SQL查询。

```sql
CREATE TABLE Orders (
  order_time  STRING,
  user        STRING,
  product     STRING,
  num         BIGINT,
  proctime AS PROCTIME()
) WITH (...);

-- remove duplicate rows on order_id and keep the first occurrence row,
-- because there shouldn't be two orders with the same order_id.
SELECT order_id, user, product, num
FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY proctime ASC) AS row_num
  FROM Orders)
WHERE row_num = 1
```

