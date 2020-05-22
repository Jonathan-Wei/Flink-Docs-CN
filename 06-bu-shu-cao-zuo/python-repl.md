# Python REPL

Flink附带了一个集成的交互式Python Shell。它可以在本地设置中使用，也可以在集群设置中使用。 有关如何设置本地Flink的更多信息，请参见[本地设置页面](https://ci.apache.org/projects/flink/flink-docs-release-1.10/getting-started/tutorials/local_setup.html)。也可以[从源代码构建本地设置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html)。

{% hint style="info" %}
 注意：Python Shell将运行命令“ python”。请参考Python Table API [安装指南](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/python/installation.html)，了解如何设置Python执行环境。
{% endhint %}

要将Shell与的Flink群集集成使用，只需安装PyFlink和PyPi，并直接执行Shell：

```text
# install PyFlink
$ python -m pip install apache-flink
# execute the shell
$ pyflink-shell.sh local
```

要在群集上运行Shell，请参阅下面的“设置”部分。

## 用法

Shell当前仅支持Table API。启动后，表环境会自动进行预绑定。使用“ bt\_env”和“ st\_env”分别访问BatchTableEnvironment和StreamTableEnvironment。

### Table API

下面是的一个简单Python Shell程序示例：

{% tabs %}
{% tab title="流" %}
```python
>>> import tempfile
>>> import os
>>> import shutil
>>> sink_path = tempfile.gettempdir() + '/streaming.csv'
>>> if os.path.exists(sink_path):
...     if os.path.isfile(sink_path):
...         os.remove(sink_path)
...     else:
...         shutil.rmtree(sink_path)
>>> s_env.set_parallelism(1)
>>> t = st_env.from_elements([(1, 'hi', 'hello'), (2, 'hi', 'hello')], ['a', 'b', 'c'])
>>> st_env.connect(FileSystem().path(sink_path))\
...     .with_format(OldCsv()
...         .field_delimiter(',')
...         .field("a", DataTypes.BIGINT())
...         .field("b", DataTypes.STRING())
...         .field("c", DataTypes.STRING()))\
...     .with_schema(Schema()
...         .field("a", DataTypes.BIGINT())
...         .field("b", DataTypes.STRING())
...         .field("c", DataTypes.STRING()))\
...     .register_table_sink("stream_sink")
>>> t.select("a + 1, b, c")\
...     .insert_into("stream_sink")
>>> st_env.execute("stream_job")
>>> # If the job runs in local mode, you can exec following code in Python shell to see the result:
>>> with open(sink_path, 'r') as f:
...     print(f.read())
```
{% endtab %}

{% tab title="批量" %}
```python
>>> import tempfile
>>> import os
>>> import shutil
>>> sink_path = tempfile.gettempdir() + '/batch.csv'
>>> if os.path.exists(sink_path):
...     if os.path.isfile(sink_path):
...         os.remove(sink_path)
...     else:
...         shutil.rmtree(sink_path)
>>> b_env.set_parallelism(1)
>>> t = bt_env.from_elements([(1, 'hi', 'hello'), (2, 'hi', 'hello')], ['a', 'b', 'c'])
>>> bt_env.connect(FileSystem().path(sink_path))\
...     .with_format(OldCsv()
...         .field_delimiter(',')
...         .field("a", DataTypes.BIGINT())
...         .field("b", DataTypes.STRING())
...         .field("c", DataTypes.STRING()))\
...     .with_schema(Schema()
...         .field("a", DataTypes.BIGINT())
...         .field("b", DataTypes.STRING())
...         .field("c", DataTypes.STRING()))\
...     .register_table_sink("batch_sink")
>>> t.select("a + 1, b, c")\
...     .insert_into("batch_sink")
>>> bt_env.execute("batch_job")
>>> # If the job runs in local mode, you can exec following code in Python shell to see the result:
>>> with open(sink_path, 'r') as f:
...     print(f.read())
```
{% endtab %}
{% endtabs %}

## 设置

要大致了解Python Shell提供的选项，请使用

```text
pyflink-shell.sh --help
```

### 本地

要将Shell与集成的Flink集群一起使用，只需执行以下命令：

```text
pyflink-shell.sh local
```

### 远程

 要在运行的集群中使用它，请使用关键字`remote`启动Python shell，并使用以下命令提供JobManager的主机和端口:

```text
pyflink-shell.sh remote <hostname> <portnumber>
```

### Yarn Python Shell集群

### Yarn Session

如果之前是使用Flink Yarn Session部署了Flink集群，则Python Shell可以使用以下命令与其连接：

```text
pyflink-shell.sh yarn
```

## 完整参考

```text
Flink Python Shell
Usage: pyflink-shell.sh [local|remote|yarn] [options] <args>...

Command: local [options]
Starts Flink Python shell with a local Flink cluster
usage:
     -h,--help   Show the help message with descriptions of all options.
Command: remote [options] <host> <port>
Starts Flink Python shell connecting to a remote cluster
  <host>
        Remote host name as string
  <port>
        Remote port as integer

usage:
     -h,--help   Show the help message with descriptions of all options.
Command: yarn [options]
Starts Flink Python shell connecting to a yarn cluster
usage:
     -h,--help                       Show the help message with descriptions of
                                     all options.
     -jm,--jobManagerMemory <arg>    Memory for JobManager Container with
                                     optional unit (default: MB)
     -nm,--name <arg>                Set a custom name for the application on
                                     YARN
     -qu,--queue <arg>               Specify YARN queue.
     -s,--slots <arg>                Number of slots per TaskManager
     -tm,--taskManagerMemory <arg>   Memory per TaskManager Container with
                                     optional unit (default: MB)
-h | --help
      Prints this usage text
```

