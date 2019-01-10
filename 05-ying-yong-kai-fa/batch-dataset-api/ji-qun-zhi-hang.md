# 集群执行

Flink程序可以在许多机器的集群上分布式运行。将程序发送到集群以执行有两种方法：

## 命令行界面

命令行界面允许您将打包程序（JAR）提交到群集（或单机设置）。

有关详细信息，请参阅[命令行界面](https://ci.apache.org/projects/flink/flink-docs-master/ops/cli.html)文档

## 远程环境

远程环境允许您直接在集群上执行Flink Java程序。远程环境指向要在其上执行程序的群集。

### Maven依赖

如果您将程序开发为Maven项目，则必须使用此依赖项添加flink-clients模块:

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-clients_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

示例：

以下是`RemoteEnvironment`的使用说明：

```java
public static void main(String[] args) throws Exception {
    ExecutionEnvironment env = ExecutionEnvironment
        .createRemoteEnvironment("flink-master", 8081, "/home/user/udfs.jar");

    DataSet<String> data = env.readTextFile("hdfs://path/to/file");

    data
        .filter(new FilterFunction<String>() {
            public boolean filter(String value) {
                return value.startsWith("http://");
            }
        })
        .writeAsText("hdfs://path/to/result");

    env.execute();
}

```

{% hint style="info" %}
注意：该程序包含自定义用户代码，因此需要一个带有附加代码类的JAR文件。远程环境的构造函数接受到JAR文件的路径。
{% endhint %}

