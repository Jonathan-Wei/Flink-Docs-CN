# 在scala shell中使用Hive连接器

[Flink Scala Shell](https://ci.apache.org/projects/flink/flink-docs-release-1.9/ops/scala_shell.html)是一种尝试flink方便快捷的方式。你也可以在scala shell中使用配置单元，而不是在pom文件中指定配置单元依赖项，打包程序并通过flink run命令提交它。为了在scala shell中使用hive连接器，需要将以下[hive连接器依赖项](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/hive/#depedencies)放在flink dist的lib文件夹下。

* flink-connector-hive\_{scala\_version}-{flink.version}.jar
* flink-hadoop-compatibility\_{scala\_version}-{flink.version}.jar
* flink-shaded-hadoop-2-uber-{hadoop.version}-{flink-shaded.version}.jar
* hive-exec-2.x.jar \(for Hive 1.x, you need to copy hive-exec-1.x.jar, hive-metastore-1.x.jar, libfb303-0.9.2.jar and libthrift-0.9.2.jar\)

然后你可以在scala shell中使用hive连接器，如下所示：

```scala
Scala-Flink> import org.apache.flink.table.catalog.hive.HiveCatalog
Scala-Flink> val hiveCatalog = new HiveCatalog("hive", "default", "<Replace it with HIVE_CONF_DIR>", "2.3.4");
Scala-Flink> btenv.registerCatalog("hive", hiveCatalog)
Scala-Flink> btenv.useCatalog("hive")
Scala-Flink> btenv.listTables
Scala-Flink> btenv.sqlQuery("<sql query>").toDataSet[Row].print()
```

