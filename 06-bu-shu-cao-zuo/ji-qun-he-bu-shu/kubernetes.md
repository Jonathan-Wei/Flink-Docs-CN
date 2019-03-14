# Kubernetes

此页面介绍如何在[Kubernetes](https://kubernetes.io/)上部署Flink作业和会话群集

## 设置Kubernetes

请按照[Kubernetes的设置指南](https://kubernetes.io/docs/setup/)来部署Kubernetes集群。如果您想在本地运行Kubernetes，我们建议您使用[MiniKube](https://kubernetes.io/docs/setup/minikube/)。

{% hint style="info" %}
**注意：**如果使用MiniKube，请确保`minikube ssh 'sudo ip link set docker0 promisc on'`在部署Flink群集之前执行。否则Flink组件无法通过Kubernetes服务自行引用自己。
{% endhint %}

## Kubernetes上的Flink会话群集

Flink会话群集作为长期运行的Kubernetes部署执行。请注意，您可以在会话群集上运行多个Flink作业。部署群集后，需要将每个作业提交到群集。

Kubernetes中的基本Flink会话群集部署有三个组件：

* 部署/作业运行JobManager
* 部署TaskManagers池
* 暴露JobManager的REST和UI端口的服务

### 在Kubernetes上部署Flink会话群集

使用[会话群集](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/kubernetes.html#session-cluster-resource-definitions)的资源定义，使用以下`kubectl`命令启动群集：

```text
kubectl create -f jobmanager-service.yaml
kubectl create -f jobmanager-deployment.yaml
kubectl create -f taskmanager-deployment.yaml
```

然后，可以通过`kubectl proxy`以下方式访问Flink UI ：

1. `kubectl proxy`在终端中运行
2. 在浏览器输入如下地址：[ http://localhost:8001/api/v1/namespaces/default/services/flink-jobmanager:ui/proxy](%20http://localhost:8001/api/v1/namespaces/default/services/flink-jobmanager:ui/proxy)

要终止Flink会话群集，请使用`kubectl`：

```text
kubectl delete -f jobmanager-deployment.yaml
kubectl delete -f taskmanager-deployment.yaml
kubectl delete -f jobmanager-service.yaml

```

## Kubernetes上的Flink作业集群

Flink作业集群是一个运行单个作业的专用集群。作业是镜像的一部分，因此不需要额外的作业提交。

### 创建特定于作业的镜像

Flink作业集群镜像需要包含启动集群的作业的用户代码jar。因此，需要为每个作业构建专用的容器镜像。请按照这些[说明](https://github.com/apache/flink/blob/release-1.7/flink-container/docker/README.md)构建Docker镜像。

### 在Kubernetes上部署Flink作业集群

要在Kubernetes上部署作业集群，请按照这些[说明进行操作](https://github.com/apache/flink/blob/release-1.7/flink-container/kubernetes/README.md#deploy-flink-job-cluster)。

## 高级群集部署

GitHub上提供了[Flink Helm图表](https://github.com/docker-flink/examples)的早期版本。

## 附录

### 会话群集资源定义

部署定义使用`flink:latest`可[在Docker Hub](https://hub.docker.com/r/_/flink/)上找到的预构建映像。该映像是从这个[Github存储库](https://github.com/docker-flink/docker-flink)构建的。

`jobmanager-deployment.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-jobmanager
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: flink
        component: jobmanager
    spec:
      containers:
      - name: jobmanager
        image: flink:latest
        args:
        - jobmanager
        ports:
        - containerPort: 6123
          name: rpc
        - containerPort: 6124
          name: blob
        - containerPort: 6125
          name: query
        - containerPort: 8081
          name: ui
        env:
        - name: JOB_MANAGER_RPC_ADDRESS
          value: flink-jobmanager
```

`taskmanager-deployment.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-taskmanager
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: flink
        component: taskmanager
    spec:
      containers:
      - name: taskmanager
        image: flink:latest
        args:
        - taskmanager
        - "-Dtaskmanager.host=$(K8S_POD_IP)"
        ports:
        - containerPort: 6121
          name: data
        - containerPort: 6122
          name: rpc
        - containerPort: 6125
          name: query
        env:
        - name: JOB_MANAGER_RPC_ADDRESS
          value: flink-jobmanager
        - name: K8S_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```

`jobmanager-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flink-jobmanager
spec:
  ports:
  - name: rpc
    port: 6123
  - name: blob
    port: 6124
  - name: query
    port: 6125
  - name: ui
    port: 8081
  selector:
    app: flink
    component: jobmanager
```

