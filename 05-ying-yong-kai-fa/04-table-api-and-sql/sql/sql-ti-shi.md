# SQL提示

SQL提示可以与SQL语句一起使用以更改执行计划。本章说明如何使用提示来强制执行各种方法。

通常，提示可以用于：

* 执行规划器：没有完美的计划器，因此实现提示以使用户更好地控制执行是有意义的。
* 附加元数据（或统计信息）：某些统计信息（例如“扫描的表索引”和“某些shuffle键的倾斜信息”）对于查询而言是动态的，用提示来配置它们会非常方便，因为我们从planner获得的计划元数据通常不那么准确；
* 运算符资源约束：在很多情况下，我们将为执行运算符提供默认的资源配置，即最小并行度或托管内存（消耗资源的UDF）或特殊资源要求（GPU或SSD磁盘）等等，这将非常灵活使用每个查询（而不是Job）的提示来配置资源。

## 动态表选项

动态表选项允许动态指定或覆盖表选项，这与通过SQL DDL或connect API定义的静态表选项不同，可以在每个查询的每个表范围内灵活地指定这些选项。

因此，非常适合用于交互式终端中的即席查询，例如，在SQL-CLI中，只需添加动态选项，就可以指定忽略CSV源的解析错误`/*+ OPTIONS('csv.ignore-parse-errors'='true') */`。

注意:动态表选项默认是禁止使用的，因为它可能会改变查询的语义。需要将config选项`table.dynamic-table-options.enabled`设置为`true`（默认为false），有关如何设置config选项的详细信息，请参见“[配置](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/config.html)”。

### 语法

为了不破坏SQL的兼容性，我们使用Oracle风格的SQL提示语法:

```sql
table_path /*+ OPTIONS(key=val [, key=val]*) */

key:
    stringLiteral
val:
    stringLiteral
```

### 示例：

```sql
CREATE TABLE kafka_table1 (id BIGINT, name STRING, age INT) WITH (...);
CREATE TABLE kafka_table2 (id BIGINT, name STRING, age INT) WITH (...);

-- override table options in query source
select id, name from kafka_table1 /*+ OPTIONS('scan.startup.mode'='earliest-offset') */;

-- override table options in join
select * from
    kafka_table1 /*+ OPTIONS('scan.startup.mode'='earliest-offset') */ t1
    join
    kafka_table2 /*+ OPTIONS('scan.startup.mode'='earliest-offset') */ t2
    on t1.id = t2.id;

-- override table options for INSERT target table
insert into kafka_table1 /*+ OPTIONS('sink.partitioner'='round-robin') */ select * from kafka_table2;
```

