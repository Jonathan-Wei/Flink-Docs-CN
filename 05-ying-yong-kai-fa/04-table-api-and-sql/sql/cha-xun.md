# 查询

使用`TableEnvironment`的`sqlQuery()`方法指定SQL查询。该方法以表的形式返回SQL查询的结果。表可以用于后续的[SQL和表API查询](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#mixing-table-api-and-sql)，可以[转换为`DataSet`或`DataStream`](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#integration-with-datastream-and-dataset-api)，也可以写入[TableSink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#emit-a-table)\)。可以无缝地混合SQL和Table API查询，并对其进行整体优化，并将其转换为单个程序。

为了访问SQL查询中的表，必须在`TableEnvironment`中注册表。可以从[TableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource), [Table](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-table), [DataStream, 或 DataSet](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-datastream-or-dataset-as-table).注册表。另外，用户还可以在`TableEnvironment`中注册外部目录，以指定数据源的位置。

为了方便起见，`Table.toString()`在`TableEnvironment`中以惟一的名称自动注册表并返回该名称。因此，表对象可以直接内联到SQL查询中\(通过字符串连接\)，如下面的示例所示。

{% hint style="info" %}
注意：包含不受支持的SQL功能的查询会导致`TableException`。以下各节列出了批处理表和流表上SQL的受支持功能。
{% endhint %}

## 指定查询

下面的示例展示如何在注册表和内联表上指定SQL查询。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// ingest a DataStream from an external source
DataStream<Tuple3<Long, String, Integer>> ds = env.addSource(...);

// SQL query with an inlined (unregistered) table
Table table = tableEnv.fromDataStream(ds, "user, product, amount");
Table result = tableEnv.sqlQuery(
  "SELECT SUM(amount) FROM " + table + " WHERE product LIKE '%Rubber%'");

// SQL query with a registered table
// register the DataStream as view "Orders"
tableEnv.createTemporaryView("Orders", ds, "user, product, amount");
// run a SQL query on the Table and retrieve the result as a new Table
Table result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// create and register a TableSink
final Schema schema = new Schema()
    .field("product", DataTypes.STRING())
    .field("amount", DataTypes.INT());

tableEnv.connect(new FileSystem("/path/to/file"))
    .withFormat(...)
    .withSchema(schema)
    .createTemporaryTable("RubberOrders");

// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = StreamTableEnvironment.create(env)

// read a DataStream from an external source
val ds: DataStream[(Long, String, Integer)] = env.addSource(...)

// SQL query with an inlined (unregistered) table
val table = ds.toTable(tableEnv, 'user, 'product, 'amount)
val result = tableEnv.sqlQuery(
  s"SELECT SUM(amount) FROM $table WHERE product LIKE '%Rubber%'")

// SQL query with a registered table
// register the DataStream under the name "Orders"
tableEnv.createTemporaryView("Orders", ds, 'user, 'product, 'amount)
// run a SQL query on the Table and retrieve the result as a new Table
val result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")

// SQL update with a registered table
// create and register a TableSink
val schema = new Schema()
    .field("product", DataTypes.STRING())
    .field("amount", DataTypes.INT())

tableEnv.connect(new FileSystem("/path/to/file"))
    .withFormat(...)
    .withSchema(schema)
    .createTemporaryTable("RubberOrders")

// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)

# SQL query with an inlined (unregistered) table
# elements data type: BIGINT, STRING, BIGINT
table = table_env.from_elements(..., ['user', 'product', 'amount'])
result = table_env \
    .sql_query("SELECT SUM(amount) FROM %s WHERE product LIKE '%%Rubber%%'" % table)

# SQL update with a registered table
# create and register a TableSink
t_env.connect(FileSystem().path("/path/to/file")))
    .with_format(Csv()
                 .field_delimiter(',')
                 .deriveSchema())
    .with_schema(Schema()
                 .field("product", DataTypes.STRING())
                 .field("amount", DataTypes.BIGINT()))
    .create_temporary_table("RubberOrders")

# run a SQL update query on the Table and emit the result to the TableSink
table_env \
    .sql_update("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}
{% endtabs %}

## 支持的语法

Flink使用支持标准ANSI SQL的[Apache Calcite](https://calcite.apache.org/docs/reference.html)解析SQL。

以下BNF语法描述了批处理和流式查询中支持的SQL功能的超集。“ [操作符”](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sql.html#operations)部分展示支持的功能示例，并指出哪些特性只支持批处理或流查询。

```sql
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

Flink SQL对类似于Java的标识符（表，属性，函数名称）使用词法策略：

* 不管是否引用标识符，都保留标识符的大小写。
* 之后，标识符区分大小写。
* 与Java不同，反引号允许标识符包含非字母数字字符（例如``"SELECT a AS `my field` FROM t"``）。

字符串文字必须用单引号引起来（例如`SELECT 'Hello World'`）。复制单引号以进行转义（例如`SELECT 'It''s me.'`）。字符串文字中支持Unicode字符。如果需要明确的unicode代码点，请使用以下语法：

* 使用反斜杠（`\`）作为转义字符（默认）：`SELECT U&'\263A'`
* 使用自定义转义字符： `SELECT U&'#263A' UESCAPE '#'`

## 操作符

### Show and Use

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Show</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p><b>&#x663E;&#x793A;&#x6240;&#x6709;&#x76EE;&#x5F55;</b>
        </p>
        <p><b>SHOW</b> CATALOGS;</p>
        <p><b>&#x663E;&#x793A;&#x5F53;&#x524D;&#x76EE;&#x5F55;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x6570;&#x636E;&#x5E93;</b>
        </p>
        <p><b>SHOW</b> DATABASES;</p>
        <p><b>&#x663E;&#x793A;&#x5F53;&#x524D;&#x76EE;&#x5F55;&#x4E2D;&#x5F53;&#x524D;&#x6570;&#x636E;&#x5E93;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x8868;</b>
        </p>
        <p><b>SHOW</b> TABLES;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Use</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p><b>&#x8BBE;&#x7F6E;&#x4F1A;&#x8BDD;&#x7684;&#x5F53;&#x524D;&#x76EE;&#x5F55;</b>
        </p>
        <p>USE <b>CATALOG</b> mycatalog;</p>
        <p><b>&#x8BBE;&#x7F6E;&#x4F1A;&#x8BDD;&#x7684;&#x5F53;&#x524D;&#x76EE;&#x5F55;&#x7684;&#x5F53;&#x524D;&#x6570;&#x636E;&#x5E93;</b>
        </p>
        <p>USE mydatabase;</p>
      </td>
    </tr>
  </tbody>
</table>### 扫描，投影，过滤\(Scan, Projection, and Filter\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x64CD;&#x4F5C;caozuo&#x64CD;&#x4F5C;</th>
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
</table>### 聚合\(Aggregations\)

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
    <tr>
      <td style="text-align:left"><b>Distinct</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Grouping sets, Rollup, Cube</b>
        <br />Batch Streaming Result Updating</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Having</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>SUM</b>(amount)</p>
        <p><b>FROM</b> Orders</p>
        <p><b>GROUP</b>  <b>BY</b> users</p>
        <p><b>HAVING</b>  <b>SUM</b>(amount) <b>&gt;</b> 50</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>User-defined Aggregate Functions (UDAGG)</b>
        <br />Batch Streaming</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>### 关联\(Joins\)

### Set 操作

#### OrderBy & Limit

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
</table>### Top-N

### 重复数据删除

### 组窗口\(Group Windows\)

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

### 模式识别\(Pattern Recognition\)

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
</table>### 模式识别

