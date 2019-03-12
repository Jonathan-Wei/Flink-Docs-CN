# 项目打包

如前所述，Flink程序可以通过使用远程环境在集群上执行。或者，可以将程序打包到JAR文件（Java Archives）中以供执行。打包程序是通过[命令行界面](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/cli.html)执行它们的先决条件 。

## 包装程序

请注意，使用Flink可以使用genericData.record类型，但不推荐使用。由于记录包含完整的模式，它的数据非常密集，因此pr要支持通过命令行或Web界面从打包的JAR文件执行，程序必须使用`streamExecutionenvironment.getExecutionenvironment()`获得的环境。当JAR被提交到命令行或Web界面时，这个环境将充当集群的环境。如果Flink程序的调用方式与通过这些接口不同，那么环境将像本地环境一样工作。

要打包程序，只需将所有涉及的类导出为JAR文件即可。JAR文件的清单必须指向包含程序_入口点_的类（具有public `main`方法的类 ）。最简单的方法是将_main-class_条目放入清单（例如`main-class: org.apache.flinkexample.MyProgram`）。在_主类_属性是通过命令执行JAR文件时所使用的Java虚拟机发现的主要方法相同的一个`java -jar pathToTheJarFile`。大多数IDE都提供在导出JAR文件时自动包含该属性。

## 通过计划包装程序

此外，我们支持包装程序作为_计划_。`execute()`计划打包不是在主方法中定义程序并调用环境，而是 返回_程序计划_，_程序计划_是程序数据流的描述。为此，程序必须实现 `org.apache.flink.api.common.Program`接口，定义`getPlan(String...)`方法。传递给该方法的字符串是命令行参数。可以通过该`ExecutionEnvironment#createProgramPlan()`方法从环境创建程序的计划。在打包程序的计划时，JAR清单必须指向实现 `org.apache.flink.api.common.Program`接口的类，而不是使用main方法的类。

## 摘要

调用打包程序的整个过程如下：

1. 搜索JAR的清单以查找_主类_或_程序类_属性。如果找到这两个属性，则_program-class_属性优先于_main-class_ 属性。对于JAR清单既不包含属性的情况，命令行和Web界面都支持手动传递入口点类名的参数。
2. 如果入口点类实现了`org.apache.flink.api.common.Program`，则系统调用该`getPlan(String...)`方法以获取要执行的程序计划。
3. 如果入口点类没有实现`org.apache.flink.api.common.Program`接口，系统将调用该类的main方法。

