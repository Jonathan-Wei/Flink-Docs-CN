# CLI

Flink提供了一个命令行界面（CLI），用于运行打包为JAR文件的程序，并控制它们的执行。CLI是任何Flink设置的一部分，在本地单节点设置和分布式设置中都可用。它位于`/bin/flink`下，默认情况下连接到从同一安装目录启动的正在运行的Flink Master（JobManager）。

使用命令行界面的先决条件是Flink Master（JobManager）已启动（通过`/bin/start cluster.sh`）或有可用的Yarn环境。

命令行可用于

* 提交要执行的Job，
* 取消正在执行的Job，
* 提供有关Job的信息，
* 列出正在运行和正在等待的Job，
* 触发并处理保存点

 使用命令行界面的先决条件是（已通过`<flink-home>/bin/start-cluster.sh`）启动了Flink主服务器（JobManager ），或者可以使用其他部署目标，例如YARN或Kubernetes。

## 部署目标

Flink有定义可用部署目标的执行器的概念。 您可以在`bin/flink --help`的输出中看到可用的执行程序

```text
Options for executor mode:
   -D <property=value>   Generic configuration options for
                         execution/deployment and for the configured executor.
                         The available options can be found at
                         https://ci.apache.org/projects/flink/flink-docs-stabl
                         e/ops/config.html
   -e,--executor <arg>   The name of the executor to be used for executing the
                         given job, which is equivalent to the
                         "execution.target" config option. The currently
                         available executors are: "remote", "local",
                         "kubernetes-session", "yarn-per-job", "yarn-session".
```

当运行`bin/flink`操作之一时，使用`——executor`选项指定执行程序。

## 例子

### 作业提交实例

{% tabs %}
{% tab title="Java" %}
* 运行没有参数的示例程序：

  ```text
  ./bin/flink run ./examples/batch/WordCount.jar
  ```

* 使用输入和结果文件的参数运行示例程序：

  ```text
  ./bin/flink run ./examples/batch/WordCount.jar \
                       --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
  ```

* 运行带有并行性16的示例程序以及输入和结果文件的参数：

  ```text
  ./bin/flink run -p 16 ./examples/batch/WordCount.jar \
                       --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
  ```

* 运行禁用flink日志输出的示例程序：

  ```text
      ./bin/flink run -q ./examples/batch/WordCount.jar
  ```

* 以分离模式运行示例程序：

  ```text
      ./bin/flink run -d ./examples/batch/WordCount.jar
  ```

* 在特定的JobManager上运行示例程序：

  ```text
  ./bin/flink run -m myJMHost:8081 \
                         ./examples/batch/WordCount.jar \
                         --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
  ```

* 以特定类作为入口点运行示例程序：

  ```text
  ./bin/flink run -c org.apache.flink.examples.java.wordcount.WordCount \
                         ./examples/batch/WordCount.jar \
                         --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
  ```

* 使用具有2个TaskManagers 的[每作业YARN集群](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/yarn_setup.html#run-a-single-flink-job-on-hadoop-yarn)运行示例程序：

  ```text
  ./bin/flink run -m yarn-cluster -yn 2 \
                         ./examples/batch/WordCount.jar \
                         --input hdfs:///user/hamlet.txt --output hdfs:///user/wordcount_out
  ```
{% endtab %}

{% tab title="Python" %}
{% hint style="info" %}
注意： 当通过提交Python作业时`flink run`，Flink将运行命令“ python”。请运行以下命令，以确认当前环境中的命令“ python”指向指定的Python版本3.5、3.6或3.7：
{% endhint %}

```text
$ python --version
# the version printed here must be 3.5, 3.6 or 3.7
```

* 运行Python Table程序：

  ```text
  ./bin/flink run -py examples/python/table/batch/word_count.py
  ```

* 使用pyFiles运行Python Table程序：

  ```text
  ./bin/flink run -py examples/python/table/batch/word_count.py \
                          -pyfs file:///user.txt,hdfs:///$namenode_address/username.txt
  ```

* 使用JAR文件运行Python Table程序：

  ```text
  ./bin/flink run -py examples/python/table/batch/word_count.py -j <jarFile>
  ```

* 使用pyFiles和pyModule运行Python Table程序：

  ```text
  ./bin/flink run -pym batch.word_count -pyfs examples/python/table/batch
  ```

* 运行具有并行性的Python Table程序16：

  ```text
  ./bin/flink run -p 16 -py examples/python/table/batch/word_count.py
  ```

* 在禁用flink日志输出的情况下运行Python Table程序：

  ```text
  ./bin/flink run -q -py examples/python/table/batch/word_count.py
  ```

* 以分离模式运行Python Table程序：

  ```text
  ./bin/flink run -d -py examples/python/table/batch/word_count.py
  ```

* 在特定的JobManager上运行Python Table程序：

  ```text
  ./bin/flink run -m myJMHost:8081 \
                         -py examples/python/table/batch/word_count.py
  ```

* 使用具有2个TaskManager 的 [per-job ](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/yarn_setup.html#run-a-single-flink-job-on-hadoop-yarn)[YARN集群](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/yarn_setup.html#run-a-single-flink-job-on-hadoop-yarn)运行Python Table程序：
* ```text
  ./bin/flink run -m yarn-cluster \
                         -py examples/python/table/batch/word_count.py
  ```
{% endtab %}
{% endtabs %}

### 作业管理实例

以下是关于如何在CLI中管理作业的示例。

{% tabs %}
{% tab title="Java" %}
* 将WordCount示例程序的优化执行计划展示为JSON：

  ```text
  ./bin/flink info ./examples/batch/WordCount.jar \
                          --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
  ```

* 列出计划和正在运行的作业（包括其JobID）：

  ```text
  ./bin/flink list
  ```

* 列出预定作业（包括其作业ID）：

  ```text
  ./bin/flink list -s
  ```

* 列出正在运行的作业（包括其作业ID）：

  ```text
  ./bin/flink list -r
  ```

* 列出所有现有工作（包括其工作ID）：

  ```text
  ./bin/flink list -a
  ```

* 列出在Flink YARN会话中运行Flink作业：

  ```text
  ./bin/flink list -m yarn-cluster -yid <yarnApplicationID> -r
  ```

* 取消工作：

  ```text
  ./bin/flink cancel <jobID>
  ```

* 使用保存点取消作业：

  ```text
  ./bin/flink cancel -s [targetDirectory] <jobID>
  ```

* 用保存点正常停止作业（仅用于流作业）：

  ```text
  ./bin/flink stop [-p targetDirectory] [-d] <jobID>
  ```
{% endtab %}
{% endtabs %}

**注意**：取消和停止（流媒体）作业的区别如下：

在`cancel`调用中，作业中的操作符立即接收`cancel()`方法调用，以尽快取消它们。如果操作符在`cancel`调用后没有停止，Flink将开始周期性地中断线程，直到它停止。

`“stop”`调用是停止正在运行的流作业的更优雅的方法。Stop只适用于使用实现`StoppableFunction`接口的源的作业。当用户请求停止作业时，所有源都将收到`stop()`方法调用。在所有资源正常关闭之前，该工作将继续运行。这允许作业完成对所有飞行数据的处理。

### 保存点

[保存点](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html)通过命令行客户端控制：

#### **触发保存点**

```text
./bin/flink savepoint <jobId> [savepointDirectory]
```

这将触发具有ID的作业的保存点`jobId`，并返回创建的保存点的路径。你需要此路径来还原和部署保存点。

此外，可以选择指定目标文件系统目录以存储保存点。该目录需要由JobManager访问。

如果未指定目标目录，则需要[配置默认目录](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#configuration)。否则，触发保存点将失败。

**使用YARN触发保存点**

```text
./bin/flink savepoint <jobId> [savepointDirectory] -yid <yarnAppId>
```

这将触发具有ID `jobId`和YARN应用程序ID 的作业的保存点`yarnAppId`，并返回创建的保存点的路径。

其他所有内容与上面**触发保存点**部分中描述的相同。

**停止**

 使用`stop`可以通过保存点正常停止正在运行的流作业。

```text
./bin/flink stop [-p targetDirectory] [-d] <jobID>
```

“**stop**”调用是停止正在运行的流作业的一种更优雅的方式，因为“**stop**”信号从源流到接收流。当用户请求停止作业时，所有源都将被请求发送最后一个触发保存点的检查点屏障，在成功完成该保存点之后，它们将通过调用cancel\(\)方法来完成。如果指定了-d标志，那么将在最后一个检查点屏障之前发出max\_水位线。这将导致所有已注册的事件时间计时器启动，从而清除等待特定水位线的任何状态，例如windows。作业将继续运行，直到所有源都正确关闭。这允许作业完成所有飞行数据的处理。

**取消保存点（不建议使用）**

你可以自动触发保存点并取消作业。

```text
./bin/flink cancel -s [savepointDirectory] <jobID>
```

如果未配置SavePoint目录，则需要为Flink安装配置默认保存点目录（请参阅[保存点](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#configuration)）。

只有SavePoint成功，才会取消该作业。

{% hint style="danger" %}
注意：不建议取消具有SavePoint的作业。使用“停止”代替。
{% endhint %}

**恢复保存点**

```text
./bin/flink run -s <savepointPath> ...
```

run命令具有提交作业的保存点标志，该标志将从保存点恢复其状态。保存点触发命令返回保存点路径。

默认情况下，我们尝试将所有保存点状态与提交的作业匹配。如果希望允许跳过无法通过新作业恢复的保存点状态，可以设置allowNonRestoredState标志。如果从程序中删除了一个操作符，而该操作符在触发保存点时是程序的一部分，并且仍然希望使用保存点，那么需要允许这样做。

```text
./bin/flink run -s <savepointPath> -n ...
```

如果程序删除了属于保存点的运算符，这将非常有用。

**配置保存点**

```text
./bin/flink savepoint -d <savepointPath>
```

在给定路径处置保存点。保存点 触发命令返回保存点路径。

如果使用自定义状态实例（例如自定义还原状态或RocksDB状态），则必须指定触发保存点的程序JAR的路径，以便使用用户代码类加载器处置保存点：

```text
./bin/flink savepoint -d <savepointPath> -j <jarFile>
```

否则，你会遇到一个`ClassNotFoundException`。

## 用法

命令行语法如下：

```text
./flink <ACTION> [OPTIONS] [ARGUMENTS]

The following actions are available:

Action "run" compiles and runs a program.

  Syntax: run [OPTIONS] <jar-file> <arguments>
  "run" action options:
     -c,--class <classname>               Class with the program entry point
                                          ("main()" method). Only needed if the
                                          JAR file does not specify the class in
                                          its manifest.
     -C,--classpath <url>                 Adds a URL to each user code
                                          classloader  on all nodes in the
                                          cluster. The paths must specify a
                                          protocol (e.g. file://) and be
                                          accessible on all nodes (e.g. by means
                                          of a NFS share). You can use this
                                          option multiple times for specifying
                                          more than one URL. The protocol must
                                          be supported by the {@link
                                          java.net.URLClassLoader}.
     -d,--detached                        If present, runs the job in detached
                                          mode
     -n,--allowNonRestoredState           Allow to skip savepoint state that
                                          cannot be restored. You need to allow
                                          this if you removed an operator from
                                          your program that was part of the
                                          program when the savepoint was
                                          triggered.
     -p,--parallelism <parallelism>       The parallelism with which to run the
                                          program. Optional flag to override the
                                          default value specified in the
                                          configuration.
     -py,--python <pythonFile>            Python script with the program entry
                                          point. The dependent resources can be
                                          configured with the `--pyFiles`
                                          option.
     -pyarch,--pyArchives <arg>           Add python archive files for job. The
                                          archive files will be extracted to the
                                          working directory of python UDF
                                          worker. Currently only zip-format is
                                          supported. For each archive file, a
                                          target directory be specified. If the
                                          target directory name is specified,
                                          the archive file will be extracted to
                                          a name can directory with the
                                          specified name. Otherwise, the archive
                                          file will be extracted to a directory
                                          with the same name of the archive
                                          file. The files uploaded via this
                                          option are accessible via relative
                                          path. '#' could be used as the
                                          separator of the archive file path and
                                          the target directory name. Comma (',')
                                          could be used as the separator to
                                          specify multiple archive files. This
                                          option can be used to upload the
                                          virtual environment, the data files
                                          used in Python UDF (e.g.: --pyArchives
                                          file:///tmp/py37.zip,file:///tmp/data.
                                          zip#data --pyExecutable
                                          py37.zip/py37/bin/python). The data
                                          files could be accessed in Python UDF,
                                          e.g.: f = open('data/data.txt', 'r').
     -pyexec,--pyExecutable <arg>         Specify the path of the python
                                          interpreter used to execute the python
                                          UDF worker (e.g.: --pyExecutable
                                          /usr/local/bin/python3). The python
                                          UDF worker depends on a specified Python
                                          version 3.5, 3.6 or 3.7, Apache Beam
                                          (version == 2.15.0), Pip (version >= 7.1.0)
                                          and SetupTools (version >= 37.0.0).
                                          Please ensure that the specified environment
                                          meets the above requirements.
     -pyfs,--pyFiles <pythonFiles>        Attach custom python files for job.
                                          These files will be added to the
                                          PYTHONPATH of both the local client
                                          and the remote python UDF worker. The
                                          standard python resource file suffixes
                                          such as .py/.egg/.zip or directory are
                                          all supported. Comma (',') could be
                                          used as the separator to specify
                                          multiple files (e.g.: --pyFiles
                                          file:///tmp/myresource.zip,hdfs:///$na
                                          menode_address/myresource2.zip).
     -pym,--pyModule <pythonModule>       Python module with the program entry
                                          point. This option must be used in
                                          conjunction with `--pyFiles`.
     -pyreq,--pyRequirements <arg>        Specify a requirements.txt file which
                                          defines the third-party dependencies.
                                          These dependencies will be installed
                                          and added to the PYTHONPATH of the
                                          python UDF worker. A directory which
                                          contains the installation packages of
                                          these dependencies could be specified
                                          optionally. Use '#' as the separator
                                          if the optional parameter exists
                                          (e.g.: --pyRequirements
                                          file:///tmp/requirements.txt#file:///t
                                          mp/cached_dir).
     -s,--fromSavepoint <savepointPath>   Path to a savepoint to restore the job
                                          from (for example
                                          hdfs:///flink/savepoint-1537).
     -sae,--shutdownOnAttachedExit        If the job is submitted in attached
                                          mode, perform a best-effort cluster
                                          shutdown when the CLI is terminated
                                          abruptly, e.g., in response to a user
                                          interrupt, such as typing Ctrl + C.
  Options for yarn-cluster mode:
     -d,--detached                        If present, runs the job in detached
                                          mode
     -m,--jobmanager <arg>                Address of the JobManager (master) to
                                          which to connect. Use this flag to
                                          connect to a different JobManager than
                                          the one specified in the
                                          configuration.
     -yat,--yarnapplicationType <arg>     Set a custom application type for the
                                          application on YARN
     -yD <property=value>                 use value for given property
     -yd,--yarndetached                   If present, runs the job in detached
                                          mode (deprecated; use non-YARN
                                          specific option instead)
     -yh,--yarnhelp                       Help for the Yarn session CLI.
     -yid,--yarnapplicationId <arg>       Attach to running YARN session
     -yj,--yarnjar <arg>                  Path to Flink jar file
     -yjm,--yarnjobManagerMemory <arg>    Memory for JobManager Container with
                                          optional unit (default: MB)
     -ynl,--yarnnodeLabel <arg>           Specify YARN node label for the YARN
                                          application
     -ynm,--yarnname <arg>                Set a custom name for the application
                                          on YARN
     -yq,--yarnquery                      Display available YARN resources
                                          (memory, cores)
     -yqu,--yarnqueue <arg>               Specify YARN queue.
     -ys,--yarnslots <arg>                Number of slots per TaskManager
     -yt,--yarnship <arg>                 Ship files in the specified directory
                                          (t for transfer)
     -ytm,--yarntaskManagerMemory <arg>   Memory per TaskManager Container with
                                          optional unit (default: MB)
     -yz,--yarnzookeeperNamespace <arg>   Namespace to create the Zookeeper
                                          sub-paths for high availability mode
     -z,--zookeeperNamespace <arg>        Namespace to create the Zookeeper
                                          sub-paths for high availability mode

  Options for executor mode:
     -D <property=value>   Generic configuration options for
                           execution/deployment and for the configured executor.
                           The available options can be found at
                           https://ci.apache.org/projects/flink/flink-docs-stabl
                           e/ops/config.html
     -e,--executor <arg>   The name of the executor to be used for executing the
                           given job, which is equivalent to the
                           "execution.target" config option. The currently
                           available executors are: "remote", "local",
                           "kubernetes-session", "yarn-per-job", "yarn-session".

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "info" shows the optimized execution plan of the program (JSON).

  Syntax: info [OPTIONS] <jar-file> <arguments>
  "info" action options:
     -c,--class <classname>           Class with the program entry point
                                      ("main()" method). Only needed if the JAR
                                      file does not specify the class in its
                                      manifest.
     -p,--parallelism <parallelism>   The parallelism with which to run the
                                      program. Optional flag to override the
                                      default value specified in the
                                      configuration.


Action "list" lists running and scheduled programs.

  Syntax: list [OPTIONS]
  "list" action options:
     -a,--all         Show all programs and their JobIDs
     -r,--running     Show only running programs and their JobIDs
     -s,--scheduled   Show only scheduled programs and their JobIDs
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for executor mode:
     -D <property=value>   Generic configuration options for
                           execution/deployment and for the configured executor.
                           The available options can be found at
                           https://ci.apache.org/projects/flink/flink-docs-stabl
                           e/ops/config.html
     -e,--executor <arg>   The name of the executor to be used for executing the
                           given job, which is equivalent to the
                           "execution.target" config option. The currently
                           available executors are: "remote", "local",
                           "kubernetes-session", "yarn-per-job", "yarn-session".

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "stop" stops a running program with a savepoint (streaming jobs only).

  Syntax: stop [OPTIONS] <Job ID>
  "stop" action options:
     -d,--drain                           Send MAX_WATERMARK before taking the
                                          savepoint and stopping the pipelne.
     -p,--savepointPath <savepointPath>   Path to the savepoint (for example
                                          hdfs:///flink/savepoint-1537). If no
                                          directory is specified, the configured
                                          default will be used
                                          ("state.savepoints.dir").
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for executor mode:
     -D <property=value>   Generic configuration options for
                           execution/deployment and for the configured executor.
                           The available options can be found at
                           https://ci.apache.org/projects/flink/flink-docs-stabl
                           e/ops/config.html
     -e,--executor <arg>   The name of the executor to be used for executing the
                           given job, which is equivalent to the
                           "execution.target" config option. The currently
                           available executors are: "remote", "local",
                           "kubernetes-session", "yarn-per-job", "yarn-session".

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "cancel" cancels a running program.

  Syntax: cancel [OPTIONS] <Job ID>
  "cancel" action options:
     -s,--withSavepoint <targetDirectory>   **DEPRECATION WARNING**: Cancelling
                                            a job with savepoint is deprecated.
                                            Use "stop" instead.
                                            Trigger savepoint and cancel job.
                                            The target directory is optional. If
                                            no directory is specified, the
                                            configured default directory
                                            (state.savepoints.dir) is used.
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for executor mode:
     -D <property=value>   Generic configuration options for
                           execution/deployment and for the configured executor.
                           The available options can be found at
                           https://ci.apache.org/projects/flink/flink-docs-stabl
                           e/ops/config.html
     -e,--executor <arg>   The name of the executor to be used for executing the
                           given job, which is equivalent to the
                           "execution.target" config option. The currently
                           available executors are: "remote", "local",
                           "kubernetes-session", "yarn-per-job", "yarn-session".

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "savepoint" triggers savepoints for a running job or disposes existing ones.

  Syntax: savepoint [OPTIONS] <Job ID> [<target directory>]
  "savepoint" action options:
     -d,--dispose <arg>       Path of savepoint to dispose.
     -j,--jarfile <jarfile>   Flink program JAR file.
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for executor mode:
     -D <property=value>   Generic configuration options for
                           execution/deployment and for the configured executor.
                           The available options can be found at
                           https://ci.apache.org/projects/flink/flink-docs-stabl
                           e/ops/config.html
     -e,--executor <arg>   The name of the executor to be used for executing the
                           given job, which is equivalent to the
                           "execution.target" config option. The currently
                           available executors are: "remote", "local",
                           "kubernetes-session", "yarn-per-job", "yarn-session".

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode
```

