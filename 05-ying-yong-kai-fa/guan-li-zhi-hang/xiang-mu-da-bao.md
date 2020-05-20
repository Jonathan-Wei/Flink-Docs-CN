# 项目打包

如前所述，Flink程序可以通过使用远程环境在集群上执行。或者，可以将程序打包到JAR文件（Java Archives）中以供执行。打包程序是通过[命令行界面](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/cli.html)执行它们的先决条件 。

## 包装程序

要支持通过命令行或web接口从打包的JAR文件执行，程序必须使用`StreamExecutionEnvironment.getExecutionEnvironment()`获得的环境。当将JAR提交到命令行或web接口时，此环境将充当集群的环境。如果Flink程序的调用方式与通过这些接口调用的方式不同，那么该环境将充当本地环境。

要打包程序，只需将所有涉及的类导出为一个JAR文件。JAR文件的清单必须指向包含程序入口点的类\(带有`public main`方法的类\)。最简单的方法是将main-class条目放到manifest中\(例如`main-class: org.apache.flinkexample.MyProgram`\)。main-class属性与Java虚拟机在通过`java -jar pathToTheJarFile`命令执行JAR文件时用来查找主方法的属性相同。大多数ide在导出JAR文件时自动包含该属性。

## 摘要

调用打包程序的整个过程包括两个步骤:

1. 在JAR的清单中搜索_`main-class`_或_`program-class`_属性。如果找到两个属性，则_程序类_属性优先于_主类_ 属性。命令行和Web界面都支持一个参数，以在JAR manifest不包含任何属性的情况下手动传递入口点类名称。
2. 系统调用该类的main方法。

