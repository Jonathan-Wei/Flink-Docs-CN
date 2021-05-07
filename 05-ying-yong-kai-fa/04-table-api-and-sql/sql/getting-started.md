# Getting Started

Flink SQL使使用标准SQL开发流应用程序变得简单。如果您曾经通过保持ANSI-SQL 2011兼容来使用数据库或类似SQL的系统，则很容易学习Flink。本教程将帮助您快速开始使用Flink SQL开发环境。

### 先决条件

你只需要具备SQL的基本知识即可继续学习。无需其他编程经验。

### 安装

有多种安装Flink的方法。为了进行实验，最常见的选择是下载二进制文件并在本地运行它们。您可以按照[本地安装中](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)的步骤来设置本教程其余部分的环境。

完成所有设置后，使用以下命令从安装文件夹启动本地集群：

```text
./bin/start-cluster.sh
```

启动后，可以在本地通[localhost:8081](localhost:8081)访问Flink WebUI ，可以从中监视不同的作业。

### SQL客户端

SQL客户端是一个向Flink提交SQL查询并将结果可视化的交互式客户端。要启动SQL客户端，请在安装目录下运行SQL -client脚本。

```text
./bin/sql-client.sh
```

### Hello World

## Source Tables



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

## Continuous Queries



```text
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information 
GROUP BY dep_id;
```

## Sink Tables



```text
INSERT INTO department_counts
SELECT 
   dept_id,
   COUNT(*) as emp_count 
FROM employee_information;
```

## 寻求帮助！

如果遇到困难，请查看[社区支持资源](https://flink.apache.org/community.html)。特别是，Apache Flink的[用户邮件列表](https://flink.apache.org/community.html#mailing-lists)始终被视为所有Apache项目中最活跃的[邮件列表](https://flink.apache.org/community.html#mailing-lists)之一，并且是快速获得帮助的好方法。  




