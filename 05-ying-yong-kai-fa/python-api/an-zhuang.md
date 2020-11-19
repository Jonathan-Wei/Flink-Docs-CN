# 安装

## 环境要求

{% hint style="info" %}
注意：PyFlink需要Python版本（3.5、3.6或3.7）。请运行以下命令以确保其符合要求：
{% endhint %}

```text
$ python --version
# the version printed here must be 3.5, 3.6 or 3.7
```

## 安装PyFlink

PyFlink已经部署到PyPi，可以按以下方式安装：

```text
$ python -m pip install apache-flink
```

 还可以按照[开发指南](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html#build-pyflink)从源代码构建PyFlink 。

{% hint style="info" %}
注意：从Flink 1.11开始，还支持在Windows本地运行PyFlink作业，因此您可以在Windows上开发和调试PyFlink作业。
{% endhint %}

