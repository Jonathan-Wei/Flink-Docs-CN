# SQL

## 指定查询

下面的示例展示如何在注册表和内联表上指定SQL查询。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// ingest a DataStream from an external source
DataStream<Tuple3<Long, String, Integer>> ds = env.addSource(...);

// SQL query with an inlined (unregistered) table
Table table = tableEnv.fromDataStream(ds, "user, product, amount");
Table result = tableEnv.sqlQuery(
  "SELECT SUM(amount) FROM " + table + " WHERE product LIKE '%Rubber%'");

// SQL query with a registered table
// register the DataStream as table "Orders"
tableEnv.registerDataStream("Orders", ds, "user, product, amount");
// run a SQL query on the Table and retrieve the result as a new Table
Table result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// create and register a TableSink
TableSink csvSink = new CsvTableSink("/path/to/file", ...);
String[] fieldNames = {"product", "amount"};
TypeInformation[] fieldTypes = {Types.STRING, Types.INT};
tableEnv.registerTableSink("RubberOrders", fieldNames, fieldTypes, csvSink);
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// read a DataStream from an external source
val ds: DataStream[(Long, String, Integer)] = env.addSource(...)

// SQL query with an inlined (unregistered) table
val table = ds.toTable(tableEnv, 'user, 'product, 'amount)
val result = tableEnv.sqlQuery(
  s"SELECT SUM(amount) FROM $table WHERE product LIKE '%Rubber%'")

// SQL query with a registered table
// register the DataStream under the name "Orders"
tableEnv.registerDataStream("Orders", ds, 'user, 'product, 'amount)
// run a SQL query on the Table and retrieve the result as a new Table
val result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")

// SQL update with a registered table
// create and register a TableSink
TableSink csvSink = new CsvTableSink("/path/to/file", ...)
val fieldNames: Array[String] = Array("product", "amount")
val fieldTypes: Array[TypeInformation[_]] = Array(Types.STRING, Types.INT)
tableEnv.registerTableSink("RubberOrders", fieldNames, fieldTypes, csvSink)
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}
{% endtabs %}

## 支持语法

Flink使用[Apache Calcite](https://calcite.apache.org/docs/reference.html)解析SQL ，它支持标准的ANSI SQL。Flink不支持DDL语句。

以下BNF语法描述了批处理和流式查询中支持的SQL功能的超集。“ [操作”](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sql.html#operations)部分展示支持的功能示例，并指出哪些特性只支持批处理或流查询。

```sql
insert:
  INSERT INTO tableReference
  query
  
query:
  values
  | {
      select
      | selectWithoutFrom
      | query UNION [ ALL ] query
      | query EXCEPT query
      | query INTERSECT query
    }
    [ ORDER BY orderItem [, orderItem ]* ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start { ROW | ROWS } ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY]

orderItem:
  expression [ ASC | DESC ]

select:
  SELECT [ ALL | DISTINCT ]
  { * | projectItem [, projectItem ]* }
  FROM tableExpression
  [ WHERE booleanExpression ]
  [ GROUP BY { groupItem [, groupItem ]* } ]
  [ HAVING booleanExpression ]
  [ WINDOW windowName AS windowSpec [, windowName AS windowSpec ]* ]
  
selectWithoutFrom:
  SELECT [ ALL | DISTINCT ]
  { * | projectItem [, projectItem ]* }

projectItem:
  expression [ [ AS ] columnAlias ]
  | tableAlias . *

tableExpression:
  tableReference [, tableReference ]*
  | tableExpression [ NATURAL ] [ LEFT | RIGHT | FULL ] JOIN tableExpression [ joinCondition ]

joinCondition:
  ON booleanExpression
  | USING '(' column [, column ]* ')'

tableReference:
  tablePrimary
  [ matchRecognize ]
  [ [ AS ] alias [ '(' columnAlias [, columnAlias ]* ')' ] ]

tablePrimary:
  [ TABLE ] [ [ catalogName . ] schemaName . ] tableName
  | LATERAL TABLE '(' functionName '(' expression [, expression ]* ')' ')'
  | UNNEST '(' expression ')'

values:
  VALUES expression [, expression ]*

groupItem:
  expression
  | '(' ')'
  | '(' expression [, expression ]* ')'
  | CUBE '(' expression [, expression ]* ')'
  | ROLLUP '(' expression [, expression ]* ')'
  | GROUPING SETS '(' groupItem [, groupItem ]* ')'

windowRef:
    windowName
  | windowSpec

windowSpec:
    [ windowName ]
    '('
    [ ORDER BY orderItem [, orderItem ]* ]
    [ PARTITION BY expression [, expression ]* ]
    [
        RANGE numericOrIntervalExpression {PRECEDING}
      | ROWS numericExpression {PRECEDING}
    ]
    ')'

matchRecognize:
      MATCH_RECOGNIZE '('
      [ PARTITION BY expression [, expression ]* ]
      [ ORDER BY orderItem [, orderItem ]* ]
      [ MEASURES measureColumn [, measureColumn ]* ]
      [ ONE ROW PER MATCH ]
      [ AFTER MATCH
            ( SKIP TO NEXT ROW
            | SKIP PAST LAST ROW
            | SKIP TO FIRST variable
            | SKIP TO LAST variable
            | SKIP TO variable )
      ]
      PATTERN '(' pattern ')'
      [ WITHIN intervalLiteral ]
      DEFINE variable AS condition [, variable AS condition ]*
      ')'

measureColumn:
      expression AS alias

pattern:
      patternTerm [ '|' patternTerm ]*

patternTerm:
      patternFactor [ patternFactor ]*

patternFactor:
      variable [ patternQuantifier ]

patternQuantifier:
      '*'
  |   '*?'
  |   '+'
  |   '+?'
  |   '?'
  |   '??'
  |   '{' { [ minRepeat ], [ maxRepeat ] } '}' ['?']
  |   '{' repeat '}'
```

## 操作

### ​扫描，投影，过滤\(Scan, Projection, and Filter\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">操作</th>
      <th style="text-align:left">描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Scan / Select / As</b>
        <br />Batch、Streaming</td>
      <td style="text-align:left">
        <p><code>SELECT * FROM Orders</code>
        </p>
        <p><code>SELECT a, c AS d FROM Orders</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Where / Filter</b>
        <br />Batch、Streaming</td>
      <td style="text-align:left">
        <p><code>SELECT * FROM Orders WHERE b = &apos;red&apos;</code>
        </p>
        <p><code>SELECT * FROM Orders WHERE a % 2 = 0</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>User-defined Scalar Functions (Scalar UDF)</b>
        <br />Batch、Streaming</td>
      <td style="text-align:left">
        <p>UDF必须在TableEnvironment中注册。有关如何指定和注册标量UDF的详细信息，请参阅<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/udfs.html">UDF</a>文档。</p>
        <p><code>SELECT PRETTY_PRINT(user)FROM Orders</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>### 聚合\(Aggregations\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">操作</th>
      <th style="text-align:left">描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>GroupBy Aggregation</b>
        <br />Batch、Streaming、
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>注意：流表上的GroupBy会生成更新结果。有关详细信息，请参阅<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/dynamic_tables.html">Dynamic Tables Streaming Concepts</a>页面。</p>
        <p><code>SELECT a, SUM(b) as d </code>
        </p>
        <p><code>FROM Orders </code>
        </p>
        <p><code>GROUP BY a</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>GroupBy Window Aggregation</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>使用组窗口计算每个组的单个结果行。有关详细信息，请参阅组窗口部分。</p>
        <p><code>SELECT user, SUM(amount) </code>
        </p>
        <p><code>FROM Orders </code>
        </p>
        <p><code>GROUP BY TUMBLE(rowtime, INTERVAL &apos;1&apos; DAY), user</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Over Window aggregation</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p><b>注意</b>：必须在同一窗口中定义所有聚合，即相同的分区，排序和范围。目前，仅支持具有PRREDING（UNBOUNDED和有界）到CURRENT
          ROW范围的窗口。尚不支持使用FOLLOWING的范围。必须在单个时间属性上指定ORDER BY</p>
        <p><code>SELECT COUNT(amount) OVER ( </code>
        </p>
        <p><code>    PARTITION BY user </code>
        </p>
        <p><code>    ORDER BY proctime </code>
        </p>
        <p><code>    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) </code>
        </p>
        <p><code>FROM Orders</code>
        </p>
        <p><code>SELECT COUNT(amount) OVER w, SUM(amount) OVER w </code>
        </p>
        <p><code>FROM Orders WINDOW w AS ( </code>
        </p>
        <p><code>    PARTITION BY user </code>
        </p>
        <p><code>    ORDER BY proctime </code>
        </p>
        <p><code>    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>### 关联\(Joins\)

### 集合操作\(Set Operations\)

### OrderBy & Limit

<table>
  <thead>
    <tr>
      <th style="text-align:left">操作</th>
      <th style="text-align:left">描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Order By</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>注意：流式查询的结果必须主要按升序<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/time_attributes.html">时间属性</a>排序。支持其他排序属性。</p>
        <p><code>SELECT * </code>
        </p>
        <p><code>FROM Orders </code>
        </p>
        <p><code>ORDER BY orderTime</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Limit</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p><code>SELECT * </code>
        </p>
        <p><code>ROM Orders </code>
        </p>
        <p><code>LIMIT 3</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>### 插入\(Insert\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">操作</th>
      <th style="text-align:left">描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Insert Into</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>输出表必须在TableEnvironment中<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#register-a-tablesink">注册</a>（请参阅
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#register-a-tablesink">注册TableSink</a>）。此外，已注册表的模式必须与查询的模式匹配。</p>
        <p><code>INSERT INTO OutputTable</code>
        </p>
        <p><code>SELECT users, tag</code>
        </p>
        <p><code>FROM Orders</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>### 组窗口\(Group Windows\)

| 组窗口函数 | 描述 |
| :--- | :--- |
| TUMBLE\(time\_attr, interval\) |  |
| HOP\(time\_attr, interval, interval\) |  |
| SESSION\(time\_attr, interval\) |  |

#### 时间属性

#### **选择组窗口开始和结束时间戳**

| **辅助函数** | 描述 |
| :--- | :--- |
| `TUMBLE_START(time_attr, interval)` `HOP_START(time_attr, interval, interval)` `SESSION_START(time_attr, interval)` |  |
| `TUMBLE_END(time_attr, interval)` `HOP_END(time_attr, interval, interval)` `SESSION_END(time_attr, interval)` |  |
| `TUMBLE_ROWTIME(time_attr, interval)` `HOP_ROWTIME(time_attr, interval, interval)` `SESSION_ROWTIME(time_attr, interval)` |  |
| `TUMBLE_PROCTIME(time_attr, interval)` `HOP_PROCTIME(time_attr, interval, interval)` `SESSION_PROCTIME(time_attr, interval)` |  |

{% hint style="info" %}
注意:必须使用与group BY子句中的group window函数完全相同的参数调用辅助函数
{% endhint %}

下面的示例展示如何在流表上使用组窗口指定SQL查询。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// ingest a DataStream from an external source
DataStream<Tuple3<Long, String, Integer>> ds = env.addSource(...);
// register the DataStream as table "Orders"
tableEnv.registerDataStream("Orders", ds, "user, product, amount, proctime.proctime, rowtime.rowtime");

// compute SUM(amount) per day (in event-time)
Table result1 = tableEnv.sqlQuery(
  "SELECT user, " +
  "  TUMBLE_START(rowtime, INTERVAL '1' DAY) as wStart,  " +
  "  SUM(amount) FROM Orders " +
  "GROUP BY TUMBLE(rowtime, INTERVAL '1' DAY), user");

// compute SUM(amount) per day (in processing-time)
Table result2 = tableEnv.sqlQuery(
  "SELECT user, SUM(amount) FROM Orders GROUP BY TUMBLE(proctime, INTERVAL '1' DAY), user");

// compute every hour the SUM(amount) of the last 24 hours in event-time
Table result3 = tableEnv.sqlQuery(
  "SELECT product, SUM(amount) FROM Orders GROUP BY HOP(rowtime, INTERVAL '1' HOUR, INTERVAL '1' DAY), product");

// compute SUM(amount) per session with 12 hour inactivity gap (in event-time)
Table result4 = tableEnv.sqlQuery(
  "SELECT user, " +
  "  SESSION_START(rowtime, INTERVAL '12' HOUR) AS sStart, " +
  "  SESSION_ROWTIME(rowtime, INTERVAL '12' HOUR) AS snd, " +
  "  SUM(amount) " +
  "FROM Orders " +
  "GROUP BY SESSION(rowtime, INTERVAL '12' HOUR), user");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// read a DataStream from an external source
val ds: DataStream[(Long, String, Int)] = env.addSource(...)
// register the DataStream under the name "Orders"
tableEnv.registerDataStream("Orders", ds, 'user, 'product, 'amount, 'proctime.proctime, 'rowtime.rowtime)

// compute SUM(amount) per day (in event-time)
val result1 = tableEnv.sqlQuery(
    """
      |SELECT
      |  user,
      |  TUMBLE_START(rowtime, INTERVAL '1' DAY) as wStart,
      |  SUM(amount)
      | FROM Orders
      | GROUP BY TUMBLE(rowtime, INTERVAL '1' DAY), user
    """.stripMargin)

// compute SUM(amount) per day (in processing-time)
val result2 = tableEnv.sqlQuery(
  "SELECT user, SUM(amount) FROM Orders GROUP BY TUMBLE(proctime, INTERVAL '1' DAY), user")

// compute every hour the SUM(amount) of the last 24 hours in event-time
val result3 = tableEnv.sqlQuery(
  "SELECT product, SUM(amount) FROM Orders GROUP BY HOP(rowtime, INTERVAL '1' HOUR, INTERVAL '1' DAY), product")

// compute SUM(amount) per session with 12 hour inactivity gap (in event-time)
val result4 = tableEnv.sqlQuery(
    """
      |SELECT
      |  user,
      |  SESSION_START(rowtime, INTERVAL '12' HOUR) AS sStart,
      |  SESSION_END(rowtime, INTERVAL '12' HOUR) AS sEnd,
      |  SUM(amount)
      | FROM Orders
      | GROUP BY SESSION(rowtime(), INTERVAL '12' HOUR), user
    """.stripMargin)
```
{% endtab %}
{% endtabs %}

### 模式识别\(Pattern Recognition\)

SQL运行时构建于Flink的DataSet和DataStream API之上。在内部，它还使用Flink `TypeInformation`来定义数据类型。完全支持的类型列在`org.apache.flink.table.api.Types`。下表总结了SQL类型，表API类型和生成的Java类之间的关系。

<table>
  <thead>
    <tr>
      <th style="text-align:left">操作</th>
      <th style="text-align:left">描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>MATCH_RECOGNIZE</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p>根据MATCH_RECOGNIZE<a href="https://standards.iso.org/ittf/PubliclyAvailableStandards/c065143_ISO_IEC_TR_19075-5_2016.zip"> ISO标准</a>在流表中搜索给定模式。这使得在SQL查询中表达复杂事件处理（CEP）逻辑成为可能。</p>
        <p>有关更详细的说明，请参阅<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/match_recognize.html">检测表中模式</a>的专用页面。</p>
        <p><code>SELECT T.aid, T.bid, T.cid </code>
        </p>
        <p><code>FROM MyTable </code>
        </p>
        <p><code>MATCH_RECOGNIZE ( </code>
        </p>
        <p><code>  PARTITION BY userid </code>
        </p>
        <p><code>  ORDER BY proctime </code>
        </p>
        <p><code>  MEASURES</code>
        </p>
        <p><code>    A.id AS aid, </code>
        </p>
        <p><code>    B.id AS bid, </code>
        </p>
        <p><code>    C.id AS cid </code>
        </p>
        <p><code>  PATTERN (A B C) </code>
        </p>
        <p><code>  DEFINE </code>
        </p>
        <p><code>    A AS name = &apos;a&apos;, </code>
        </p>
        <p><code>    B AS name = &apos;b&apos;, </code>
        </p>
        <p><code>    C AS name = &apos;c&apos; </code>
        </p>
        <p><code>) AS T</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>## 数据类型

| Table API | SQL | Java类型 |
| :--- | :--- | :--- |
| Types.STRING | VARCHAR | java.lang.String |
| Types.BOOLEAN | BOOLEAN | java.lang.Boolean |
| Types.BYTE | TINYINT | java.lang.Byte |
| Types.SHORT | SMALLINT | java.lang.Short |
| Types.INT | INTEGER, INT | java.lang.Integer |
| Types.LONG | BIGINT | java.lang.Long |
| Types.FLOAT | REAL, FLOAT | java.lang.Float |
| Types.DOUBLE | DOUBLE | java.lang.Double |
| Types.DECIMAL | DECIMAL | java.math.BigDecimal |
| Types.SQL\_DATE | DATE | java.sql.Date |
| Types.SQL\_TIME | TIME | java.sql.Time |
| Types.SQL\_TIMESTAMP | TIMESTAMP\(3\) | java.sql.Timestamp |
| Types.INTERVAL\_MONTHS | INTERVAL YEAR TO MONTH | java.lang.Integer |
| Types.INTERVAL\_MILLIS | INTERVAL DAY TO SECOND\(3\) | java.lang.Long |
| Types.PRIMITIVE\_ARRAY | ARRAY | 例如 `int[]` |
| Types.OBJECT\_ARRAY | ARRAY | 例如 `java.lang.Byte[]` |
| Types.MAP | MAP | java.util.HashMap |
| Types.MULTISET | MULTISET | 例如，`java.util.HashMap<String, Integer>`表示`String`的多重集合 |
| Types.ROW | ROW | org.apache.flink.types.Row |

泛型类型和\(嵌套的\)复合类型\(例如：POJO、Tuple、Row、Scala Case Class\)也可以是行中的字段。

可以使用[值访问函数](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/functions.html#value-access-functions)访问具有任意嵌套的复合类型的字段。

泛型类型被视为黑盒，可以由[用户定义的函数](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/udfs.html)传递或处理。

## 保留字

虽然并非每个SQL功能都已实现，但某些字符串组合已被保留为关键字以供将来使用。如果要将以下字符串之一用作字段名称，请确保使用反引号将其包围（例如```value```，```count```）。

```text
A, ABS, ABSOLUTE, ACTION, ADA, ADD, ADMIN, AFTER, ALL, ALLOCATE, ALLOW, ALTER, ALWAYS, AND, ANY, ARE, ARRAY, AS, ASC, 
ASENSITIVE, ASSERTION, ASSIGNMENT, ASYMMETRIC, AT, ATOMIC, ATTRIBUTE, ATTRIBUTES, AUTHORIZATION, AVG, BEFORE, BEGIN, 
BERNOULLI, BETWEEN, BIGINT, BINARY, BIT, BLOB, BOOLEAN, BOTH, BREADTH, BY, C, CALL, CALLED, CARDINALITY, CASCADE, 
CASCADED, CASE, CAST, CATALOG, CATALOG_NAME, CEIL, CEILING, CENTURY, CHAIN, CHAR, CHARACTER, CHARACTERISTICS, 
CHARACTERS, CHARACTER_LENGTH, CHARACTER_SET_CATALOG, CHARACTER_SET_NAME, CHARACTER_SET_SCHEMA, CHAR_LENGTH, CHECK, 
CLASS_ORIGIN, CLOB, CLOSE, COALESCE, COBOL, COLLATE, COLLATION, COLLATION_CATALOG, COLLATION_NAME, COLLATION_SCHEMA, 
COLLECT, COLUMN, COLUMN_NAME, COMMAND_FUNCTION, COMMAND_FUNCTION_CODE, COMMIT, COMMITTED, CONDITION, CONDITION_NUMBER, 
CONNECT, CONNECTION, CONNECTION_NAME, CONSTRAINT, CONSTRAINTS, CONSTRAINT_CATALOG, CONSTRAINT_NAME, CONSTRAINT_SCHEMA, 
CONSTRUCTOR, CONTAINS, CONTINUE, CONVERT, CORR, CORRESPONDING, COUNT, COVAR_POP, COVAR_SAMP, CREATE, CROSS, CUBE, 
CUME_DIST, CURRENT, CURRENT_CATALOG, CURRENT_DATE, CURRENT_DEFAULT_TRANSFORM_GROUP, CURRENT_PATH, CURRENT_ROLE, 
CURRENT_SCHEMA, CURRENT_TIME, CURRENT_TIMESTAMP, CURRENT_TRANSFORM_GROUP_FOR_TYPE, CURRENT_USER, CURSOR, CURSOR_NAME, 
CYCLE, DATA, DATABASE, DATE, DATETIME_INTERVAL_CODE, DATETIME_INTERVAL_PRECISION, DAY, DEALLOCATE, DEC, DECADE, 
DECIMAL, DECLARE, DEFAULT, DEFAULTS, DEFERRABLE, DEFERRED, DEFINED, DEFINER, DEGREE, DELETE, DENSE_RANK, DEPTH, DEREF, 
DERIVED, DESC, DESCRIBE, DESCRIPTION, DESCRIPTOR, DETERMINISTIC, DIAGNOSTICS, DISALLOW, DISCONNECT, DISPATCH, DISTINCT, 
DOMAIN, DOUBLE, DOW, DOY, DROP, DYNAMIC, DYNAMIC_FUNCTION, DYNAMIC_FUNCTION_CODE, EACH, ELEMENT, ELSE, END, END-EXEC, 
EPOCH, EQUALS, ESCAPE, EVERY, EXCEPT, EXCEPTION, EXCLUDE, EXCLUDING, EXEC, EXECUTE, EXISTS, EXP, EXPLAIN, EXTEND, 
EXTERNAL, EXTRACT, FALSE, FETCH, FILTER, FINAL, FIRST, FIRST_VALUE, FLOAT, FLOOR, FOLLOWING, FOR, FOREIGN, FORTRAN, 
FOUND, FRAC_SECOND, FREE, FROM, FULL, FUNCTION, FUSION, G, GENERAL, GENERATED, GET, GLOBAL, GO, GOTO, GRANT, GRANTED, 
GROUP, GROUPING, HAVING, HIERARCHY, HOLD, HOUR, IDENTITY, IMMEDIATE, IMPLEMENTATION, IMPORT, IN, INCLUDING, INCREMENT, 
INDICATOR, INITIALLY, INNER, INOUT, INPUT, INSENSITIVE, INSERT, INSTANCE, INSTANTIABLE, INT, INTEGER, INTERSECT, 
INTERSECTION, INTERVAL, INTO, INVOKER, IS, ISOLATION, JAVA, JOIN, K, KEY, KEY_MEMBER, KEY_TYPE, LABEL, LANGUAGE, 
LARGE, LAST, LAST_VALUE, LATERAL, LEADING, LEFT, LENGTH, LEVEL, LIBRARY, LIKE, LIMIT, LN, LOCAL, LOCALTIME, 
LOCALTIMESTAMP, LOCATOR, LOWER, M, MAP, MATCH, MATCHED, MAX, MAXVALUE, MEMBER, MERGE, MESSAGE_LENGTH, 
MESSAGE_OCTET_LENGTH, MESSAGE_TEXT, METHOD, MICROSECOND, MILLENNIUM, MIN, MINUTE, MINVALUE, MOD, MODIFIES, MODULE, 
MONTH, MORE, MULTISET, MUMPS, NAME, NAMES, NATIONAL, NATURAL, NCHAR, NCLOB, NESTING, NEW, NEXT, NO, NONE, NORMALIZE, 
NORMALIZED, NOT, NULL, NULLABLE, NULLIF, NULLS, NUMBER, NUMERIC, OBJECT, OCTETS, OCTET_LENGTH, OF, OFFSET, OLD, ON, 
ONLY, OPEN, OPTION, OPTIONS, OR, ORDER, ORDERING, ORDINALITY, OTHERS, OUT, OUTER, OUTPUT, OVER, OVERLAPS, OVERLAY, 
OVERRIDING, PAD, PARAMETER, PARAMETER_MODE, PARAMETER_NAME, PARAMETER_ORDINAL_POSITION, PARAMETER_SPECIFIC_CATALOG, 
PARAMETER_SPECIFIC_NAME, PARAMETER_SPECIFIC_SCHEMA, PARTIAL, PARTITION, PASCAL, PASSTHROUGH, PATH, PERCENTILE_CONT, 
PERCENTILE_DISC, PERCENT_RANK, PLACING, PLAN, PLI, POSITION, POWER, PRECEDING, PRECISION, PREPARE, PRESERVE, PRIMARY, 
PRIOR, PRIVILEGES, PROCEDURE, PUBLIC, QUARTER, RANGE, RANK, READ, READS, REAL, RECURSIVE, REF, REFERENCES, REFERENCING, 
REGR_AVGX, REGR_AVGY, REGR_COUNT, REGR_INTERCEPT, REGR_R2, REGR_SLOPE, REGR_SXX, REGR_SXY, REGR_SYY, RELATIVE, RELEASE, 
REPEATABLE, RESET, RESTART, RESTRICT, RESULT, RETURN, RETURNED_CARDINALITY, RETURNED_LENGTH, RETURNED_OCTET_LENGTH, 
RETURNED_SQLSTATE, RETURNS, REVOKE, RIGHT, ROLE, ROLLBACK, ROLLUP, ROUTINE, ROUTINE_CATALOG, ROUTINE_NAME, ROUTINE_SCHEMA, 
ROW, ROWS, ROW_COUNT, ROW_NUMBER, SAVEPOINT, SCALE, SCHEMA, SCHEMA_NAME, SCOPE, SCOPE_CATALOGS, SCOPE_NAME, SCOPE_SCHEMA, 
SCROLL, SEARCH, SECOND, SECTION, SECURITY, SELECT, SELF, SENSITIVE, SEQUENCE, SERIALIZABLE, SERVER, SERVER_NAME, SESSION, 
SESSION_USER, SET, SETS, SIMILAR, SIMPLE, SIZE, SMALLINT, SOME, SOURCE, SPACE, SPECIFIC, SPECIFICTYPE, SPECIFIC_NAME, SQL, 
SQLEXCEPTION, SQLSTATE, SQLWARNING, SQL_TSI_DAY, SQL_TSI_FRAC_SECOND, SQL_TSI_HOUR, SQL_TSI_MICROSECOND, SQL_TSI_MINUTE, 
SQL_TSI_MONTH, SQL_TSI_QUARTER, SQL_TSI_SECOND, SQL_TSI_WEEK, SQL_TSI_YEAR, SQRT, START, STATE, STATEMENT, STATIC, STDDEV_POP, 
STDDEV_SAMP, STREAM, STRUCTURE, STYLE, SUBCLASS_ORIGIN, SUBMULTISET, SUBSTITUTE, SUBSTRING, SUM, SYMMETRIC, SYSTEM, SYSTEM_USER, 
TABLE, TABLESAMPLE, TABLE_NAME, TEMPORARY, THEN, TIES, TIME, TIMESTAMP, TIMESTAMPADD, TIMESTAMPDIFF, TIMEZONE_HOUR, TIMEZONE_MINUTE, 
TINYINT, TO, TOP_LEVEL_COUNT, TRAILING, TRANSACTION, TRANSACTIONS_ACTIVE, TRANSACTIONS_COMMITTED, TRANSACTIONS_ROLLED_BACK, 
TRANSFORM, TRANSFORMS, TRANSLATE, TRANSLATION, TREAT, TRIGGER, TRIGGER_CATALOG, TRIGGER_NAME, TRIGGER_SCHEMA, TRIM, TRUE, 
TYPE, UESCAPE, UNBOUNDED, UNCOMMITTED, UNDER, UNION, UNIQUE, UNKNOWN, UNNAMED, UNNEST, UPDATE, UPPER, UPSERT, USAGE, USER, 
USER_DEFINED_TYPE_CATALOG, USER_DEFINED_TYPE_CODE, USER_DEFINED_TYPE_NAME, USER_DEFINED_TYPE_SCHEMA, USING, VALUE, VALUES, 
VARBINARY, VARCHAR, VARYING, VAR_POP, VAR_SAMP, VERSION, VIEW, WEEK, WHEN, WHENEVER, WHERE, WIDTH_BUCKET, WINDOW, WITH, 
WITHIN, WITHOUT, WORK, WRAPPER, WRITE, XML, YEAR, ZONE
```

