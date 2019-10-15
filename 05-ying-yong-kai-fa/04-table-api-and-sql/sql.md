# SQL

这是Flink支持的数据定义语言（DDL）和数据操作语言（DML）结构的完整列表。

## Query

使用`TableEnvironment`的`sqlQuery()`方法指定SQL查询。该方法以表的形式返回SQL查询的结果。表可以用于后续的[SQL和表API查询](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#mixing-table-api-and-sql)，可以[转换为`DataSet`或`DataStream`](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#integration-with-datastream-and-dataset-api)，也可以写入[TableSink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#emit-a-table)\)。可以无缝地混合SQL和Table API查询，并对其进行整体优化，并将其转换为单个程序。

为了访问SQL查询中的表，必须在`TableEnvironment`中注册表。可以从[TableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource), [Table](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-table), [DataStream, 或 DataSet](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-datastream-or-dataset-as-table).注册表。另外，用户还可以在`TableEnvironment`中注册外部目录，以指定数据源的位置。

为了方便起见，`Table.toString()`在`TableEnvironment`中以惟一的名称自动注册表并返回该名称。因此，表对象可以直接内联到SQL查询中\(通过字符串连接\)，如下面的示例所示。

**注意**:Flink的SQL支持还不够完善。包含不受支持的SQL特性的查询会导致`TableException`。下面几节列出了批处理表和流表上的SQL支持的特性。

### 指定查询

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

### 支持语法

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

Flink SQL对标识符\(表、属性、函数名\)使用类似于Java的词汇策略:

无论是否引用标识符，都会保留标识符的大小写。

* 之后，标识符区分大小写。
* 与Java不同，反向标记允许标识符包含非字母数字字符（例如``"SELECT a AS `my field` FROM t"``）。

字符串文字必须用单引号括起来\(例如，选择“Hello World”\)。复制一个转义单引号\(例如，选择'It' s me.'\)。字符串文本支持Unicode字符。如果需要显式unicode代码点，请使用以下语法:

* 使用反斜杠\(\\)作为转义字符\(默认\):`SELECT U&'\263A'`
* 使用自定义转义字符:`SELECT U&'#263A' UESCAPE '#'`

### 操作

#### ​扫描，投影，过滤\(Scan, Projection, and Filter\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Scan / Select / As</b>
        <br />Batch&#x3001;Streaming</td>
      <td style="text-align:left">
        <p><code>SELECT * FROM Orders</code>
        </p>
        <p><code>SELECT a, c AS d FROM Orders</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Where / Filter</b>
        <br />Batch&#x3001;Streaming</td>
      <td style="text-align:left">
        <p><code>SELECT * FROM Orders WHERE b = &apos;red&apos;</code>
        </p>
        <p><code>SELECT * FROM Orders WHERE a % 2 = 0</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>User-defined Scalar Functions (Scalar UDF)</b>
        <br />Batch&#x3001;Streaming</td>
      <td style="text-align:left">
        <p>UDF&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;&#x6CE8;&#x518C;&#x3002;&#x6709;&#x5173;&#x5982;&#x4F55;&#x6307;&#x5B9A;&#x548C;&#x6CE8;&#x518C;&#x6807;&#x91CF;UDF&#x7684;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/udfs.html">UDF</a>&#x6587;&#x6863;&#x3002;</p>
        <p><code>SELECT PRETTY_PRINT(user)FROM Orders</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>#### 聚合\(Aggregations\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>GroupBy Aggregation</b>
        <br />Batch&#x3001;Streaming&#x3001;
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x6D41;&#x8868;&#x4E0A;&#x7684;GroupBy&#x4F1A;&#x751F;&#x6210;&#x66F4;&#x65B0;&#x7ED3;&#x679C;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/dynamic_tables.html">Dynamic Tables Streaming Concepts</a>&#x9875;&#x9762;&#x3002;</p>
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
        <p>&#x4F7F;&#x7528;&#x7EC4;&#x7A97;&#x53E3;&#x8BA1;&#x7B97;&#x6BCF;&#x4E2A;&#x7EC4;&#x7684;&#x5355;&#x4E2A;&#x7ED3;&#x679C;&#x884C;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;&#x7EC4;&#x7A97;&#x53E3;&#x90E8;&#x5206;&#x3002;</p>
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
        <p><b>&#x6CE8;&#x610F;</b>&#xFF1A;&#x5FC5;&#x987B;&#x5728;&#x540C;&#x4E00;&#x7A97;&#x53E3;&#x4E2D;&#x5B9A;&#x4E49;&#x6240;&#x6709;&#x805A;&#x5408;&#xFF0C;&#x5373;&#x76F8;&#x540C;&#x7684;&#x5206;&#x533A;&#xFF0C;&#x6392;&#x5E8F;&#x548C;&#x8303;&#x56F4;&#x3002;&#x76EE;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x5177;&#x6709;PRREDING&#xFF08;UNBOUNDED&#x548C;&#x6709;&#x754C;&#xFF09;&#x5230;CURRENT
          ROW&#x8303;&#x56F4;&#x7684;&#x7A97;&#x53E3;&#x3002;&#x5C1A;&#x4E0D;&#x652F;&#x6301;&#x4F7F;&#x7528;FOLLOWING&#x7684;&#x8303;&#x56F4;&#x3002;&#x5FC5;&#x987B;&#x5728;&#x5355;&#x4E2A;&#x65F6;&#x95F4;&#x5C5E;&#x6027;&#x4E0A;&#x6307;&#x5B9A;ORDER
          BY</p>
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
</table>#### 关联\(Joins\)

| 操作 | 描述 |
| :--- | :--- |
| **Inner Equi-join** |  |
| **Outer Equi-join** |  |
| 时间窗口关联 |  |
| **将数组扩展为**关联 |  |
| 表函数关联 |  |
| 时态表关联 |  |

#### 集合操作\(Set Operations\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> (</p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> a <b>%</b> 2 <b>=</b> 0)</p>
        <p> <b>UNION</b>
        </p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> b <b>=</b> 0)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>UnionAll</b>
          <br />Batch</p>
        <p>Streaming</p>
      </td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> (</p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> a <b>%</b> 2 <b>=</b> 0)</p>
        <p> <b>UNION</b>  <b>ALL</b>
        </p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> b <b>=</b> 0)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>Intersect / Except</b>
        </p>
        <p>Batch</p>
      </td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> (</p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> a <b>%</b> 2 <b>=</b> 0)</p>
        <p> <b>INTERSECT</b>
        </p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> b <b>=</b> 0)</p>
        <p>)</p>
        <p></p>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> (</p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> a <b>%</b> 2 <b>=</b> 0)</p>
        <p> <b>EXCEPT</b>
        </p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> b <b>=</b> 0)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>In</b>
        </p>
        <p>Batch</p>
        <p>Streaming</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>Exists</b>
        </p>
        <p>Batch</p>
        <p>Streaming</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>#### OrderBy & Limit

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Order By</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#x7684;&#x7ED3;&#x679C;&#x5FC5;&#x987B;&#x4E3B;&#x8981;&#x6309;&#x5347;&#x5E8F;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/time_attributes.html">&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#x6392;&#x5E8F;&#x3002;&#x652F;&#x6301;&#x5176;&#x4ED6;&#x6392;&#x5E8F;&#x5C5E;&#x6027;&#x3002;</p>
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
</table>#### 插入\(Insert\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Insert Into</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x8F93;&#x51FA;&#x8868;&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;</a>&#xFF08;&#x8BF7;&#x53C2;&#x9605;<a href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;TableSink</a>&#xFF09;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x6A21;&#x5F0F;&#x5FC5;&#x987B;&#x4E0E;&#x67E5;&#x8BE2;&#x7684;&#x6A21;&#x5F0F;&#x5339;&#x914D;&#x3002;</p>
        <p><code>INSERT INTO OutputTable</code>
        </p>
        <p><code>SELECT users, tag</code>
        </p>
        <p><code>FROM Orders</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>#### 组窗口\(Group Windows\)

组窗口是在SQL查询的`Group BY`子句中定义的。就像使用常规的`GROUP BY`子句查询一样，使用包含`GROUP`窗口函数的`GROUP BY`子句查询可以计算每个组的单个结果行。批处理表和流表上的SQL支持以下组`windows`函数。

| 组窗口函数 | 描述 |
| :--- | :--- |
| TUMBLE\(time\_attr, interval\) | 定义滚动时间窗口。滚动时间窗口将行分配给具有固定持续时间（`interval`）的非重叠连续窗口。例如，5分钟的滚动窗口以5分钟为间隔对行进行分组。可以在事件时间（流+批处理）或处理时间（流）上定义滚动窗口。 |
| HOP\(time\_attr, interval, interval\) | 定义一个跳跃时间窗口\(在表API中称为滑动窗口\)。一个跳转时间窗口有一个固定的持续时间\(第二个间隔参数\)和一个指定的跳转时间间隔\(第一个间隔参数\)。如果跳转间隔小于窗口大小，则跳转窗口是重叠的。因此，可以将行分配给多个窗口。例如，一个15分钟大小和5分钟跳跃间隔的跳跃窗口将每一行分配给3个15分钟大小的不同窗口，这些窗口在5分钟的间隔内计算。可以在事件时间\(流+批处理\)或处理时间\(流\)上定义跳转窗口。 |
| SESSION\(time\_attr, interval\) | 定义会话时间窗口。会话时间窗口没有固定的持续时间，但它们的界限由`interval`不活动时间定义，即如果在定义的间隙期间没有出现事件，则会话窗口关闭。例如，在30分钟不活动后观察到一行时，会有一个30分钟间隙的会话窗口（否则该行将被添加到现有窗口中），如果在30分钟内未添加任何行，则会关闭。会话窗口可以在事件时间（流+批处理）或处理时间（流）上工作。 |

#### 时间属性

对于流表上的SQL查询，组窗口函数的`time_attr`参数必须引用一个有效的time属性，该属性指定行的处理时间或事件时间。参见时间属性文档了解如何定义时间属性。

对于批处理表上的SQL，组窗口函数的`time_attr`参数必须是`TIMESTAMP`类型的属性。

#### **选择组窗口开始和结束时间戳**

组窗口的开始和结束时间戳以及时间属性可以通过以下辅助函数选择:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>&#x8F85;&#x52A9;&#x51FD;&#x6570;</b>
      </th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>TUMBLE_START(time_attr, interval)</code>
        <br /><code>HOP_START(time_attr, interval, interval)</code>
        <br /><code>SESSION_START(time_attr, interval)</code>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x76F8;&#x5E94;&#x7684;&#x6EDA;&#x52A8;&#x3001;&#x8DF3;&#x8F6C;&#x6216;&#x4F1A;&#x8BDD;&#x7A97;&#x53E3;&#x7684;&#x5305;&#x542B;&#x4E0B;&#x9650;&#x7684;&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>TUMBLE_END(time_attr, interval)</code>
        <br /><code>HOP_END(time_attr, interval, interval)</code>
        <br /><code>SESSION_END(time_attr, interval)</code>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x76F8;&#x5E94;&#x7684;&#x6EDA;&#x52A8;&#xFF0C;&#x8DF3;&#x8DC3;&#x6216;&#x4F1A;&#x8BDD;&#x7A97;&#x53E3;&#x7684;&#x72EC;&#x5360;&#x4E0A;&#x9650;&#x7684;&#x65F6;&#x95F4;&#x6233;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x72EC;&#x6709;&#x7684;&#x4E0A;&#x9650;&#x65F6;&#x95F4;&#x6233;&#x4E0D;&#x80FD;&#x5728;&#x540E;&#x7EED;&#x57FA;&#x4E8E;&#x65F6;&#x95F4;&#x7684;&#x64CD;&#x4F5C;&#x4E2D;&#x7528;&#x4F5C;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html">&#x884C;&#x65F6;&#x5C5E;&#x6027;</a>&#xFF0C;&#x4F8B;&#x5982;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#joins">&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x8FDE;&#x63A5;</a>&#x548C;
            <a
            href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#aggregations">&#x7EC4;&#x7A97;&#x53E3;&#x6216;&#x7A97;&#x53E3;&#x805A;&#x5408;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>TUMBLE_ROWTIME(time_attr, interval)</code>
        <br /><code>HOP_ROWTIME(time_attr, interval, interval)</code>
        <br /><code>SESSION_ROWTIME(time_attr, interval)</code>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x76F8;&#x5E94;&#x7684;&#x6EDA;&#x52A8;&#xFF0C;&#x8DF3;&#x8DC3;&#x6216;&#x4F1A;&#x8BDD;&#x7A97;&#x53E3;&#x7684;&#x5305;&#x542B;&#x4E0A;&#x9650;&#x7684;&#x65F6;&#x95F4;&#x6233;&#x3002;</p>
        <p>&#x7ED3;&#x679C;&#x5C5E;&#x6027;&#x662F;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html">rowtime&#x5C5E;&#x6027;</a>&#xFF0C;&#x53EF;&#x4EE5;&#x5728;&#x540E;&#x7EED;&#x57FA;&#x4E8E;&#x65F6;&#x95F4;&#x7684;&#x64CD;&#x4F5C;&#x4E2D;&#x4F7F;&#x7528;&#xFF0C;&#x4F8B;&#x5982;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#joins">&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x8FDE;&#x63A5;</a>&#x548C;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#aggregations">&#x7EC4;&#x7A97;&#x53E3;&#x6216;&#x7A97;&#x53E3;&#x805A;&#x5408;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>TUMBLE_PROCTIME(time_attr, interval)</code>
        <br /><code>HOP_PROCTIME(time_attr, interval, interval)</code>
        <br /><code>SESSION_PROCTIME(time_attr, interval)</code>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html#processing-time">proctime&#x5C5E;&#x6027;</a>&#xFF0C;&#x8BE5;
        <a
        href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html#processing-time">&#x5C5E;&#x6027;</a>&#x53EF;&#x7528;&#x4E8E;&#x540E;&#x7EED;&#x57FA;&#x4E8E;&#x65F6;&#x95F4;&#x7684;&#x64CD;&#x4F5C;&#xFF0C;&#x4F8B;&#x5982;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#joins">&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x8FDE;&#x63A5;</a>&#x548C;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#aggregations">&#x7EC4;&#x7A97;&#x53E3;&#x6216;&#x7A97;&#x53E3;&#x805A;&#x5408;</a>&#x3002;</td>
    </tr>
  </tbody>
</table>{% hint style="info" %}
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

#### 模式识别\(Pattern Recognition\)

SQL运行时构建于Flink的DataSet和DataStream API之上。在内部，它还使用Flink `TypeInformation`来定义数据类型。完全支持的类型列在`org.apache.flink.table.api.Types`。下表总结了SQL类型，表API类型和生成的Java类之间的关系。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>MATCH_RECOGNIZE</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p>&#x6839;&#x636E;MATCH_RECOGNIZE<a href="https://standards.iso.org/ittf/PubliclyAvailableStandards/c065143_ISO_IEC_TR_19075-5_2016.zip"> ISO&#x6807;&#x51C6;</a>&#x5728;&#x6D41;&#x8868;&#x4E2D;&#x641C;&#x7D22;&#x7ED9;&#x5B9A;&#x6A21;&#x5F0F;&#x3002;&#x8FD9;&#x4F7F;&#x5F97;&#x5728;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x8868;&#x8FBE;&#x590D;&#x6742;&#x4E8B;&#x4EF6;&#x5904;&#x7406;&#xFF08;CEP&#xFF09;&#x903B;&#x8F91;&#x6210;&#x4E3A;&#x53EF;&#x80FD;&#x3002;</p>
        <p>&#x6709;&#x5173;&#x66F4;&#x8BE6;&#x7EC6;&#x7684;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/match_recognize.html">&#x68C0;&#x6D4B;&#x8868;&#x4E2D;&#x6A21;&#x5F0F;</a>&#x7684;&#x4E13;&#x7528;&#x9875;&#x9762;&#x3002;</p>
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
</table>## DDL



{% hint style="info" %}
**注意：** Flink的DDL支持尚未完成。包含不受支持的SQL功能的查询会导致`TableException`。以下各节列出了批处理和流表上SQL DDL支持的功能。
{% endhint %}

### 指定DDL

以下示例显示如何指定SQL DDL。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
Table result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH (...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = StreamTableEnvironment.create(env)

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
val result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH ('connector.path'='/path/to/file' ...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)

# SQL update with a registered table
# register a TableSink
table_env.sql_update("CREATE TABLE RubberOrders(product VARCHAR, amount INT) with (...)")
# run a SQL update query on the Table and emit the result to the TableSink
table_env \
    .sql_update("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}
{% endtabs %}

**PARTITIONED BY**

按指定的列对创建的表进行分区。如果此表用作文件系统接收器，则为每个分区创建一个目录。

**WITH OPTIONS**

用于创建表源/接收器的表属性。这些属性通常用于查找和创建基础连接器。

### 创建表

```sql
CREATE TABLE [catalog_name.][db_name.]table_name
  [(col_name1 col_type1 [COMMENT col_comment1], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name1, col_name2, ...)]
  WITH (key1=val1, key2=val2, ...)
```

### 删除表

```sql
DROP TABLE [IF EXISTS] [catalog_name.][db_name.]table_name
```

删除具有给定表名的表。如果要删除的表不存在，则抛出异常。

**如果存在**

如果表不存在，则不会发生任何事情。

## 数据类型

SQL运行时构建在Flink的DataSet和DataStream API之上。在内部，它还使用Flink的类型信息来定义数据类型。所有支持的类型列在`org.apache.flink.table.api.Types`中。下表总结了SQL类型、表API类型和生成的Java类之间的关系。

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

