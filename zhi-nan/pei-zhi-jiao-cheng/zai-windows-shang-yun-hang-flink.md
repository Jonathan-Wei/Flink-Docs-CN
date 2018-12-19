# 在Windows上运行Flink

如果要在Windows计算机上本地运行Flink，则需要[下载](http://flink.apache.org/downloads.html)并解压缩二进制Flink发布包。之后，可以使用**Windows批处理**文件（`.bat`），或使用**Cygwin**运行Flink Jobmanager。

## 通过Windows bat文件启动

要从_Windows命令行_启动Flink ，请打开命令窗口，导航到Flink的`bin/`并运行`start-cluster.bat`。

{% hint style="info" %}
Java运行时环境的bin文件夹必须包含在窗口的%PATH%变量中。按照本[指南](http://www.java.com/en/download/help/path.xml)将Java添加到%PATH%变量中。
{% endhint %}

```bash
$ cd flink
$ cd bin
$ start-cluster.bat
Starting a local cluster with one JobManager process and one TaskManager process.
You can terminate the processes via CTRL-C in the spawned shell windows.
Web interface by default on http://localhost:8081/.
```

之后，您需要打开第二个终端，通过`flink.bat`来运行作业。

## 通过Cygwin和Unix脚本启动

使用_Cygwin，_您需要启动Cygwin终端，导航到您的Flink目录并运行`start-cluster.sh`脚本：

```bash
$ cd flink
$ bin/start-cluster.sh
Starting cluster.
```

## 通过Git安装Flink

如果是通过git安装Flink，并且使用的是windows git shell，则Cygwin可能会产生类似于以下的故障：

```bash
c:/flink/bin/start-cluster.sh: line 30: $'\r': command not found
```

发生此错误是因为在Windows中运行时，git会自动将UNIX行结尾转换为Windows样式行结尾。问题是Cygwin只能处理UNIX样式的行结尾。解决方案是通过以下三个步骤调整Cygwin设置以处理正确的行结尾：

1. 启动一个Cygwin shell。
2. 输入确定您的主目录

```bash
 cd; pwd
```

这将返回Cygwin根路径下的路径。

1. 使用NotePad，写字板或其他文本编辑器打开`.bash_profile`主目录中的文件并附加以下内容:\(如果文件不存在，则必须创建它）

```bash
export SHELLOPTS
set -o igncr
```

保存文件并打开一个新的bash shell。

