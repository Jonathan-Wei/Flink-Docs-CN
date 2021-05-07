# Getting Started

Flink SQL使使用标准SQL开发流应用程序变得简单。如果您曾经通过保持ANSI-SQL 2011兼容来使用数据库或类似SQL的系统，则很容易学习Flink。本教程将帮助您快速开始使用Flink SQL开发环境。

### 先决条件

你只需要具备SQL的基本知识即可继续学习。无需其他编程经验。

### 安装

有多种安装Flink的方法。为了进行实验，最常见的选择是下载二进制文件并在本地运行它们。您可以按照[本地安装中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)的步骤来设置本教程其余部分的环境。

完成所有设置后，使用以下命令从安装文件夹启动本地集群：

```bash
./bin/start-cluster.sh
```

启动后，可以在本地通过[localhost:8081](localhost:8081)访问Flink WebUI ，可以从中监视不同的作业。

### SQL客户端

SQL客户端是一个向Flink提交SQL查询并将结果可视化的交互式客户端。要启动SQL客户端，请在安装目录下运行SQL -client脚本。

```text
./bin/sql-client.sh
```

### Hello World

SQL客户机\(查询编辑器\)启动并运行后，就可以开始编写查询了。让我们从打印' Hello World '开始，使用以下简单的查询:

```sql
SELECT 'Hello World';
```

运行该`HELP`命令将列出支持的SQL语句的完整集合。让我们运行`SHOW`命令查看Flink[内置函数](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/functions/systemfunctions/)的完整列表。

```sql
SHOW FUNCTIONS;
```

这些函数为用户在开发SQL查询时提供了功能强大的工具箱。例如，`CURRENT_TIMESTAMP`将在执行该命令的机器上打印该机器的当前系统时间。

```sql
SELECT CURRENT_TIMESTAMP;
```

## Source Tables

与所有SQL引擎一样，Flink查询在表的顶部进行操作。它不同于传统的数据库，因为Flink不在本地管理静态数据；相反，它的查询在外部表上连续运行。

Flink数据处理管道从Source Table开始。Table Source产生行数据是通过查询执行生成的;它们是查询的FROM子句中引用的表。这些可能是Kafka Topic、数据库、文件系统，或者是Flink支持的任何其他系统。

可以通过SQL客户端或使用环境配置文件来定义表。SQL客户端支持类似于传统SQL的[SQL DDL命令](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/overview/)。标准SQL DDL用于[创建](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/create/)，[更改](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/alter/)，[删除](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/drop/)表。

Flink支持可用于Table的不同[连接器](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/connectors/table/overview/)和[格式](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/connectors/table/formats/overview/)。下面的CREATE Table示例定义一个由CSV文件支持的Source Table，其中emp\_id、name、dept\_id为列。

```text
CREATE TABLE employee_information (
    emp_id INT,
    name VARCHAR,
    dept_id INT
) WITH ( 
    'connector' = 'filesystem',
    'path' = '/path/to/something.csv',
    'format' = 'csv'
);
```

可以从该表中定义一个连续查询，当新行可用时，该查询将读取新行并立即输出结果。例如，我们可以只过滤那些在部门1工作的员工。

```text
SELECT * from employee_information WHERE DeptId = 1;
```

## Continuous Queries

尽管最初并不是在设计时就考虑到流语义的，但是SQL是构建连续数据管道的强大工具。Flink SQL与传统数据库查询的不同之处在于，Flink SQL在行到达时不断消耗并对其结果进行更新。

[连续查询](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/dynamic_tables/#continuous-queries)永远不会终止，并产生一个动态表作为结果。[动态表](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/dynamic_tables/#continuous-queries)是Flink的Table API和SQL对流数据的支持的核心概念

连续流上的聚合需要在查询执行期间连续存储聚合结果。例如，假设您需要从传入的数据流中计算每个部门的员工人数。查询需要保持每个部门的最新计数，以便在处理新行时及时输出结果。

```text
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information 
GROUP BY dep_id;
```

此类查询是有状态的。Flink的高级容错机制将保持内部状态和一致性，因此查询总是返回正确的结果，即使在遇到硬件故障时也是如此。

## Sink Tables

运行此查询时，SQL客户机以只读方式实时提供输出。存储结果—为报表或仪表板提供动力—需要写入另一个表。这可以通过INSERT INTO语句来实现。该子句中引用的表称为SINK Table。INSERT INTO语句将作为分离查询提交给Flink集群。

```text
INSERT INTO department_counts
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information;
```

提交后，它将运行并将结果直接存储到Sink Table中，而不是将结果加载到系统内存中。

## 寻求帮助！

如果遇到困难，请查看[社区支持资源](https://flink.apache.org/community.html)。特别是，Apache Flink的[用户邮件列表](https://flink.apache.org/community.html#mailing-lists)始终被视为所有Apache项目中最活跃的[邮件列表](https://flink.apache.org/community.html#mailing-lists)之一，并且是快速获得帮助的好方法。  




