# 总览

## Alluxio

 对于Alluxio支持，将以下条目添加到`core-site.xml`文件中：

```text
<property>
  <name>fs.alluxio.impl</name>
  <value>alluxio.hadoop.FileSystem</value>
</property>
```

