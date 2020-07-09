# JDBC

此连接器提供一个用于将数据写入JDBC数据库的接收器。

要使用它，请将以下依赖项添加到项目中（以及JDBC驱动程序）：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-jdbc_2.11</artifactId>
  <version>1.11.0</version>
</dependency>
```

注意，流连接器目前不是二进制发行版的一部分。在[`这里`](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/project-configuration.html)查看如何链接它们以执行集群。

创建的JDBC接收器至少提供一次保证。使用upsert语句或幂等更新可以有效地实现一次。

用法示例：

```text
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env
        .fromElements(...)
        .addSink(JdbcSink.sink(
                "insert into books (id, title, author, price, qty) values (?,?,?,?,?)",
                (ps, t) -> {
                    ps.setInt(1, t.id);
                    ps.setString(2, t.title);
                    ps.setString(3, t.author);
                    ps.setDouble(4, t.price);
                    ps.setInt(5, t.qty);
                },
                new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                        .withUrl(getDbMetadata().getUrl())
                        .withDriverName(getDbMetadata().getDriverClass())
                        .build()));
env.execute();
```

有关更多详细信息，请参阅[`API文档`](https://ci.apache.org/projects/flink/flink-docs-release-1.11/api/java/org/apache/flink/connector/jdbc/JdbcSink.html)。

