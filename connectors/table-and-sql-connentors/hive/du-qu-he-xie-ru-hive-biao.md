# 读取和写入Hive表

使用`HiveCatalog`和Flink连接到Hive，Flink可以从Hive数据读取和写入，作为Hive批处理引擎的替代。请务必按照说明在您的应用程序中包含正确的[依赖项](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/hive/#depedencies)。

## 从Hive中读取

假设Hive在其`default`数据库中包含一个表，包含若干行的已命名人员。

```sql
hive> show databases;
OK
default
Time taken: 0.841 seconds, Fetched: 1 row(s)

hive> show tables;
OK
Time taken: 0.087 seconds

hive> CREATE TABLE mytable(name string, value double);
OK
Time taken: 0.127 seconds

hive> SELECT * FROM mytable;
OK
Tom   4.72
John  8.0
Tom   24.2
Bob   3.14
Bob   4.72
Tom   34.9
Mary  4.79
Tiff  2.72
Bill  4.33
Mary  77.7
Time taken: 0.097 seconds, Fetched: 10 row(s)
```

准备好数据后，您可以连接到Hive [连接到现有的Hive](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/hive/#connecting-to-hive)并开始查询。

```sql
Flink SQL> show catalogs;
myhive
default_catalog

# ------ Set the current catalog to be 'myhive' catalog if you haven't set it in the yaml file ------

Flink SQL> use catalog myhive;

# ------ See all registered database in catalog 'mytable' ------

Flink SQL> show databases;
default

# ------ See the previously registered table 'mytable' ------

Flink SQL> show tables;
mytable

# ------ The table schema that Flink sees is the same that we created in Hive, two columns - name as string and value as double ------ 
Flink SQL> describe mytable;
root
 |-- name: name
 |-- type: STRING
 |-- name: value
 |-- type: DOUBLE


Flink SQL> SELECT * FROM mytable;

   name      value
__________ __________

    Tom      4.72
    John     8.0
    Tom      24.2
    Bob      3.14
    Bob      4.72
    Tom      34.9
    Mary     4.79
    Tiff     2.72
    Bill     4.33
    Mary     77.7
```

## 写入Hive

同样，可以使用`INSERT INTO`子句将数据写入Hive。

```sql
Flink SQL> INSERT INTO mytable (name, value) VALUES ('Tom', 4.72);
```

### 限制

以下是Hive连接器的主要限制列表。我们正在积极努力缩小这些差距。

1. 不支持INSERT OVERWRITE。
2. 不支持插入分区表。
3. 不支持ACID表。
4. 不支持分段表。
5. 某些数据类型不受支持。请参阅[限制](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/hive/#limitations)以获取详细信息
6. 仅测试了有限数量的表存储格式，即文本，SequenceFile，ORC和Parquet。
7. 视图不受支持。

