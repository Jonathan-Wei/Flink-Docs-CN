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
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>DISTINCT</b> users <b>FROM</b> Orders</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Grouping sets, Rollup, Cube</b>
        <br />Batch Streaming Result Updating</td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>SUM</b>(amount)</p>
        <p><b>FROM</b> Orders</p>
        <p><b>GROUP</b>  <b>BY</b>  <b>GROUPING</b>  <b>SETS</b> ((<b>user</b>), (product))</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;&#x4EC5;&#x5728;Blink Planner&#x4E2D;&#x652F;&#x6301;Grouping sets, Rollup, Cube&#x3002;</b>
        </p>
      </td>
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
      <td style="text-align:left">
        <p>UDAGG&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;&#x6CE8;&#x518C;&#x3002;&#x6709;&#x5173;&#x5982;&#x4F55;&#x6307;&#x5B9A;&#x548C;&#x6CE8;&#x518C;UDAGG&#x7684;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/udfs.html">UDF&#x6587;&#x6863;</a>&#x3002;</p>
        <p><b>SELECT</b> MyAggregate(amount)</p>
        <p><b>FROM</b> Orders</p>
        <p><b>GROUP</b>  <b>BY</b> users</p>
      </td>
    </tr>
  </tbody>
</table>### 关联\(Joins\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Inner Equi-join</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>
          <br />&#x5F53;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x7B49;&#x8054;&#x63A5;&#xFF0C;&#x5373;&#x5177;&#x6709;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x5177;&#x6709;&#x76F8;&#x7B49;&#x8C13;&#x8BCD;&#x7684;&#x8054;&#x5408;&#x6761;&#x4EF6;&#x7684;&#x8054;&#x63A5;&#x3002;
          &#x4E0D;&#x652F;&#x6301;&#x4EFB;&#x610F;&#x4EA4;&#x53C9;&#x6216;theta&#x8054;&#x63A5;&#x3002;</p>
        <p></p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x8054;&#x63A5;&#x987A;&#x5E8F;&#x672A;&#x4F18;&#x5316;&#x3002;&#x8868;&#x6309;&#x7167;&#x5728;FROM&#x5B50;&#x53E5;&#x4E2D;&#x6307;&#x5B9A;&#x7684;&#x987A;&#x5E8F;&#x8FDB;&#x884C;&#x8FDE;&#x63A5;&#x3002;&#x786E;&#x4FDD;&#x4EE5;&#x4E0D;&#x4EA7;&#x751F;&#x4EA4;&#x53C9;&#x8054;&#x63A5;&#xFF08;&#x7B1B;&#x5361;&#x5C14;&#x4E58;&#x79EF;&#xFF09;&#x7684;&#x987A;&#x5E8F;&#x6307;&#x5B9A;&#x8868;&#xFF0C;&#x8BE5;&#x987A;&#x5E8F;&#x4E0D;&#x88AB;&#x652F;&#x6301;&#x5E76;&#x4E14;&#x4F1A;&#x5BFC;&#x81F4;&#x67E5;&#x8BE2;&#x5931;&#x8D25;&#x3002;</p>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> Orders <b>INNER</b>  <b>JOIN</b> Product <b>ON</b> Orders.productId <b>=</b> Product.id</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Outer Equi-join</b>
        <br />Batch Streaming Result Updating</td>
      <td style="text-align:left">
        <p>&#x5F53;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x7B49;&#x8054;&#x63A5;&#xFF0C;&#x5373;&#x5177;&#x6709;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x5177;&#x6709;&#x76F8;&#x7B49;&#x8C13;&#x8BCD;&#x7684;&#x8054;&#x5408;&#x6761;&#x4EF6;&#x7684;&#x8054;&#x63A5;&#x3002;
          &#x4E0D;&#x652F;&#x6301;&#x4EFB;&#x610F;&#x4EA4;&#x53C9;&#x6216;theta&#x8054;&#x63A5;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x8054;&#x63A5;&#x987A;&#x5E8F;&#x672A;&#x4F18;&#x5316;&#x3002;&#x8868;&#x6309;&#x7167;&#x5728;FROM&#x5B50;&#x53E5;&#x4E2D;&#x6307;&#x5B9A;&#x7684;&#x987A;&#x5E8F;&#x8FDB;&#x884C;&#x8FDE;&#x63A5;&#x3002;&#x786E;&#x4FDD;&#x4EE5;&#x4E0D;&#x4EA7;&#x751F;&#x4EA4;&#x53C9;&#x8054;&#x63A5;&#xFF08;&#x7B1B;&#x5361;&#x5C14;&#x4E58;&#x79EF;&#xFF09;&#x7684;&#x987A;&#x5E8F;&#x6307;&#x5B9A;&#x8868;&#xFF0C;&#x8BE5;&#x987A;&#x5E8F;&#x4E0D;&#x88AB;&#x652F;&#x6301;&#x5E76;&#x4E14;&#x4F1A;&#x5BFC;&#x81F4;&#x67E5;&#x8BE2;&#x5931;&#x8D25;&#x3002;</p>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> Orders <b>LEFT</b>  <b>JOIN</b> Product <b>ON</b> Orders.productId <b>=</b> Product.id</p>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> Orders <b>RIGHT</b>  <b>JOIN</b> Product <b>ON</b> Orders.productId <b>=</b> Product.id</p>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> Orders <b>FULL</b>  <b>OUTER</b>  <b>JOIN</b> Product <b>ON</b> Orders.productId <b>=</b> Product.id</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x65F6;&#x95F4;&#x7A97;&#x53E3; &#x5173;&#x8054;</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x662F;&#x53EF;&#x4EE5;&#x4EE5;&#x6D41;&#x65B9;&#x5F0F;&#x5904;&#x7406;&#x7684;&#x5E38;&#x89C4;&#x5173;&#x8054;&#x7684;&#x5B50;&#x96C6;</p>
        <p>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x9700;&#x8981;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x7B49;&#x503C;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x548C;&#x5728;&#x4E24;&#x4FA7;&#x9650;&#x5236;&#x65F6;&#x95F4;&#x7684;&#x5173;&#x8054;&#x6761;&#x4EF6;&#x3002;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;&#x6BD4;&#x8F83;&#x4E24;&#x4E2A;&#x8F93;&#x5165;&#x8868;&#x4E2D;&#x76F8;&#x540C;&#x7C7B;&#x578B;&#x7684;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html">&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#xFF08;&#x5373;&#x5904;&#x7406;&#x65F6;&#x95F4;&#x6216;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#xFF09;&#x7684;&#x4E24;&#x4E2A;&#x9002;&#x5F53;&#x7684;&#x8303;&#x56F4;&#x8C13;&#x8BCD;&#xFF08;&lt;,
            &lt;=, &gt;=, &gt;&#xFF09;BETWEEN&#x8C13;&#x8BCD;&#x6216;&#x5355;&#x4E2A;&#x76F8;&#x7B49;&#x8C13;&#x8BCD;&#x6765;&#x5B9A;&#x4E49;&#x8FD9;&#x79CD;&#x6761;&#x4EF6;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x4EE5;&#x4E0B;&#x8C13;&#x8BCD;&#x662F;&#x6709;&#x6548;&#x7684;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x6761;&#x4EF6;&#xFF1A;</p>
        <ul>
          <li>ltime = rtime</li>
          <li>ltime &gt;= rtime AND ltime &lt; rtime + INTERVAL &apos;10&apos; MINUTE</li>
          <li>ltime BETWEEN rtime - INTERVAL &apos;10&apos; SECOND AND rtime + INTERVAL
            &apos;5&apos; SECOND</li>
        </ul>
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> Orders o, Shipments s</p>
        <p><b>WHERE</b> o.id <b>=</b> s.orderId <b>AND</b>
        </p>
        <p>o.ordertime <b>BETWEEN</b> s.shiptime <b>-</b> INTERVAL &apos;4&apos; HOUR <b>AND</b> s.shiptime</p>
        <p>&#x5982;&#x679C;&#x8BA2;&#x5355;&#x5728;&#x6536;&#x5230;&#x8BA2;&#x5355;&#x540E;&#x56DB;&#x4E2A;&#x5C0F;&#x65F6;&#x5185;&#x53D1;&#x8D27;&#xFF0C;&#x5219;&#x4E0A;&#x9762;&#x7684;&#x793A;&#x4F8B;&#x4F1A;&#x5C06;&#x6240;&#x6709;&#x8BA2;&#x5355;&#x4E0E;&#x5176;&#x76F8;&#x5E94;&#x7684;&#x53D1;&#x8D27;&#x5408;&#x5E76;&#x5728;&#x4E00;&#x8D77;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>&#x8BB2;</b>
        </p>
        <p><b>&#x5C06;&#x6570;&#x7EC4;&#x62D3;&#x5C55;&#x4E3A;&#x5173;&#x7CFB;</b>
          <br
          />Batch Straming</p>
      </td>
      <td style="text-align:left">
        <p><b>&#x76EE;&#x524D;&#x5C1A;&#x4E0D;&#x652F;&#x6301;&#x4F7F;&#x7528;ORDINALITY&#x53D6;&#x6D88;&#x5D4C;&#x5957;&#x3002;</b>
        </p>
        <p><b>SELECT</b> users, tag</p>
        <p><b>FROM</b> Orders <b>CROSS</b>  <b>JOIN</b>  <b>UNNEST</b>(tags) <b>AS</b> t
          (tag)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x5173;&#x8054;&#x8868;&#x51FD;&#x6570; (UDTF)</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x5173;&#x8054;&#x8868;&#x3002;&#x5DE6;(&#x5916;)&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x90FD;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x8FDE;&#x63A5;&#x5728;&#x4E00;&#x8D77;&#x3002;</p>
        <p>&#x5FC5;&#x987B;&#x5148;&#x6CE8;&#x518C;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x8868;&#x51FD;&#x6570;&#xFF08;UDTF&#xFF09;&#x3002;&#x6709;&#x5173;&#x5982;&#x4F55;&#x6307;&#x5B9A;&#x548C;&#x6CE8;&#x518C;UDTF&#x7684;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/udfs.html">UDF&#x6587;&#x6863;</a>&#x3002;</p>
        <p><b>&#x5185;&#x90E8;&#x8054;&#x63A5;</b>
        </p>
        <p>&#x5982;&#x679C;&#x5DE6;&#x8868;&#xFF08;&#x5916;&#x90E8;&#xFF09;&#x7684;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x8BE5;&#x884C;&#x5C06;&#x88AB;&#x5220;&#x9664;&#x3002;</p>
        <p><b>SELECT</b> users, tag</p>
        <p><b>FROM</b> Orders, <b>LATERAL</b>  <b>TABLE</b>(unnest_udtf(tags)) t <b>AS</b> tag</p>
        <p><b>&#x5DE6;&#x5916;&#x8FDE;&#x63A5;</b>
        </p>
        <p>&#x5982;&#x679C;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x5C06;&#x4FDD;&#x7559;&#x5BF9;&#x5E94;&#x7684;&#x5916;&#x90E8;&#x884C;&#xFF0C;&#x5E76;&#x7528;&#x7A7A;&#x503C;&#x586B;&#x5145;&#x7ED3;&#x679C;&#x3002;</p>
        <p><b>SELECT</b> users, tag</p>
        <p><b>FROM</b> Orders <b>LEFT</b>  <b>JOIN</b>  <b>LATERAL</b>  <b>TABLE</b>(unnest_udtf(tags))
          t <b>AS</b> tag <b>ON</b>  <b>TRUE</b>
        </p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5F53;&#x524D;&#xFF0C;&#x4EC5;&#x6587;&#x5B57;TRUE&#x652F;&#x6301;&#x4F5C;&#x4E3A;&#x9488;&#x5BF9;&#x6A2A;&#x5411;&#x8868;&#x7684;&#x5DE6;&#x5916;&#x90E8;&#x8054;&#x63A5;&#x7684;&#x8C13;&#x8BCD;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x5173;&#x8054;&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html">&#x65F6;&#x6001;<b>&#x8868;</b></a><b>&#x662F;&#x8DDF;&#x8E2A;&#x968F;&#x65F6;&#x95F4;&#x53D8;&#x5316;&#x7684;&#x8868;&#x3002;</b>
        </p>
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#temporal-table-functions">&#x65F6;&#x6001;<b>&#x8868;&#x529F;&#x80FD;</b></a><b>&#x63D0;&#x4F9B;&#x5BF9;&#x7279;&#x5B9A;&#x65F6;&#x95F4;&#x70B9;&#x65F6;&#x6001;&#x8868;&#x72B6;&#x6001;&#x7684;&#x8BBF;&#x95EE;&#x3002;&#x4F7F;&#x7528;&#x4E34;&#x65F6;&#x8868;&#x51FD;&#x6570;</b>&#x8054;&#x63A5;&#x8868;<b>&#x7684;&#x8BED;&#x6CD5;&#x4E0E;&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;</b>&#x8054;&#x63A5;<b>&#x7684;&#x8BED;&#x6CD5;&#x76F8;&#x540C;&#x3002;</b>
        </p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;&#x5F53;&#x524D;&#x4EC5;&#x652F;&#x6301;&#x4F7F;&#x7528;&#x4E34;&#x65F6;&#x8868;&#x7684;&#x5185;&#x90E8;&#x8054;&#x63A5;&#x3002;</b>
        </p>
        <p><b>&#x5047;&#x8BBE;</b>Rates<b>&#x662F;&#x4E00;&#x4E2A;</b><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#temporal-table-functions"><b>&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;</b></a><b>&#xFF0C;&#x5219;&#x8054;&#x63A5;&#x53EF;&#x4EE5;&#x7528;SQL&#x8868;&#x793A;&#x5982;&#x4E0B;&#xFF1A;</b>
        </p>
        <p><b>SELECT</b>
        </p>
        <p>o_amount, r_rate</p>
        <p><b>FROM</b>
        </p>
        <p>Orders,</p>
        <p> <b>LATERAL</b>  <b>TABLE</b> (Rates(o_proctime))</p>
        <p><b>WHERE</b>
        </p>
        <p>r_currency <b>=</b> o_currency</p>
        <p><b>&#x6709;&#x5173;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x68C0;&#x67E5;&#x66F4;&#x8BE6;&#x7EC6;&#x7684;</b>
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html"><b>&#x65F6;&#x6001;&#x8868;&#x6982;&#x5FF5;&#x63CF;&#x8FF0;</b>
            </a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>&#x5173;</b>
        </p>
        <p><b>&#x5173;&#x8054;&#x65F6;&#x6001;&#x8868;</b>
          <br />Batch Streming</p>
      </td>
      <td style="text-align:left">
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html"><b>&#x65F6;&#x6001;&#x8868;</b></a><b>&#x662F;</b>&#x8DDF;&#x8E2A;&#x968F;&#x65F6;&#x95F4;&#x53D8;&#x5316;&#x7684;&#x8868;<b>&#x3002;</b>
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#temporal-table"><b>&#x65F6;&#x6001;&#x8868;</b>
            </a>&#x63D0;&#x4F9B;&#x5BF9;&#x7279;&#x5B9A;&#x65F6;&#x95F4;&#x70B9;&#x7684;&#x4E34;&#x65F6;&#x8868;&#x7248;&#x672C;&#x7684;&#x8BBF;&#x95EE;&#x3002;</p>
        <p><b>&#x4EC5;&#x652F;&#x6301;&#x5E26;&#x6709;&#x5904;&#x7406;&#x65F6;&#x95F4;&#x65F6;&#x6001;&#x8868;&#x7684;&#x5185;&#x90E8;&#x8054;&#x63A5;&#x548C;&#x5DE6;&#x8054;&#x63A5;&#x3002;</b>
        </p>
        <p><b>&#x4E0B;&#x9762;&#x7684;&#x793A;&#x4F8B;&#x5047;&#x5B9A;LatestRates&#x662F;&#x4E00;&#x4E2A;&#x4EE5;&#x6700;&#x65B0;&#x901F;&#x7387;&#x5B9E;&#x73B0;&#x7684;</b>
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#temporal-table"><b>&#x65F6;&#x6001;&#x8868;</b>
            </a><b>&#x3002;</b>
        </p>
        <p><b>SELECT</b>
        </p>
        <p>o.amout, o.currency, r.rate, o.amount <b>*</b> r.rate</p>
        <p><b>FROM</b>
        </p>
        <p>Orders <b>AS</b> o</p>
        <p> <b>JOIN</b> LatestRates <b>FOR</b> SYSTEM_TIME <b>AS</b>  <b>OF</b> o.proctime <b>AS</b> r</p>
        <p> <b>ON</b> r.currency <b>=</b> o.currency</p>
        <p><b>&#x6709;&#x5173;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x68C0;&#x67E5;&#x66F4;&#x8BE6;&#x7EC6;&#x7684;&#x201C;</b>
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html"><b>&#x65F6;&#x6001;&#x8868;&#x201D;</b>
            </a><b>&#x6982;&#x5FF5;&#x63CF;&#x8FF0;&#x3002;</b>
        </p>
        <p><b>&#x4EC5;&#x5728;&#x7728;&#x773C;&#x8BA1;&#x5212;&#x7A0B;&#x5E8F;&#x4E2D;&#x53D7;&#x652F;&#x6301;&#x3002;</b>
        </p>
      </td>
    </tr>
  </tbody>
</table>### Set 操作

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
      <td style="text-align:left"><b>UnionAll</b>
        <br />Batch Streaming</td>
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
      <td style="text-align:left"><b>Intersect / Except</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p><b>SELECT</b>  <b>*</b>
        </p>
        <p><b>FROM</b> (</p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> a <b>%</b> 2 <b>=</b> 0)</p>
        <p> <b>INTERSECT</b>
        </p>
        <p>(<b>SELECT</b>  <b>user</b>  <b>FROM</b> Orders <b>WHERE</b> b <b>=</b> 0)</p>
        <p>)</p>
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
      <td style="text-align:left"><b>In</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x8868;&#x5B50;&#x67E5;&#x8BE2;&#x4E2D;&#x5B58;&#x5728;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;true&#x3002;
          &#x5B50;&#x67E5;&#x8BE2;&#x8868;&#x5FC5;&#x987B;&#x7531;&#x4E00;&#x5217;&#x7EC4;&#x6210;&#x3002;
          &#x6B64;&#x5217;&#x5FC5;&#x987B;&#x4E0E;&#x8868;&#x8FBE;&#x5F0F;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x6570;&#x636E;&#x7C7B;&#x578B;&#x3002;
          <br
          /><b>SELECT</b>  <b>user</b>, amount</p>
        <p><b>FROM</b> Orders</p>
        <p><b>WHERE</b> product <b>IN</b> (</p>
        <p> <b>SELECT</b> product <b>FROM</b> NewProducts</p>
        <p>)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x67E5;&#x8BE2;&#xFF0C;&#x8BE5;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Exists</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x5B50;&#x67E5;&#x8BE2;&#x8FD4;&#x56DE;&#x81F3;&#x5C11;&#x4E00;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;true&#x3002;
          &#x4EC5;&#x5F53;&#x53EF;&#x4EE5;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x64CD;&#x4F5C;&#x65F6;&#x624D;&#x652F;&#x6301;&#x3002;</p>
        <p><b>SELECT</b>  <b>user</b>, amount</p>
        <p><b>FROM</b> Orders</p>
        <p><b>WHERE</b> product <b>EXISTS</b> (</p>
        <p> <b>SELECT</b> product <b>FROM</b> NewProducts</p>
        <p>)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x67E5;&#x8BE2;&#xFF0C;&#x8BE5;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>### OrderBy & Limit

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

{% hint style="danger" %}
注意：Top-N仅支持Blink Planner
{% endhint %}

Top-N查询要求按列排序的N个最小或最大的值。最小值集和最大值集都被认为是Top-N查询。在需要只显示批处理/流表中的N个最底层记录或N个最顶层记录的情况下，Top-N查询很有用。 此结果集可用于进一步分析。

Flink使用OVER窗口子句和过滤条件的组合来表示Top-N查询。 借助OVER window PARTITION BY子句的强大功能，Flink还支持每组Top-N。 例如，每个类别中实时销量最高的前五种产品。 批处理表和流表上的SQL支持Top-N查询。

下面显示了TOP-N语句的语法：

```sql
SELECT [column_list]
FROM (
   SELECT [column_list],
     ROW_NUMBER() OVER ([PARTITION BY col1[, col2...]]
       ORDER BY col1 [asc|desc][, col2 [asc|desc]...]) AS rownum
   FROM table_name)
WHERE rownum <= N [AND conditions]
```

**参数规格：**

* `ROW_NUMBER()`：根据分区内各行的顺序，为每一行分配一个唯一的顺序号（从1开始）。目前，我们仅支持`ROW_NUMBER`作为窗口功能。将来，我们将支持`RANK()`和`DENSE_RANK()`。
* `PARTITION BY col1[, col2...]`：指定分区列。每个分区都有一个Top-N结果。
* `ORDER BY col1 [asc|desc][, col2 [asc|desc]...]`：指定排序列。在不同的列上，订购方向可以不同。
* `WHERE rownum <= N`：`rownum <= N`Flink识别此查询是Top-N查询所必需。N代表将保留N个最小或最大记录。
* `[AND conditions]`：可以在where子句中随意添加其他条件，但是其他条件只能与`rownum <= N`使用`AND`joining 组合使用。

{% hint style="danger" %}
注意：在流模式下TopN查询的结果实时更新的。Flink SQL将根据order键对输入数据流进行排序，因此，如果前N条记录已被更改，则更改后的记录将作为回退/更新记录发送给下游。建议使用支持更新的存储作为Top-N查询的sink。此外，如果top N记录需要存储在外部存储中，那么结果表应该具有与top N查询相同的惟一键。
{% endhint %}

Top-N查询的惟一键是分区列和rownum列的组合。Top-N查询还可以派生上游的唯一键。以下面的job为例，假设product\_id是ShopSales的唯一键，那么Top-N查询的唯一键是\[category, rownum\]和\[product\_id\]。

以下示例说明如何在流表上使用Top-N指定SQL查询。这是我们上面提到的“每个类别中实时销量最高的前五种产品”的示例。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// ingest a DataStream from an external source
DataStream<Tuple3<String, String, String, Long>> ds = env.addSource(...);
// register the DataStream as table "ShopSales"
tableEnv.createTemporaryView("ShopSales", ds, "product_id, category, product_name, sales");

// select top-5 products per category which have the maximum sales.
Table result1 = tableEnv.sqlQuery(
  "SELECT * " +
  "FROM (" +
  "   SELECT *," +
  "       ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) as row_num" +
  "   FROM ShopSales)" +
  "WHERE row_num <= 5");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// read a DataStream from an external source
val ds: DataStream[(String, String, String, Long)] = env.addSource(...)
// register the DataStream under the name "ShopSales"
tableEnv.createTemporaryView("ShopSales", ds, 'product_id, 'category, 'product_name, 'sales)


// select top-5 products per category which have the maximum sales.
val result1 = tableEnv.sqlQuery(
    """
      |SELECT *
      |FROM (
      |   SELECT *,
      |       ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) as row_num
      |   FROM ShopSales)
      |WHERE row_num <= 5
    """.stripMargin)
```
{% endtab %}
{% endtabs %}

#### 无排序输出优化

如上所述，rownum字段将作为惟一键的一个字段写入结果表，这可能导致将大量记录写入结果表。例如，当更新排名9的记录\(比如product-1001\)并将其排名提升到1时，排名1 ~ 9的所有记录都将作为更新消息输出到结果表中。如果结果表接收的数据太多，就会成为SQL作业的瓶颈。

优化方法是在Top-N查询的外部SELECT子句中省略rownum字段。这是合理的，因为前N条记录的数量通常不大，因此使用者可以自己快速地对记录进行排序。在上面的例子中，如果没有rownum字段，只需要将更改的记录\(product-1001\)发送到下游，这可以减少对结果表的很多IO。

下面的例子展示了如何优化上面的Top-N例子:

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// ingest a DataStream from an external source
DataStream<Tuple3<String, String, String, Long>> ds = env.addSource(...);
// register the DataStream as table "ShopSales"
tableEnv.createTemporaryView("ShopSales", ds, "product_id, category, product_name, sales");

// select top-5 products per category which have the maximum sales.
Table result1 = tableEnv.sqlQuery(
  "SELECT product_id, category, product_name, sales " + // omit row_num field in the output
  "FROM (" +
  "   SELECT *," +
  "       ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) as row_num" +
  "   FROM ShopSales)" +
  "WHERE row_num <= 5");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// read a DataStream from an external source
val ds: DataStream[(String, String, String, Long)] = env.addSource(...)
// register the DataStream under the name "ShopSales"
tableEnv.createTemporaryView("ShopSales", ds, 'product_id, 'category, 'product_name, 'sales)


// select top-5 products per category which have the maximum sales.
val result1 = tableEnv.sqlQuery(
    """
      |SELECT product_id, category, product_name, sales  -- omit row_num field in the output
      |FROM (
      |   SELECT *,
      |       ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) as row_num
      |   FROM ShopSales)
      |WHERE row_num <= 5
    """.stripMargin)
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**流传输模式中的注意事项:**为了将以上查询输出到外部存储并获得正确的结果，外部存储必须具有与Top-N查询相同的唯一键。在上面的示例查询中，如果`product_id`\_是查询的唯一键，则外部表也应具有`product_id`作为唯一键。
{% endhint %}

### 重复数据删除

{% hint style="danger" %}
**注意：**重复数据删除仅在Blink Planner中支持。
{% endhint %}

重复数据删除是指删除在一组列上重复的行，仅保留第一个或最后一个。在某些情况下，上游ETL作业不是一次精确的端到端，这可能导致在故障转移的情况下，接收器中有重复的记录。然而，重复的记录会影响到下游的分析工作（例如正确性`SUM`，`COUNT`）。因此，在进一步分析之前需要进行重复数据删除。

Flink用来`ROW_NUMBER()`删除重复项，就像Top-N查询一样。从理论上讲，重复数据删除是Top-N的一种特殊情况，其中N为1，并按处理时间或事件时间排序。

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

**参数规格：**

* `ROW_NUMBER()`：从第一行开始，为每行分配一个唯一的顺序号。
* `PARTITION BY col1[, col2...]`：指定分区列，即重复数据删除键。
* `ORDER BY time_attr [asc|desc]`：指定排序列，它必须是[time属性](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html)。目前仅支持[proctime属性](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html#processing-time)。将来将支持[行时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html#event-time)。按ASC排序意味着保留第一行，按DESC排序意味着保留最后一行。
* `WHERE rownum = 1`：`rownum = 1`Flink识别此查询为重复数据删除所必需。

以下示例说明如何在流表上指定带有重复数据删除功能的SQL查询。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// ingest a DataStream from an external source
DataStream<Tuple3<String, String, String, Integer>> ds = env.addSource(...);
// register the DataStream as table "Orders"
tableEnv.createTemporaryView("Orders", ds, "order_id, user, product, number, proctime.proctime");

// remove duplicate rows on order_id and keep the first occurrence row,
// because there shouldn't be two orders with the same order_id.
Table result1 = tableEnv.sqlQuery(
  "SELECT order_id, user, product, number " +
  "FROM (" +
  "   SELECT *," +
  "       ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY proctime ASC) as row_num" +
  "   FROM Orders)" +
  "WHERE row_num = 1");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// read a DataStream from an external source
val ds: DataStream[(String, String, String, Int)] = env.addSource(...)
// register the DataStream under the name "Orders"
tableEnv.createTemporaryView("Orders", ds, 'order_id, 'user, 'product, 'number, 'proctime.proctime)

// remove duplicate rows on order_id and keep the first occurrence row,
// because there shouldn't be two orders with the same order_id.
val result1 = tableEnv.sqlQuery(
    """
      |SELECT order_id, user, product, number
      |FROM (
      |   SELECT *,
      |       ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY proctime DESC) as row_num
      |   FROM Orders)
      |WHERE row_num = 1
    """.stripMargin)
```
{% endtab %}
{% endtabs %}

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
</table>