# Hadoop集成

## 使用Hadoop类路径配置Flink

Flink将使用环境变量HADOOP\_CLASSPATH来扩充启动Flink组件（如Client，JobManager或TaskManager）时使用的类路径。大多数Hadoop发行版和云环境在默认情况下不会设置此变量，因此，如果Flink应该选择Hadoop类路径，则必须在运行Flink组件的所有计算机上导出环境变量。

在运行yarn时，这通常不是问题，因为在yarn中运行的组件将从Hadoop类路径开始，但在向yarn提交作业时，Hadoop依赖项可能必须位于类路径中。为此，通常只需

```text
export HADOOP_CLASSPATH=`hadoop classpath`
```

在shell中。请注意，`hadoop`是hadoop的二进制文件，`classpath`是一个参数，用于打印已配置的Hadoop类路径。

