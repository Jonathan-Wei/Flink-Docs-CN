# 总览

Flink提供了一组表格式，可与表连接器一起使用。表格式是一种存储格式，定义了如何将二进制数据映射到表列上。

Flink支持以下格式：

| 格式 | 支持的连接 |
| :--- | :--- |
| [CSV](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/csv.html) | [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/kafka.html), [Filesystem](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/filesystem.html) |
| [JSON](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/json.html) | [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/kafka.html), [Filesystem](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/filesystem.html), [Elasticsearch](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/elasticsearch.html) |
| [Apache Avro](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/avro.html) | [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/kafka.html), [Filesystem](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/filesystem.html) |
| [Debezium CDC](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/debezium.html) | [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/kafka.html) |
| [Canal CDC](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/canal.html) | [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/kafka.html) |
| [Apache Parquet](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/parquet.html) | [Filesystem](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/filesystem.html) |
| [Apache ORC](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/formats/orc.html) | [Filesystem](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/connectors/filesystem.html) |

