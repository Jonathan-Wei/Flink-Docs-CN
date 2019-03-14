# Docker

[Docker](https://www.docker.com/)是一个流行的容器运行时。Docker Hub上有用于Apache Flink的Docker镜像，可用于部署会话群集。Flink存储库还包含用于创建容器映像以部署作业集群的工具。

## Flink会话群集

Flink会话群集可用于运行多个作业。部署后，每个作业都需要提交到集群。

### Docker镜像

Flink Docker存储库驻留在Docker Hub上，提供Flink版本1.2.1及更高版本的镜像。

提供了Hadoop和Scala的每种受支持组合的镜像，并提供了标记别名以方便使用。

从Flink 1.5开始，省略Hadoop版本\(例如-hadoop28\)的镜像标记对应于不包含捆绑Hadoop发行版的Flink的无Hadoop发行版。

例如，下面的别名可以使用：_（`1.5.y`表示弗林克1.5的最新版本）_

* `flink:latest` → `flink:<latest-flink>-scala_<latest-scala>`
* `flink:1.5` → `flink:1.5.y-scala_2.11`
* `flink:1.5-hadoop27` → `flink:1.5.y-hadoop27-scala_2.11`

## Flink作业集群

Flink作业集群是一个运行单个作业的专用集群。作业是镜像的一部分，因此不需要额外的作业提交。

### Docker镜像

Flink作业集群镜像需要包含启动集群的作业的用户代码jar。因此，需要为每个作业构建专用的容器图像。`flink-container`模块包含一个`build.sh`脚本，可用于创建此类图像。请参阅[说明](https://github.com/apache/flink/blob/release-1.7/flink-container/docker/README.md)了解更多详情。

## Flink与Docker Compose

[Docker Compose](https://docs.docker.com/compose/)是一种在本地运行一组Docker容器的便捷方式。

GitHub上提供了[会话群集](https://github.com/docker-flink/examples/blob/master/docker-compose.yml)和[作业群集的](https://github.com/apache/flink/blob/release-1.7/flink-container/docker/docker-compose.yml)示例配置文件。

### 用法

* 在前台启动集群

  ```text
    docker-compose up
  ```

* 在后台启动集群

  ```text
    docker-compose up -d
  ```

* 将群集向上或向下扩展到_N_ TaskManagers

  ```text
    docker-compose scale taskmanager=<N>
  ```

* 杀死集群

  ```text
    docker-compose kill
  ```

群集运行时，可以访问位于[http://localhost:8081](http://localhost:8081/)的Web UI 。还可以使用Web UI将作业提交到会话群集。

要通过命令行将作业提交到会话群集，必须将JAR复制到JobManager容器并从那里提交作业。

例如：

```bash
$ JOBMANAGER_CONTAINER=$(docker ps --filter name=jobmanager --format={{.ID}})
$ docker cp path/to/jar "$JOBMANAGER_CONTAINER":/job.jar
$ docker exec -t -i "$JOBMANAGER_CONTAINER" flink run /job.jar
```



