# 常见问题

本页介绍了针对PyFlink用户的一些常见问题的解决方案。

## 准备Python虚拟环境

 可以下载一个[便利脚本](https://ci.apache.org/projects/flink/flink-docs-release-1.10/downloads/setup-pyflink-virtual-env.sh)来准备一个Python虚拟环境zip，该zip可以在Mac OS和大多数Linux发行版中使用。可以指定version参数来生成对应的PyFlink版本所需的Python虚拟环境，否则将安装最新版本。

```text
$ setup-pyflink-virtual-env.sh 1.10.0
```

## 在Python虚拟环境中执行PyFlink作业

 如上一节所述，在设置了[python虚拟环境](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/python/common_questions.html#preparing-python-virtual-environment)之后，应该在执行PyFlink作业之前激活环境。

**本地**

```text
# activate the conda python virtual environment
$ source venv/bin/activate
$ python xxx.py
```

**集群**

```text
$ # specify the Python virtual environment
$ table_env.add_python_archive("venv.zip")
$ # specify the path of the python interpreter which is used to execute the python UDF workers
$ table_env.get_config().set_python_executable("venv.zip/venv/bin/python")
```

 有关`add_python_archive`和`set_python_executable`用法的详细信息，您可以参考[相关文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/python/dependency_management.html#usage)。

## 添加jar文件

PyFlink工作可能依赖于jar文件，即连接器、Java udf等。根据部署模式的不同，添加jar文件的方式也不同。

**本地**

需要将jar文件复制到`site-packages/pyflink/lib`使用的Python解释器的路径。可以执行以下命令来查找路径：

```text
python -c "import pyflink;import os;print(os.path.dirname(os.path.abspath(pyflink.__file__))+'/lib')"
```

**集群**

 可以使用命令行参数`-j <jarFile>`来指定使用的jar文件。有关的命令行参数的更多详细信息`-j <jarFile>`，可以参考[相关文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/cli.html)。

{% hint style="info" %}
当前，Flink CLI仅允许指定一个jar文件。可以将它们打包为一个zip文件，如下所示：
{% endhint %}

```text
$ # create a directory named `lib`
$ mkdir lib
$ # move the jar files to the lib directory
$ # the jar files must be located in a directory named `lib`
$ zip -r lib.zip lib
$ flink run -py xxx.py -j lib.zip
```

## 添加Python文件

可以使用命令行参数`pyfs`或`TableEnvironment`API的 add\_python\_file来添加python文件依赖项，这些依赖项可以是python文件、python包或本地目录。例如，如果有一个名为myDir的目录，该目录具有以下层次结构:

```text
myDir
├──utils
    ├──__init__.py
    ├──my_util.py
```

可以添加目录myDir的Python文件如下:

```text
table_env.add_python_file('myDir')

def my_udf():
    from utils import my_util
```

