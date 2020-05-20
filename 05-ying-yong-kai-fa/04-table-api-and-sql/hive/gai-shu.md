# 概述

[Apache Hive](https://hive.apache.org/)已将自己确立为数据仓库生态系统的焦点。它不仅充当用于大数据分析和ETL的SQL引擎，而且还充当在其中发现，定义和发展数据的数据管理平台。

Flink提供了与Hive的双重集成。

首先是利用Hive的Metastore作为一个持久的目录，用Flink的HiveCatalog跨会话存储Flink特定的元数据。例如，用户可以使用HiveCatalog将Kafka或ElasticSearch表存储在Hive Metastore中，然后在SQL查询中重用它们。

其次是提供Flink作为读取和写入Hive表的替代引擎。

HiveCatalog被设计成与现有的Hive安装兼容的“开箱即用”的方式。不需要修改现有的Hive Metastore或更改表的数据位置或分区。

## 支持的Hive版本

Flink支持以下Hive版本。

* 1.0
  * 1.0.0
  * 1.0.1
* 1.1
  * 1.1.0
  * 1.1.1
* 1.2
  * 1.2.0
  * 1.2.1
  * 1.2.2
* 2.0
  * 2.0.0
  * 2.0.1
* 2.1
  * 2.1.0
  * 2.1.1
* 2.2
  * 2.2.0
* 2.3
  * 2.3.0
  * 2.3.1
  * 2.3.2
  * 2.3.3
  * 2.3.4
  * 2.3.5
  * 2.3.6
* 3.1
  * 3.1.0
  * 3.1.1
  * 3.1.2

请注意，Hive本身具有适用于不同版本的不同功能，并且这些问题不是由Flink引起的：

* Hive内置功能在1.2.0及更高版本中受支持。
* 3.1.0及更高版本支持列约束，即PRIMARY KEY和NOT NULL。
* 1.2.0及更高版本支持更改表统计信息。
*  1.2.0及更高版本支持`DATE`列统计信息。
* 2.0.x不支持写入ORC表。

### Depedencies

要与Hive集成，用户需要在他们的项目中使用以下依赖项。

{% tabs %}
{% tab title="Hive 2.3.4" %}
```markup
/flink-1.10.0
   /lib

       // Flink's Hive connector.Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.7.5-8.0.jar

       // Hive dependencies
       hive-exec-2.3.4.jar
```
{% endtab %}

{% tab title="Hive 1.0.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.6.5-8.0.jar

       // Hive dependencies
       hive-metastore-1.0.0.jar
       hive-exec-1.0.0.jar
       libfb303-0.9.0.jar // libfb303 is not packed into hive-exec in some versions, need to add it separately
```
{% endtab %}

{% tab title="HIve 1.1.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.6.5-8.0.jar

       // Hive dependencies
       hive-metastore-1.1.0.jar
       hive-exec-1.1.0.jar
       libfb303-0.9.2.jar // libfb303 is not packed into hive-exec in some versions, need to add it separately
```
{% endtab %}

{% tab title="Hive 1.2.1" %}
```markup
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.6.5-8.0.jar

       // Hive dependencies
       hive-metastore-1.2.1.jar
       hive-exec-1.2.1.jar
       libfb303-0.9.2.jar // libfb303 is not packed into hive-exec in some versions, need to add it separately
```
{% endtab %}

{% tab title="Hive 2.0.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.7.5-8.0.jar

       // Hive dependencies
       hive-exec-2.0.0.jar
```
{% endtab %}

{% tab title="Hive 2.1.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.7.5-8.0.jar

       // Hive dependencies
       hive-exec-2.1.0.jar
```
{% endtab %}

{% tab title="Hive 2.2.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.7.5-8.0.jar

       // Hive dependencies
       hive-exec-2.2.0.jar

       // Orc dependencies -- required by the ORC vectorized optimizations
       orc-core-1.4.3.jar
       aircompressor-0.8.jar // transitive dependency of orc-core
```
{% endtab %}

{% tab title="Hive 3.1.0" %}
```
/flink-1.10.0
   /lib

       // Flink's Hive connector. Contains flink-hadoop-compatibility and flink-orc jars
       flink-connector-hive_2.11-1.10.0.jar

       // Hadoop dependencies
       // You can pick a pre-built Hadoop uber jar provided by Flink, alternatively
       // you can use your own hadoop jars. Either way, make sure it's compatible with your Hadoop
       // cluster and the Hive version you're using.
       flink-shaded-hadoop-2-uber-2.8.3-8.0.jar

       // Hive dependencies
       hive-exec-3.1.0.jar
       libfb303-0.9.3.jar // libfb303 is not packed into hive-exec in some versions, need to add it separately
```
{% endtab %}
{% endtabs %}

如果要构建自己的程序，则在mvn文件中需要以下依赖项。建议不要在生成的jar文件中包括这些依赖项。应该在运行时如上所述添加依赖项。

```markup
<!-- Flink Dependency -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-hive_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>

<!-- Hive Dependency -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>${hive.version}</version>
    <scope>provided</scope>
</dependency>
```

## 连接到Hive

 通过表环境或YAML配置，使用 [Catalog接口](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)和[HiveCatalog](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/hive_catalog.html)连接到现有的Hive安装。

 如果`hive-conf/hive-site.xml`文件存储在远程存储系统中，则用户应首先将配置单元配置文件下载到其本地环境。

请注意，虽然HiveCatalog不需要特定的计划程序，但读取/写入Hive表仅适用于Blink Planner。因此，强烈建议在连接到Hive仓库时使用Blink Planner。

以Hive版本2.3.4为例：

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build();
TableEnvironment tableEnv = TableEnvironment.create(settings);

String name            = "myhive";
String defaultDatabase = "mydatabase";
String hiveConfDir     = "/opt/hive-conf"; // a local path
String version         = "2.3.4";

HiveCatalog hive = new HiveCatalog(name, defaultDatabase, hiveConfDir, version);
tableEnv.registerCatalog("myhive", hive);

// set the HiveCatalog as the current catalog of the session
tableEnv.useCatalog("myhive");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build()
val tableEnv = TableEnvironment.create(settings)

val name            = "myhive"
val defaultDatabase = "mydatabase"
val hiveConfDir     = "/opt/hive-conf" // a local path
val version         = "2.3.4"

val hive = new HiveCatalog(name, defaultDatabase, hiveConfDir, version)
tableEnv.registerCatalog("myhive", hive)

// set the HiveCatalog as the current catalog of the session
tableEnv.useCatalog("myhive")
```
{% endtab %}

{% tab title="YAML" %}
```yaml
execution:
    planner: blink
    ...
    current-catalog: myhive  # set the HiveCatalog as the current catalog of the session
    current-database: mydatabase
    
catalogs:
   - name: myhive
     type: hive
     hive-conf-dir: /opt/hive-conf
     hive-version: 2.3.4
```
{% endtab %}
{% endtabs %}

## DDL

DDL创建的Hive表，视图，分区，功能在Flink将很快得到支持。

## DML

 Flink支持DML写入Hive表。请参[阅读写蜂房表中的](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/read_write_hive.html)详细信息

