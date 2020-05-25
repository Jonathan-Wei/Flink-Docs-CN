# 常用配置

Apache Flink提供了适用于所有文件系统实现的几种标准配置设置。

## 默认文件系统

如果文件路径未明确指定文件系统方案（和权限），则使用默认方案（和权限）。

```text
fs.default-scheme: <default-fs>
```

例如，如果默认文件系统配置为`fs.default-scheme:hdfs://localhost:9000/`，则文件路径 `/user/hugo/in.txt`解释为`hdfs://localhost:9000/user/hugo/in.txt`。

## 连接限制

可以限制文件系统可以同时打开的连接总数，当文件系统无法同时处理大量并发的读/写或打开连接时，这很有用。

例如，带有少量RPC处理程序的小型HDFS群集有时可能会被试图在检查点期间建立许多连接的大型Flink作业淹没。

要限制特定文件系统的连接，请将以下配置添加到Flink配置中。要限制的文件系统由其方案标识。

```text
fs.<scheme>.limit.total: (number, 0/-1 mean no limit)
fs.<scheme>.limit.input: (number, 0/-1 mean no limit)
fs.<scheme>.limit.output: (number, 0/-1 mean no limit)
fs.<scheme>.limit.timeout: (milliseconds, 0 means infinite)
fs.<scheme>.limit.stream-timeout: (milliseconds, 0 means infinite)
```

可以分别限制输入/输出连接（流）的数量（`fs.<scheme>.limit.input`和`fs.<scheme>.limit.output`），也可以限制并发流的总数（`fs.<scheme>.limit.total`）。如果文件系统尝试打开更多流，则该操作将阻塞直到某些流关闭。如果流的打开花费的时间大于`fs.<scheme>.limit.timeout`，则流打开失败。

为防止不活动的流占用整个池（防止打开新连接），您可以添加不活动的超时，如果它们在至少一定的时间内未读写任何字节，则将其强制关闭`fs.<scheme>.limit.stream-timeout`。

在每个TaskManager /文件系统的基础上限制执行。由于文件系统的创建是按方案和权限进行的，因此不同的权限具有独立的连接池。例如，`hdfs://myhdfs:50010/`并且`hdfs://anotherhdfs:4399/`将具有单独的池。

