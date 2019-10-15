# 概述

[Apache Hive](https://hive.apache.org/)已成为数据仓库生态系统的焦点。它不仅是大数据分析和ETL的SQL引擎，还是数据管理平台，可以在其中发现，定义和发展数据。

Flink提供与Hive的双重集成。第一个是利用Hive的Metastore作为持久目录，用于跨会话存储Flink特定元数据。第二个是提供Flink作为读取和编写Hive表的替代引擎。

Hive Catalog设计为与现有Hive安装兼容的“开箱即用”方式。无需修改​​现有的Hive Metastore或更改表的数据放置或分区。

## 支持的Hive版本

Flink支持Hive `2.3.4`，`1.2.1`并且依赖于Hive对其他次要版本的兼容性保证。

如果你使用不同较小的Hive版本（如`1.2.2`或）`2.3.1`，也可以选择最接近的版本`1.2.1`（for `1.2.2`）或`2.3.4`（for `2.3.1`）来解决。例如，你希望使用Flink `2.3.1`在sql客户端中集成hive版本，只需将hive-version设置`2.3.4`为YAML配置即可。同样，在通过Table API创建HiveCatalog实例时传递版本字符串。

欢迎用户使用此解决方法尝试不同的版本。由于只有`2.3.4`和`1.2 .1`已经过测试，可能会有意想不到的问题。我们将在未来的版本中测试和支持更多版本。

### Depedencies

要与Hive集成，用户需要在他们的项目中使用以下依赖项。

{% tabs %}
{% tab title="Hive 2.3.4" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-hive_2.11</artifactId>
  <version>1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hadoop Dependencies -->

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-hadoop-compatibility_1.9.0</artifactId>
  <version>1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hive 2.3.4 is built with Hadoop 2.7.2. We pick 2.7.5 which flink-shaded-hadoop is pre-built with, but users can pick their own hadoop version, as long as it's compatible with Hadoop 2.7.2 -->

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-shaded-hadoop-2-uber</artifactId>
  <version>2.7.5-1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hive Metastore -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>2.3.4</version>
</dependency>
```
{% endtab %}

{% tab title="Hive 1.2.1" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-hive_2.11</artifactId>
  <version>1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hadoop Dependencies -->

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-hadoop-compatibility_1.9.0</artifactId>
  <version>1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hive 1.2.1 is built with Hadoop 2.6.0. We pick 2.6.5 which flink-shaded-hadoop is pre-built with, but users can pick their own hadoop version, as long as it's compatible with Hadoop 2.6.0 -->

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-shaded-hadoop-2-uber</artifactId>
  <version>2.6.5-1.9.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hive Metastore -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-metastore</artifactId>
    <version>1.2.1</version>
</dependency>

<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>1.2.1</version>
</dependency>

<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libfb303</artifactId>
    <version>0.9.3</version>
</dependency>
```
{% endtab %}
{% endtabs %}

## 连接到Hive

通过Table Environment或YAML配置使用Hive Catalog连接到现有Hive。

{% tabs %}
{% tab title="Java" %}
```java
String name            = "myhive";
String defaultDatabase = "mydatabase";
String hiveConfDir     = "/opt/hive-conf";
String version         = "2.3.4"; // or 1.2.1

HiveCatalog hive = new HiveCatalog(name, defaultDatabase, hiveConfDir, version);
tableEnv.registerCatalog("myhive", hive);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val name            = "myhive"
val defaultDatabase = "mydatabase"
val hiveConfDir     = "/opt/hive-conf"
val version         = "2.3.4" // or 1.2.1

val hive = new HiveCatalog(name, defaultDatabase, hiveConfDir, version)
tableEnv.registerCatalog("myhive", hive)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
catalogs:
   - name: myhive
     type: hive
     property-version: 1
     hive-conf-dir: /opt/hive-conf
     hive-version: 2.3.4 # or 1.2.1
```
{% endtab %}
{% endtabs %}

## 支持的类型

目前`HiveCatalog`支持大多数Flink数据类型，具有以下映射：



| Flink数据类型 | Hive数据类型 |
| :--- | :--- |
| CHAR（P） | CHAR（P） |
| VARCHAR（P） | VARCHAR（P） |
| STRING | STRING |
| BOOLEAN | BOOLEAN |
| TINYINT | TINYINT |
| SMALLINT | SMALLINT |
| INT | INT |
| BIGINT | LONG |
| FLOAT | FLOAT |
| DOUBLE | DOUBLE |
| DECIMAL\(p, s\) | DECIMAL\(p, s\) |
| DATE | DATE |
| BYTES | BINARY |
| ARRAY &lt;T&gt; | LIST &lt;T&gt; |
| MAP &lt;K，V&gt; | MAP &lt;K，V&gt; |
| ROW | STRUCT |

### 限制

Hive的数据类型中的以下限制会影响Flink和Hive之间的映射：

* `CHAR(p)` 最大长度 255
* `VARCHAR(p)` 最大长度为65535
* Hive `MAP`只支持简单key类型，而Flink `MAP`可以是任何数据类型
* 不支持Hive `UNION`类型
* Flink的`INTERVAL`类型无法映射到Hive `INTERVAL`类型
* Flink `TIMESTAMP_WITH_TIME_ZONE`和`TIMESTAMP_WITH_LOCAL_TIME_ZONE`Hive不支持
* 由于精度差异，Flink的`TIMESTAMP_WITHOUT_TIME_ZONE`类型无法映射到Hive的`TIMESTAMP`类型。
* Flink`MULTISET`Hive不支持

