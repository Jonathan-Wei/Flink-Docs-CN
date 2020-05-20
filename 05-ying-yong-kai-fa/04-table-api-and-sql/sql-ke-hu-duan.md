# SQL客户端

Flink的表和SQL API可以处理用SQL语言编写的查询，但是这些查询需要嵌入到用Java或Scala编写的表程序中。此外，这些程序在提交到集群之前需要与构建工具打包。这或多或少地限制了Java/Scala程序员对Flink的使用。

SQL客户端旨在提供一种简单的方式，无需一行Java或Scala代码，即可将表程序编写、调试和提交到Flink集群。SQL客户机CLI允许检索和可视化命令行上运行的分布式应用程序的实时结果。

![](../../.gitbook/assets/sql_client_demo.gif)

{% hint style="danger" %}
SQL客户机处于早期开发阶段。尽管该应用程序还没有生产就绪，但是对于原型化和使用Flink SQL来说，它仍然是一个非常有用的工具。将来，社区计划通过提供基于rest的[SQL客户端网关](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sqlClient.html#limitations--future)来扩展其功能。
{% endhint %}

## 入门

本节描述如何从命令行设置和运行第一个Flink SQL程序。

SQL客户机绑定在常规Flink发行版中，因此可以开箱即用。它只需要一个正在运行的Flink集群，其中可以执行表程序。有关设置Flink集群的更多信息，请参阅[集群和部署](https://ci.apache.org/projects/flink/flink-docs-master/ops/deployment/cluster_setup.html)部分。如果您只是想尝试SQL客户机，还可以使用以下命令启动一个本地集群，其中有一个worker:

```bash
./bin/start-cluster.sh
```

### 启动SQL客户端CLI

SQL客户机脚本也位于Flink的bin目录中。将来，用户将有两种可能启动SQL客户机CLI，一种是启动嵌入式独立进程，另一种是连接到远程SQL客户机网关。目前只支持嵌入式模式。您可以通过调用以下命令启动CLI:

```bash
./bin/sql-client.sh embedded
```

默认情况下，SQL客户端将读取环境文件`./conf/sql-client-defaults.yaml`中的配置。有关环境文件结构的更多信息，请参阅[配置部分](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sqlClient.html#environment-files)。

### 运行SQL查询

一旦CLI启动，您就可以使用HELP命令列出所有可用的SQL语句。为了验证您的设置和集群连接，您可以输入第一个SQL查询并按enter键执行它:

```sql
SELECT 'Hello World';
```

该查询不需要表源，只生成一行结果。CLI将从集群中检索结果并将其可视化。您可以通过按Q键关闭结果视图。

CLI支持**两种模式**来维护和可视化结果。

**表模式**在内存中实现结果，并以规则的分页表表示将结果可视化。它可以通过在CLI中执行以下命令来启用:

```sql
SET execution.result-mode=table;
```

**变更日志模式**不会物化结果，而是可视化由插入\(+\)和撤销\(-\)组成的连续查询生成的结果流。

```sql
SET execution.result-mode=changelog;
```

您可以使用以下查询查看这两种结果模式的运行情况:

```sql
SELECT name, COUNT(*) AS cnt FROM (VALUES ('Bob'), ('Alice'), ('Greg'), ('Bob')) AS NameTable(name) GROUP BY name;
```

该查询执行一个有界word count示例。

在变更日志模式下，可视化的变更日志应该类似于:

```text
+ Bob, 1
+ Alice, 1
+ Greg, 1
- Bob, 1
+ Bob, 2
```

在表格模式下，不断更新可视化结果表格，直到表格程序结束:

```text
Bob, 2
Alice, 1
Greg, 1
```

这两种结果模式在SQL查询原型化过程中都很有用。在这两种模式中，结果都存储在SQL客户机的Java堆内存中。为了保持CLI接口的响应，变更日志模式只显示最新的1000个更改。table模式允许在较大的结果中导航，这些结果只受可用主内存和配置的最大行数\(max-table-result-rows\)的限制。

{% hint style="danger" %}
在批处理环境中执行的查询只能使用表结果模式检索。
{% endhint %}

定义查询之后，可以将其作为一个长时间运行的、分离的Flink作业提交给集群。为此，需要使用[INSERT INTO语句](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sqlClient.html#detached-sql-queries)指定存储结果的目标系统。[配置部分](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sqlClient.html#configuration)解释了如何声明读取数据的表源，如何声明写入数据的表接收，以及如何配置其他表程序属性。

## 配置

可以使用以下可选CLI命令启动SQL客户机。下文各段将详细讨论这些问题。

```bash
./bin/sql-client.sh embedded --help

Mode "embedded" submits Flink jobs from the local machine.

  Syntax: embedded [OPTIONS]
  "embedded" mode options:
     -d,--defaults <environment file>      The environment properties with which
                                           every new session is initialized.
                                           Properties might be overwritten by
                                           session properties.
     -e,--environment <environment file>   The environment properties to be
                                           imported into the session. It might
                                           overwrite default environment
                                           properties.
     -h,--help                             Show the help message with
                                           descriptions of all options.
     -j,--jar <JAR file>                   A JAR file to be imported into the
                                           session. The file might contain
                                           user-defined classes needed for the
                                           execution of statements such as
                                           functions, table sources, or sinks.
                                           Can be used multiple times.
     -l,--library <JAR directory>          A JAR file directory with which every
                                           new session is initialized. The files
                                           might contain user-defined classes
                                           needed for the execution of
                                           statements such as functions, table
                                           sources, or sinks. Can be used
                                           multiple times.
     -s,--session <session identifier>     The identifier for a session.
                                           'default' is the default identifier.
```

### 环境文件

 SQL查询需要在其中执行配置环境。所谓的_环境文件_定义了可用的目录，表源和接收器，用户定义的函数以及执行和部署所需的其他属性。

 每个环境文件都是常规的[YAML文件](http://yaml.org/)。下面提供了此类文件的示例。

```yaml
# Define tables here such as sources, sinks, views, or temporal tables.

tables:
  - name: MyTableSource
    type: source-table
    update-mode: append
    connector:
      type: filesystem
      path: "/path/to/something.csv"
    format:
      type: csv
      fields:
        - name: MyField1
          type: INT
        - name: MyField2
          type: VARCHAR
      line-delimiter: "\n"
      comment-prefix: "#"
    schema:
      - name: MyField1
        type: INT
      - name: MyField2
        type: VARCHAR
  - name: MyCustomView
    type: view
    query: "SELECT MyField2 FROM MyTableSource"

# Define user-defined functions here.

functions:
  - name: myUDF
    from: class
    class: foo.bar.AggregateUDF
    constructor:
      - 7.6
      - false

# Define available catalogs

catalogs:
   - name: catalog_1
     type: hive
     property-version: 1
     hive-conf-dir: ...
   - name: catalog_2
     type: hive
     property-version: 1
     default-database: mydb2
     hive-conf-dir: ...
     hive-version: 1.2.1

# Properties that change the fundamental execution behavior of a table program.

execution:
  planner: blink                    # optional: either 'blink' (default) or 'old'
  type: streaming                   # required: execution mode either 'batch' or 'streaming'
  result-mode: table                # required: either 'table' or 'changelog'
  max-table-result-rows: 1000000    # optional: maximum number of maintained rows in
                                    #   'table' mode (1000000 by default, smaller 1 means unlimited)
  time-characteristic: event-time   # optional: 'processing-time' or 'event-time' (default)
  parallelism: 1                    # optional: Flink's parallelism (1 by default)
  periodic-watermarks-interval: 200 # optional: interval for periodic watermarks (200 ms by default)
  max-parallelism: 16               # optional: Flink's maximum parallelism (128 by default)
  min-idle-state-retention: 0       # optional: table program's minimum idle state time
  max-idle-state-retention: 0       # optional: table program's maximum idle state time
  current-catalog: catalog_1        # optional: name of the current catalog of the session ('default_catalog' by default)
  current-database: mydb1           # optional: name of the current database of the current catalog
                                    #   (default database of the current catalog by default)
  restart-strategy:                 # optional: restart strategy
    type: fallback                  #   "fallback" to global restart strategy by default

# Configuration options for adjusting and tuning table programs.

# A full list of options and their default values can be found
# on the dedicated "Configuration" page.
configuration:
  table.optimizer.join-reorder-enabled: true
  table.exec.spill-compression.enabled: true
  table.exec.spill-compression.block-size: 128kb

# Properties that describe the cluster to which table programs are submitted to.

deployment:
  response-timeout: 5000
```

配置：

* 使用从CSV文件读取的表源mytables源定义一个环境
*  定义使用SQL查询声明虚拟表的视图MyCustomView
* 定义一个用户定义的函数myUDF，该函数可以使用类名和两个构造函数参数实例化
* 指定在此流环境中执行的查询的并行度为1
* 指定事件时间特征
* 在表结果模式运行查询。

根据用例的不同，配置可以分为多个文件。因此，环境文件可以用于一般目的\(使用缺省环境文件—缺省环境\)，也可以基于每个会话创建\(使用会话环境文件—环境\)。每个CLI会话都使用默认属性和会话属性初始化。例如，默认环境文件可以指定在每个会话中查询的所有表源，而会话环境文件仅声明特定的状态保持时间和并行性。启动CLI应用程序时，可以传递缺省环境文件和会话环境文件。如果没有指定默认环境文件，SQL客户机将在Flink的配置目录中搜索./conf/sql-client-defaults.yaml文件。

{% hint style="danger" %}
在CLI会话中设置的属性\(例如使用set命令\)具有最高的优先级:
{% endhint %}

```text
CLI commands > session environment file > defaults environment file
```

#### 重启策略

重启策略控制在失败的情况下Flink作业如何重新启动。与Flink集群的全局重启策略类似，可以在环境文件中声明更细粒度的重启配置。

支持以下策略:

```yaml
execution:
  # falls back to the global strategy defined in flink-conf.yaml
  restart-strategy:
    type: fallback

  # job fails directly and no restart is attempted
  restart-strategy:
    type: none

  # attempts a given number of times to restart the job
  restart-strategy:
    type: fixed-delay
    attempts: 3      # retries before job is declared as failed (default: Integer.MAX_VALUE)
    delay: 10000     # delay in ms between retries (default: 10 s)

  # attempts as long as the maximum number of failures per time interval is not exceeded
  restart-strategy:
    type: failure-rate
    max-failures-per-interval: 1   # retries in interval until failing (default: 1)
    failure-rate-interval: 60000   # measuring interval in ms for failure rate
    delay: 10000                   # delay in ms between retries (default: 10 s)
```

### 依赖

SQL客户机不需要使用Maven或SBT设置Java项目。相反，您可以将依赖项作为常规JAR文件传递到集群。您可以单独指定每个JAR文件\(使用--jar\)，也可以定义整个库目录\(使用--library\)。对于外部系统\(如Apache Kafka\)和相应数据格式\(如JSON\)的连接器，Flink提供了现成的JAR包。这些JAR文件以sql-jar作为后缀，可以从Maven中央存储库下载每个版本的JAR文件。

 提供的SQL JAR的完整列表以及有关如何使用它们的文档可以在与[外部系统](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html)的[连接页面上找到](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html)。

下面的示例显示了一个环境文件，该文件定义了一个从Apache Kafka读取JSON数据的表源。

```yaml
tables:
  - name: TaxiRides
    type: source-table
    update-mode: append
    connector:
      property-version: 1
      type: kafka
      version: "0.11"
      topic: TaxiRides
      startup-mode: earliest-offset
      properties:
        zookeeper.connect: localhost:2181
        bootstrap.servers: localhost:9092
        group.id: testGroup
    format:
      property-version: 1
      type: json
      schema: "ROW<rideId LONG, lon FLOAT, lat FLOAT, rideTime TIMESTAMP>"
    schema:
      - name: rideId
        data-type: BIGINT
      - name: lon
        data-type: FLOAT
      - name: lat
        data-type: FLOAT
      - name: rowTime
        data-type: TIMESTAMP(3)
        rowtime:
          timestamps:
            type: "from-field"
            from: "rideTime"
          watermarks:
            type: "periodic-bounded"
            delay: "60000"
      - name: procTime
        data-type: TIMESTAMP(3)
        proctime: true
```

TaxiRide表的结果模式包含JSON模式的大部分字段。此外，它还添加了一个rowtime属性和一个processing-time属性procTime。

connector和format都允许定义一个属性版本\(当前的版本1\)，以便将来向后兼容。

### 用户定义函数

SQL客户机允许用户创建用于SQL查询的自定义用户定义函数。目前，这些函数被限制在Java/Scala类中以编程方式定义。

为了提供用户定义的函数，首先需要实现和编译扩展ScalarFunction、AggregateFunction或TableFunction的函数类\(参见用户定义的函数\)。然后可以将一个或多个函数打包到SQL客户机的依赖项JAR中。

所有函数在调用之前必须在环境文件中声明。对于函数列表中的每个项，必须指定：

* name：函数注册时使用的名称， 
* from：指定函数的源函数\(目前限制为class\)
* class：指示函数的完全限定类名和用于实例化的可选构造函数参数列表。

```yaml
functions:
  - name: ...               # required: name of the function
    from: class             # required: source of the function (can only be "class" for now)
    class: ...              # required: fully qualified class name of the function
    constructor:            # optional: constructor parameters of the function class
      - ...                 # optional: a literal parameter with implicit type
      - class: ...          # optional: full class name of the parameter
        constructor:        # optional: constructor parameters of the parameter's class
          - type: ...       # optional: type of the literal parameter
            value: ...      # optional: value of the literal parameter
```

确保指定参数的顺序和类型严格匹配函数类的一个构造函数。  


#### 构造函数参数

根据用户定义的函数，可能需要在SQL语句中使用实现之前对其进行参数化。

如上例所示，在声明用户定义的函数时，可以通过以下三种方法之一使用构造函数参数来配置类:

**隐式类型的字面值**:SQL客户机将根据文字值本身自动派生类型。目前，这里只支持布尔值、INT值、DOUBLE值和VARCHAR值。如果自动派生没有按照预期工作\(例如，您需要一个VARCHAR false\)，则使用显式类型。

```text
- true         # -> BOOLEAN (case sensitive)
- 42           # -> INT
- 1234.222     # -> DOUBLE
- foo          # -> VARCHAR
```

**具有显式类型的字面值**:为了类型安全，显式声明具有类型和值属性的参数。

```text
- type: DECIMAL
  value: 11111111111111111
```

下表说明了受支持的Java参数类型和相应的SQL类型字符串。

| Java类型 | SQL类型 |
| :--- | :--- |
| java.math.BigDecimal | DECIMAL |
| java.lang.Boolean | BOOLEAN |
| java.lang.Byte | TINYINT |
| java.lang.Double | DOUBLE |
| java.lang.Float | REAL，FLOAT |
| java.lang.Integer | INTEGER，INT |
| java.lang.Long | BIGINT |
| java.lang.Short | SMALLINT |
| java.lang.String | VARCHAR |

还不支持其他更多类型\(例如，TIMESTAMP或ARRAY\)、基本类型和null。

**（嵌套）类实例：**除字面值外，您还可以通过指定`class`和`constructor`属性为构造函数参数创建（嵌套）类实例。可以递归地执行此过程，直到所有构造函数参数都使用字面值表示。

```yaml
- class: foo.bar.paramClass
  constructor:
    - StarryName
    - class: java.lang.Integer
      constructor:
        - class: java.lang.String
          constructor:
            - type: VARCHAR
              value: 3
```

## Catalogs

可以将目录定义为一组YAML属性，并在启动SQL客户端时自动注册到环境中。

用户可以指定要在SQL CLI中使用哪个目录作为当前目录，以及要使用目录的哪个数据库作为当前数据库。

```text
catalogs:
   - name: catalog_1
     type: hive
     property-version: 1
     default-database: mydb2
     hive-version: 1.2.1
     hive-conf-dir: <path of Hive conf directory>
   - name: catalog_2
     type: hive
     property-version: 1
     hive-conf-dir: <path of Hive conf directory>

execution:
   ...
   current-catalog: catalog_1
   current-database: mydb1
```

有关目录的更多信息，请参见[目录](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)。

## 分离的SQL查询

为了定义端到端SQL管道，可以使用SQL的INSERT INTO语句向Flink集群提交长时间运行的分离查询。这些查询将结果生成到外部系统，而不是SQL客户机。这允许处理更高的并行性和更大数量的数据。CLI本身对提交后分离的查询没有任何控制。

```sql
INSERT INTO MyTableSink SELECT * FROM MyTableSource
```

必须在环境文件中声明表sink MyTableSink。有关受支持的外部系统及其配置的更多信息，请参见[连接](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/connect.html)页。下面是Apache Kafka表接收器的一个例子。

```yaml
tables:
  - name: MyTableSink
    type: sink-table
    update-mode: append
    connector:
      property-version: 1
      type: kafka
      version: "0.11"
      topic: OutputTopic
      properties:
        zookeeper.connect: localhost:2181
        bootstrap.servers: localhost:9092
        group.id: testGroup
    format:
      property-version: 1
      type: json
      derive-schema: true
    schema:
      - name: rideId
        data-type: BIGINT
      - name: lon
        data-type: FLOAT
      - name: lat
        data-type: FLOAT
      - name: rideTime
        data-type: TIMESTAMP(3)
```

SQL客户端确保将语句成功提交到群集。提交查询后，CLI将显示有关Flink作业的信息。

```text
[INFO] Table update statement has been successfully submitted to the cluster:
Cluster ID: StandaloneClusterId
Job ID: 6f922fe5cba87406ff23ae4a7bb79044
Web interface: http://localhost:8081
```

{% hint style="danger" %}
SQL Client在提交后不会跟踪正在运行的Flink作业的状态。提交后可以关闭CLI进程，而不会影响分离的查询。Flink的[重启策略](https://ci.apache.org/projects/flink/flink-docs-master/dev/restart_strategies.html)负责容错。可以使用Flink的Web界面，命令行或REST API取消查询。
{% endhint %}

## SQL视图

视图允许从SQL查询中定义虚拟表。视图定义立即被解析和验证。但是，实际执行是在提交一般INSERT INTO或SELECT语句时访问视图。

视图可以在环境文件中定义，也可以在CLI会话中定义。

下面的例子展示了如何在一个文件中定义多个视图。这些视图是按照在环境文件中定义它们的顺序注册的。支持像视图A依赖于视图B依赖于视图C这样的引用链。

```yaml
tables:
  - name: MyTableSource
    # ...
  - name: MyRestrictedView
    type: view
    query: "SELECT MyField2 FROM MyTableSource"
  - name: MyComplexView
    type: view
    query: >
      SELECT MyField2 + 42, CAST(MyField1 AS VARCHAR)
      FROM MyTableSource
      WHERE MyField2 > 200
```

与表source和sink类似，在会话环境文件中定义的视图具有最高的优先级。

还可以使用CREATE VIEW语句在CLI会话中创建视图:

```sql
CREATE VIEW MyNewView AS SELECT MyField2 FROM MyTableSource;
```

在CLI会话中创建的视图也可以使用DROP VIEW语句再次删除:

```sql
DROP VIEW MyNewView;
```

{% hint style="danger" %}
CLI中视图的定义仅限于上述语法。在未来的版本中，将支持为视图定义模式或转义表名中的空白。
{% endhint %}

## 时态表

[时态表](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/temporal_tables.html)允许更改历史表上的\(参数化\)视图，该视图在特定时间点返回表的内容。这对于在特定时间戳将表与另一个表的内容连接起来特别有用。更多信息可以在[时态表连接](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/joins.html#join-with-a-temporal-table)页中找到。

下面的例子展示了如何定义一个时态表SourceTemporalTable:

```yaml
tables:

  # Define the table source (or view) that contains updates to a temporal table
  - name: HistorySource
    type: source-table
    update-mode: append
    connector: # ...
    format: # ...
    schema:
      - name: integerField
        data-type: INT
      - name: stringField
        data-type: STRING
      - name: rowtimeField
        data-type: TIMESTAMP(3)
        rowtime:
          timestamps:
            type: from-field
            from: rowtimeField
          watermarks:
            type: from-source

  # Define a temporal table over the changing history table with time attribute and primary key
  - name: SourceTemporalTable
    type: temporal-table
    history-table: HistorySource
    primary-key: integerField
    time-attribute: rowtimeField  # could also be a proctime field
```

如示例所示，表源、视图和时态表的定义可以相互混合。它们是按照在环境文件中定义它们的顺序注册的。例如，时态表可以引用依赖于另一个视图或表源的视图。

## 局限与未来

当前的SQL客户端实现处于非常早期的开发阶段，将来可能会作为更大的Flink改进建议24 \(FLIP-24\)的一部分进行更改。欢迎加入讨论，并公开讨论您认为有用的bug和特性。

