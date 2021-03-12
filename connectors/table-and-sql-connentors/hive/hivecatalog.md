# HiveCatalog

多年来，在Hadoop生态系统中，Hive Metastore已经发展成为事实上的元数据中心。许多公司在他们的产品中有一个单一的Hive Metastore服务实例来管理他们所有的元数据，无论是Hive元数据还是非Hive元数据，作为真理的源泉。

对于同时拥有Hive和Flink部署的用户，HiveCatalog允许他们使用Hive Metastore来管理Flink的元数据。

对于只有Flink部署的用户来说，HiveCatalog是Flink提供的惟一一个即时可用的持久目录。如果没有持久的目录，使用Flink SQL CREATE DDL的用户必须在每个会话中重复创建元对象\(比如Kafka表\)，这将浪费大量时间。HiveCatalog通过允许用户只创建一次表和其他元对象，并在以后的会话中方便地引用和管理它们，从而填补了这一空白。

## 配置HiveCatalog

### 依赖关系

在Flink中设置HiveCatalog需要与整个Flink- hive集成具有相同的 [依赖](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/#dependencies)关系。

### 配置

在Flink中设置HiveCatalog需要与整个Flink- hive集成相同的 [配置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/#connecting-to-hive)。

## 如何使用HiveCatalog

一旦正确配置，HiveCatalog就可以开箱即用了。用户可以用DDL创建Flink元对象，然后立即看到它们。

HiveCatalog可用于处理两种表:hive兼容的表和泛型表。hive兼容的表是那些以hive兼容的方式存储的表，包括元数据和存储层中的数据。因此，可以从Hive端查询通过Flink创建的与Hive兼容的表。

另一方面，泛型表是特定于Flink的。当使用HiveCatalog创建泛型表时，我们只是使用HMS来持久化元数据。虽然这些表对Hive是可见的，但Hive不太可能理解元数据。因此，在Hive中使用这样的表会导致未定义的行为。

Flink使用属性' `is_generic` '来判断一个表是hive兼容的还是泛型的。当使用HiveCatalog创建表时，默认情况下认为它是通用的。如果想要创建一个hive兼容的表，请确保在表属性中将`is_generic`设置为`false`。

如上所述，泛型表不应该在Hive中使用。在Hive CLI中， 可以通过调用`DESCRIBE FORMATTED`表并检查`is_generic`属性来决定它是否通用。

### 例子

在这里，我们将通过一个简单的示例来掌握如何使用HiveCatalog。

**步骤1：建立Hive Metastore**

运行Hive Metastore

 在这里，我们建立了一个本地Hive Metastore，并将`hive-site.xml`文件放在本地path中`/opt/hive-conf/hive-site.xml`。配置如下：

```markup
<configuration>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value>
      <description>metadata is stored in a MySQL server</description>
   </property>

   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>MySQL JDBC driver class</description>
   </property>

   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>...</value>
      <description>user name for connecting to mysql server</description>
   </property>

   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>...</value>
      <description>password for connecting to mysql server</description>
   </property>

   <property>
       <name>hive.metastore.uris</name>
       <value>thrift://localhost:9083</value>
       <description>IP address (or fully-qualified domain name) and port of the metastore host</description>
   </property>

   <property>
       <name>hive.metastore.schema.verification</name>
       <value>true</value>
   </property>

</configuration>
```

 使用Hive Cli测试与HMS的连接。运行以下命令，我​​们可以看到我们有一个名为`default`的数据库，没有表。

```sql
hive> show databases;
OK
default
Time taken: 0.032 seconds, Fetched: 1 row(s)

hive> show tables;
OK
Time taken: 0.028 seconds, Fetched: 0 row(s)
```

**步骤2：配置Flink群集和SQL CLI**

 在Flink发行版中，将所有Hive依赖项添加到/lib目录中，并修改SQL CLI的sql-cli-defaults.yaml配置文件。如下:

```yaml
execution:
    planner: blink
    type: streaming
    ...
    current-catalog: myhive  # set the HiveCatalog as the current catalog of the session
    current-database: mydatabase
    
catalogs:
   - name: myhive
     type: hive
     hive-conf-dir: /opt/hive-conf  # contains hive-site.xml
     hive-version: 2.3.4
```

**步骤3：建立Kafka集群**

通过本地Kafka 2.3.0集群创建一个名为“test”的Topic，并为Topic生成一些简单的数据，作为名称和年龄的元组

```text
localhost$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>tom,15
>john,21
```

启动Kafka控制台消费者可以看到这些消息。

```text
localhost$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

tom,15
john,21
```

**步骤4：启动SQL Client，并使用Flink SQL DDL创建一个Kafka表**

启动Flink SQL客户端，通过DDL创建一个简单的Kafka 2.3.0表，并验证它的模式。

```sql
Flink SQL> CREATE TABLE mykafka (name String, age Int) WITH (
   'connector.type' = 'kafka',
   'connector.version' = 'universal',
   'connector.topic' = 'test',
   'connector.properties.zookeeper.connect' = 'localhost:2181',
   'connector.properties.bootstrap.servers' = 'localhost:9092',
   'format.type' = 'csv',
   'update-mode' = 'append'
);
[INFO] Table has been created.

Flink SQL> DESCRIBE mykafka;
root
 |-- name: STRING
 |-- age: INT
```

通过Hive Cli验证表对Hive也是可见的，注意表的属性is\_generic=true:

```sql
hive> show tables;
OK
mykafka
Time taken: 0.038 seconds, Fetched: 1 row(s)

hive> describe formatted mykafka;
OK
# col_name            	data_type           	comment


# Detailed Table Information
Database:           	default
Owner:              	null
CreateTime:         	......
LastAccessTime:     	UNKNOWN
Retention:          	0
Location:           	......
Table Type:         	MANAGED_TABLE
Table Parameters:
	flink.connector.properties.bootstrap.servers	localhost:9092
	flink.connector.properties.zookeeper.connect	localhost:2181
	flink.connector.topic	test
	flink.connector.type	kafka
	flink.connector.version	universal
	flink.format.type   	csv
	flink.generic.table.schema.0.data-type	VARCHAR(2147483647)
	flink.generic.table.schema.0.name	name
	flink.generic.table.schema.1.data-type	INT
	flink.generic.table.schema.1.name	age
	flink.update-mode   	append
	is_generic          	true
	transient_lastDdlTime	......

# Storage Information
SerDe Library:      	org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
InputFormat:        	org.apache.hadoop.mapred.TextInputFormat
OutputFormat:       	org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat
Compressed:         	No
Num Buckets:        	-1
Bucket Columns:     	[]
Sort Columns:       	[]
Storage Desc Params:
	serialization.format	1
Time taken: 0.158 seconds, Fetched: 36 row(s)
```

**第5步：运行Flink SQL查询Kakfa表**

在Flink集群\(独立集群或yarn-session集群\)中从Flink SQL客户端运行一个简单的select查询。

```text
Flink SQL> select * from mykafka;
```

生成更多的数据到Kafka Topic中

```text
localhost$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

tom,15
john,21
kitty,30
amy,24
kaiky,18
```

你应该看到Flink在SQL客户端产生的结果，如:

```text
          SQL Query Result (Table)
 Refresh: 1 s    Page: Last of 1     

        name                       age
         tom                        15
        john                        21
       kitty                        30
         amy                        24
       kaiky                        18
```

## 支持的类型

HiveCatalog支持Flink所有通用表类型。

对于HiveCatalog的Hive兼容表，需要将Flink数据类型映射到相应的Hive类型，如下表所示:

| Flink数据类型 | Hive数据类型 |
| :--- | :--- |
| CHAR\(p\) | CHAR\(p\) |
| VARCHAR\(p\) | VARCHAR\(p\) |
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
| TIMESTAMP\(9\) | TIMESTAMP |
| BYTES | BINARY |
| ARRAY&lt;T&gt; | LIST&lt;T&gt; |
| MAP&lt;K, V&gt; | MAP&lt;K, V&gt; |
| ROW | STRUCT |

有关类型映射的注意事项：

* Hive的`CHAR(p)`最大长度为255
* Hive的`VARCHAR(p)`最大长度为65535
* Hive `MAP`仅支持原始键类型，而Flink `MAP`可以是任何数据类型
* `UNION`不支持Hive的类型
* Hive `TIMESTAMP`始终具有9精度，不支持其他精度。另一方面，配置单元UDF可以处理`TIMESTAMP`精度&lt;= 9的值。
* Hive不支持Flink的`TIMESTAMP_WITH_TIME_ZONE`，`TIMESTAMP_WITH_LOCAL_TIME_ZONE`和`MULTISET`
* Flink的`INTERVAL`类型尚不能映射到Hive `INTERVAL`类型

## Scala Shell

{% hint style="info" %}
注意：由于Scala Shell目前不支持blink planner，因此**不**建议在Scala Shell中使用Hive连接器。
{% endhint %}

