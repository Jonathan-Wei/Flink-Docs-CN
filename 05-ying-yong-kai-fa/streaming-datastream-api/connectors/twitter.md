# Twitter

[Twitter Streaming API](https://dev.twitter.com/docs/streaming-apis)提供对[Twitter](https://dev.twitter.com/docs/streaming-apis)提供的推文流的访问。Flink Streaming附带了一个`TwitterSource`用于建立与此流的连接的内置类。要使用此连接器，请将以下依赖项添加到项目中：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-twitter_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

请注意，流连接器当前不是二进制分发的一部分。请参阅[此处的](https://ci.apache.org/projects/flink/flink-docs-master/dev/linking.html)链接以进行群集执行

### **认证**

为了连接到Twitter流，用户必须注册他们的程序并获取用于认证的必要信息。该过程如下所述。

#### **获取认证信息**

首先，需要一个Twitter帐户。在[twitter.com/signup](https://twitter.com/signup)免费注册 或在Twitter的[应用程序管理中](https://apps.twitter.com/)登录并通过单击“创建新应用程序”按钮注册该应用程序。填写有关您的计划的表格并接受条款和条件。选择应用程序，API密钥和API密钥（称为后`twitter-source.consumerKey`和`twitter-source.consumerSecret`在`TwitterSource`分别）位于“API密钥”选项卡上。可以在“密钥和访问令牌”选项卡上生成和获取必要的OAuth访问令牌数据（`twitter-source.token`和`twitter-source.tokenSecret`in `TwitterSource`）。请记住保密这些信息，不要将它们推送到公共存储库。

#### **用法**

与其他连接器相比，它不`TwitterSource`依赖于其他服务。例如，设置用户twitter信息后，以下代码应该能正常运行：

{% tabs %}
{% tab title="Java" %}
```java
Properties props = new Properties();
props.setProperty(TwitterSource.CONSUMER_KEY, "");
props.setProperty(TwitterSource.CONSUMER_SECRET, "");
props.setProperty(TwitterSource.TOKEN, "");
props.setProperty(TwitterSource.TOKEN_SECRET, "");
DataStream<String> streamSource = env.addSource(new TwitterSource(props));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val props = new Properties()
props.setProperty(TwitterSource.CONSUMER_KEY, "")
props.setProperty(TwitterSource.CONSUMER_SECRET, "")
props.setProperty(TwitterSource.TOKEN, "")
props.setProperty(TwitterSource.TOKEN_SECRET, "")
val streamSource = env.addSource(new TwitterSource(props))
```
{% endtab %}
{% endtabs %}

`TwitterSource`发送包含JSON对象\(表示为`Tweet`\)的字符串。

包中的`TwitterExample`类`flink-examples-streaming`显示了如何使用`TwitterSource`的完整示例。

默认情况下，`TwitterSource`使用`StatusesSampleEndpoint`。此端点返回随机的推文样本。有一个`TwitterSource.EndpointInitializer`接口允许用户提供自定义端点。

