# 数据类型

## 数据类型

由于历史原因，在Flink 1.9之前，Flink的Table和SQL API数据类型与Flink紧密耦合`TypeInformation`。`TypeInformation`在DataStream和DataSet API中使用，足以描述在分布式设置中序列化和反序列化基于JVM的对象所需的所有信息。  


从Flink 1.9开始，Table＆SQL API将获得一个新的类型系统，作为API稳定性和标准合规性的长期解决方案。

重新设计类型系统是一项主要工作，它几乎涉及所有面向用户的界面。因此，它的介绍涵盖多个版本，社区旨在通过Flink 1.10完成这项工作。

数据类型示例：

* `INT`
* `INT NOT NULL`
* `INTERVAL DAY TO SECOND(3)`
* `ROW<myField ARRAY<BOOLEAN>, myOtherField TIMESTAMP(3)>`

{% hint style="danger" %}
注意：在使用数据类型之前，请参阅规划器兼容性表和限制部分。
{% endhint %}

### Table API数据类型

{% tabs %}
{% tab title="Java" %}
```java
import static org.apache.flink.table.api.DataTypes.*;

DataType t = INTERVAL(DAY(), SECOND(3));
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.table.api.DataTypes._

val t: DataType = INTERVAL(DAY(), SECOND(3));
```
{% endtab %}
{% endtabs %}

