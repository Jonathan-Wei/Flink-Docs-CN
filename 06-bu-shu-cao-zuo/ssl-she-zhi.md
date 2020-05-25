# SSL设置

本页提供有关如何启用TLS / SSL身份验证和加密以与Flink进程之间进行网络通信的说明。

## 内部和外部连接

通过身份验证和加密保护机器进程之间的网络连接时，Apache Flink区分_内部_和_外部_连接。 _内部连接_是指Flink进程之间建立的所有连接。这些连接运行Flink自定义协议。用户永远不会直接连接到内部连接端点。 _外部/REST连接_端点是指从外部到Flink进程的所有连接。这包括用于启动和控制正在运行的Flink作业/应用程序的Web UI和REST命令，包括Flink CLI与JobManager / Dispatcher的通信。

为了获得更大的灵活性，可以分别启用和配置内部和外部连接的安全性。

![](../.gitbook/assets/image%20%2844%29.png)

### 内部连接

内部连接包括：

* 控制消息：JobManager / TaskManager / Dispatcher / ResourceManager之间的RPC
* 数据平面：TaskManager之间的连接，用于在随机播放，广播，重新分发等期间交换数据。
* Blob服务（库和其他工件的分发）。

 所有内部连接均经过SSL身份验证和加密。这些连接使用**相互身份验证**，这意味着每个连接的服务器端和客户端都需要相互提供证书。当使用专用CA对内部证书进行独占签名时，证书可以有效地充当共享机密。内部通信的证书不需要任何其他方与Flink进行交互，可以将其简单地添加到容器映像中，或附加到YARN部署中。

* 实现此设置的最简单方法是为Flink部署生成专用的公钥/私钥对和自签名证书。密钥和信任存储区是相同的，只包含那个密钥/证书对。[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/security-ssl.html#example-ssl-setup-standalone-and-kubernetes)展示一个示例。
* 在这样的环境中，操作符必须使用全公司范围的内部CA\(不能生成自签名证书\),仍然建议为该CA保留Flink部署专用的密钥对/证书。但是，TrustStore然后还必须包含CA的公共证书，以便在SSL握手期间接受部署的证书（JDK TrustStore实现中的要求）。

 **注意：**因此，至关重要的是，指定部署证书（`security.ssl.internal.cert.fingerprint`）的指纹（如果不是自签名的），则将该证书固定为唯一的受信任证书，并阻止TrustStore信任该CA签署的所有证书

 _注意：由于内部连接是使用共享证书相互认证的，因此Flink可以跳过主机名验证。这使基于容器的设置更加容易。_

### 外部/REST连接

所有外部连接都通过HTTP / REST端点公开，例如Web UI和CLI使用的端点：

* 与Dispatcher 通信以提交作业（会话集群）
* 与_JobManager_通信以检查和修改正在运行的作业/应用程序

可以将REST端点配置为要求SSL连接。但是，默认情况下，服务器将接受来自任何客户端的连接，这意味着REST端点不对客户端进行身份验证。

如果需要对与REST端点的连接进行身份验证，则可以通过配置启用简单的相互身份验证，但是我们建议部署“side car proxy”：将REST端点绑定到环回接口（或Kubernetes中的pod-local接口），并且启动一个REST代理，该代理对请求进行身份验证并将其转发到Flink。Flink用户已部署的代理的示例是[Envoy代理](https://www.envoyproxy.io/)或带 [MOD\_AUTH的NGINX](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html)。

将身份验证委托给代理的基本原理是这样的：代理提供了各种各样的身份验证选项，从而更好地集成到现有的基础设施中。

### 可查询状态

到可查询状态端点的连接当前未经过身份验证或加密。

## 配置SSL

可以分别为_内部_和_外部_连接启用S​​SL ：

* **security.ssl.internal.enabled**：为所有_内部_连接启用S​​SL 。
* **security.ssl.rest.enabled**：为_REST/外部_连接启用S​​SL 。

_注意：为了向后兼容，**security.ssl.enabled**选项仍然存在，并为内部和REST端点启用SSL。_

对于内部连接，可以选择禁用不同连接类型的安全性。当`security.ssl.internal.enabled`设置为`true`，可以设置以下参数`false`，为特定的连接类型禁用SSL：

* `taskmanager.data.ssl.enabled`：TaskManager之间的数据通信
* `blob.service.ssl.enabled`：BLOB从JobManager到TaskManager的传输
* `akka.ssl.enabled`：JobManager / TaskManager / ResourceManager之间基于Akka的RPC连接

### 密钥库和信任库

#### **内部连接**

由于内部通信在服务器和客户端之间相互进行身份验证，因此密钥库和信任库通常引用充当共享机密的专用证书。在这种设置中，证书可以使用通配符主机名或地址。使用自签名证书时，甚至可以使用与密钥库和信任库相同的文件。

```text
security.ssl.internal.keystore: /path/to/file.keystore
security.ssl.internal.keystore-password: keystore_password
security.ssl.internal.key-password: key_password
security.ssl.internal.truststore: /path/to/file.truststore
security.ssl.internal.truststore-password: truststore_password
```

使用非自签名而是由CA签名的证书时，需要使用证书固定来建立连接时仅信任特定证书。

```text
security.ssl.internal.cert.fingerprint: 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
```

**REST端点（外部连接）**

对于REST端点，默认情况下，服务器端点使用密钥库，而REST客户端（包括CLI客户端）使用信任库来接受服务器的证书。在REST密钥库具有自签名证书的情况下，信任库必须直接信任该证书。如果REST端点使用通过适当的证书层次结构签名的证书，则该层次结构的根应位于信任存储区中。

如果启用了相互身份验证，则服务器端点和REST客户端将使用密钥库和信任库，就像内部连接一样。

```text
security.ssl.rest.keystore: /path/to/file.keystore
security.ssl.rest.keystore-password: keystore_password
security.ssl.rest.key-password: key_password
security.ssl.rest.truststore: /path/to/file.truststore
security.ssl.rest.truststore-password: truststore_password
security.ssl.rest.authentication-enabled: false
```

### 密码套件

{% hint style="danger" %}
**重要的是**，[IETF RFC 7525](https://tools.ietf.org/html/rfc7525)推荐使用一组特定的密码套件来实现强大的安全性。因为这些密码套件在很多情况下都是不可用的，所以Flink的默认值被设置为一个稍弱但更兼容的密码套件。如果可能，我们建议将SSL设置更新到更强大的密码套件，方法是在Flink配置中添加以下配置:
{% endhint %}

```text
security.ssl.algorithms: TLS_DHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

如果您的设置不支持这些密码套件，将看到Flink进程将无法相互连接。

### SSL选项的完整列表

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x952E;</th>
      <th style="text-align:left">&#x9ED8;&#x8BA4;</th>
      <th style="text-align:left">&#x7C7B;&#x578B;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>security.kerberos.login.contexts</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x7528;&#x9017;&#x53F7;&#x5206;&#x9694;&#x7684;&#x767B;&#x5F55;&#x4E0A;&#x4E0B;&#x6587;&#x5217;&#x8868;&#xFF0C;&#x7528;&#x4E8E;&#x5411;&#x5176;&#x63D0;&#x4F9B;Kerberos&#x51ED;&#x636E;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x201C;
        Client&#xFF0C;KafkaClient&#x201D;&#xFF0C;&#x7528;&#x4E8E;&#x5C06;&#x51ED;&#x636E;&#x7528;&#x4E8E;ZooKeeper&#x8EAB;&#x4EFD;&#x9A8C;&#x8BC1;&#x548C;Kafka&#x8EAB;&#x4EFD;&#x9A8C;&#x8BC1;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.kerberos.login.keytab</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5305;&#x542B;&#x7528;&#x6237;&#x51ED;&#x636E;&#x7684;Kerberos keytab&#x6587;&#x4EF6;&#x7684;&#x7EDD;&#x5BF9;&#x8DEF;&#x5F84;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.kerberos.login.principal</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x4E0E;&#x5BC6;&#x94A5;&#x8868;&#x5173;&#x8054;&#x7684;Kerberos&#x4E3B;&#x4F53;&#x540D;&#x79F0;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.kerberos.login.use-ticket-cache</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6307;&#x793A;&#x662F;&#x5426;&#x4ECE;Kerberos&#x7968;&#x8BC1;&#x7F13;&#x5B58;&#x4E2D;&#x8BFB;&#x53D6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.authentication-enabled</b>
      </td>
      <td style="text-align:left">&#x201C; TLS_RSA_WITH_AES_128_CBC_SHA&#x201D;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x8981;&#x652F;&#x6301;&#x7684;&#x6807;&#x51C6;SSL&#x7B97;&#x6CD5;&#x7684;&#x9017;&#x53F7;&#x5206;&#x9694;&#x5217;&#x8868;&#x3002;
        <a
        href="http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#ciphersuites">&#x5728;&#x8FD9;&#x91CC;</a>&#x9605;&#x8BFB;&#x66F4;&#x591A;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.cert.fingerprint</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5185;&#x90E8;&#x8BC1;&#x4E66;&#x7684;sha1&#x6307;&#x7EB9;&#x3002;&#x8FD9;&#x53EF;&#x4EE5;&#x8FDB;&#x4E00;&#x6B65;&#x4FDD;&#x62A4;&#x5185;&#x90E8;&#x901A;&#x4FE1;&#xFF0C;&#x4EE5;&#x663E;&#x793A;Flink&#x4F7F;&#x7528;&#x7684;&#x786E;&#x5207;&#x8BC1;&#x4E66;&#x3002;&#x5728;&#x4E0D;&#x80FD;&#x4F7F;&#x7528;&#x79C1;&#x6709;CA&#xFF08;&#x81EA;&#x7B7E;&#x540D;&#xFF09;&#x6216;&#x9700;&#x8981;&#x5185;&#x90E8;&#x516C;&#x53F8;&#x8303;&#x56F4;CA&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x8FD9;&#x662F;&#x5FC5;&#x9700;&#x7684;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.close-notify-flush-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">&#x5237;&#x65B0;&#x7531;&#x5173;&#x95ED;&#x901A;&#x9053;&#x89E6;&#x53D1;&#x7684;close_notify&#x7684;&#x8D85;&#x65F6;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;&#x5982;&#x679C;&#x5728;&#x7ED9;&#x5B9A;&#x7684;&#x8D85;&#x65F6;&#x65F6;&#x95F4;&#x5185;&#x6CA1;&#x6709;&#x5237;&#x65B0;`close_notify`&#xFF0C;&#x901A;&#x9053;&#x5C06;&#x88AB;&#x5F3A;&#x5236;&#x5173;&#x95ED;&#x3002;&#xFF08;-1
        =&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x9ED8;&#x8BA4;&#x503C;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.enabled</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6253;&#x5F00;SSL&#x4EE5;&#x8FDB;&#x884C;&#x5185;&#x90E8;&#x7F51;&#x7EDC;&#x901A;&#x4FE1;&#x3002;&#xFF08;&#x53EF;&#x9009;&#xFF09;&#x7279;&#x5B9A;&#x7EC4;&#x4EF6;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;&#x5176;&#x81EA;&#x5DF1;&#x7684;&#x8BBE;&#x7F6E;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;REST&#x7B49;&#xFF09;&#x8986;&#x76D6;&#x6B64;&#x8BBE;&#x7F6E;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.shake-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">SSL&#x63E1;&#x624B;&#x671F;&#x95F4;&#x7684;&#x8D85;&#x65F6;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;&#xFF08;-1
        =&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x9ED8;&#x8BA4;&#x503C;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.key-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5728;Flink&#x7684;&#x5185;&#x90E8;&#x7AEF;&#x70B9;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;blob&#x670D;&#x52A1;&#x5668;&#xFF09;&#x7684;&#x5BC6;&#x94A5;&#x5E93;&#x4E2D;&#x89E3;&#x5BC6;&#x5BC6;&#x94A5;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.keystore</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5177;&#x6709;SSL&#x5BC6;&#x94A5;&#x548C;&#x8BC1;&#x4E66;&#x7684;Java&#x5BC6;&#x94A5;&#x5E93;&#x6587;&#x4EF6;&#xFF0C;&#x5C06;&#x7528;&#x4E8E;Flink&#x7684;&#x5185;&#x90E8;&#x7AEF;&#x70B9;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;blob&#x670D;&#x52A1;&#x5668;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.keystore-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x89E3;&#x5BC6;Flink&#x5185;&#x90E8;&#x7AEF;&#x70B9;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;blob&#x670D;&#x52A1;&#x5668;&#xFF09;&#x7684;Flink&#x5BC6;&#x94A5;&#x5E93;&#x6587;&#x4EF6;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.session-cache-size</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">&#x7528;&#x4E8E;&#x5B58;&#x50A8;SSL&#x4F1A;&#x8BDD;&#x5BF9;&#x8C61;&#x7684;&#x7F13;&#x5B58;&#x7684;&#x5927;&#x5C0F;&#x3002;&#x6839;&#x636E;https://github.com/netty/netty/issues/832&#xFF0C;&#x60A8;&#x5E94;&#x8BE5;&#x59CB;&#x7EC8;&#x5C06;&#x5176;&#x8BBE;&#x7F6E;&#x4E3A;&#x9002;&#x5F53;&#x7684;&#x6570;&#x5B57;&#xFF0C;&#x4EE5;&#x514D;&#x5728;&#x5783;&#x573E;&#x56DE;&#x6536;&#x671F;&#x95F4;&#x9047;&#x5230;&#x56E0;&#x62D6;&#x5EF6;IO&#x7EBF;&#x7A0B;&#x800C;&#x5BFC;&#x81F4;&#x9519;&#x8BEF;&#x7684;&#x9519;&#x8BEF;&#x3002;&#xFF08;-1
        =&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x9ED8;&#x8BA4;&#x503C;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.session-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">&#x7F13;&#x5B58;&#x7684;SSL&#x4F1A;&#x8BDD;&#x5BF9;&#x8C61;&#x7684;&#x8D85;&#x65F6;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;&#xFF08;-1
        =&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x9ED8;&#x8BA4;&#x503C;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.truststore</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5305;&#x542B;&#x516C;&#x5171;CA&#x8BC1;&#x4E66;&#x7684;&#x4FE1;&#x4EFB;&#x5E93;&#x6587;&#x4EF6;&#xFF0C;&#x7528;&#x4E8E;&#x9A8C;&#x8BC1;Flink&#x5185;&#x90E8;&#x7AEF;&#x70B9;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;blob&#x670D;&#x52A1;&#x5668;&#xFF09;&#x7684;<b>TrustStore</b>&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.truststore-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x7528;&#x4E8E;&#x89E3;&#x5BC6;Flink&#x5185;&#x90E8;&#x7AEF;&#x70B9;&#xFF08;rpc&#xFF0C;&#x6570;&#x636E;&#x4F20;&#x8F93;&#xFF0C;blob&#x670D;&#x52A1;&#x5668;&#xFF09;&#x7684;&#x4FE1;&#x4EFB;&#x5E93;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.protocol</b>
      </td>
      <td style="text-align:left">&#x201C; TLSv1.2&#x201D;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">SSL&#x4F20;&#x8F93;&#x652F;&#x6301;&#x7684;SSL&#x534F;&#x8BAE;&#x7248;&#x672C;&#x3002;&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x5B83;&#x4E0D;&#x652F;&#x6301;&#x4EE5;&#x9017;&#x53F7;&#x5206;&#x9694;&#x7684;&#x5217;&#x8868;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.provider</b>
      </td>
      <td style="text-align:left">&#x201C; JDK&#x201D;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>&#x7528;&#x4E8E;ssl&#x4F20;&#x8F93;&#x7684;SSL&#x5F15;&#x64CE;&#x63D0;&#x4F9B;&#x7A0B;&#x5E8F;&#xFF1A;</p>
        <ul>
          <li><code>JDK</code>&#xFF1A;&#x9ED8;&#x8BA4;&#x7684;&#x57FA;&#x4E8E;Java&#x7684;SSL&#x5F15;&#x64CE;</li>
          <li><code>OPENSSL</code>&#xFF1A;&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x5E93;&#x7684;&#x57FA;&#x4E8E;openSSL&#x7684;SSL&#x5F15;&#x64CE;</li>
        </ul>
        <p><code>OPENSSL</code>&#x57FA;&#x4E8E;<a href="http://netty.io/wiki/forked-tomcat-native.html#wiki-h2-4">netty-tcnative</a>&#xFF0C;&#x6709;&#x4E24;&#x79CD;&#x98CE;&#x683C;&#xFF1A;</p>
        <ul>
          <li>&#x52A8;&#x6001;&#x94FE;&#x63A5;&#xFF1A;&#x8FD9;&#x5C06;&#x4F7F;&#x7528;&#x7CFB;&#x7EDF;&#x7684;openSSL&#x5E93;&#xFF08;&#x5982;&#x679C;&#x517C;&#x5BB9;&#xFF09;&#xFF0C;&#x5E76;&#x4E14;&#x9700;&#x8981;<code>opt/flink-shaded-netty-tcnative-dynamic-*.jar</code>&#x5C06;&#x5176;&#x590D;&#x5236;&#x5230;<code>lib/</code>
          </li>
          <li>&#x9759;&#x6001;&#x94FE;&#x63A5;&#xFF1A;&#x7531;&#x4E8E;openSSL&#x53EF;&#x80FD;&#x5B58;&#x5728;&#x8BB8;&#x53EF;&#x95EE;&#x9898;&#xFF08;&#x8BF7;&#x53C2;&#x9605;
            <a
            href="https://issues.apache.org/jira/browse/LEGAL-393">LEGAL-393</a>&#xFF09;&#xFF0C;&#x6211;&#x4EEC;&#x65E0;&#x6CD5;&#x4EA4;&#x4ED8;&#x9884;&#x6784;&#x5EFA;&#x7684;&#x5E93;&#x3002;&#x4F46;&#x662F;&#xFF0C;&#x60A8;&#x53EF;&#x4EE5;&#x81EA;&#x5DF1;&#x6784;&#x5EFA;&#x6240;&#x9700;&#x7684;&#x5E93;&#x5E76;&#x5C06;&#x5176;&#x653E;&#x5165;<code>lib/</code>&#xFF1A;
              <br
              /><code>git clone https://github.com/apache/flink-shaded.git &amp;amp;&amp;amp; cd flink-shaded &amp;amp;&amp;amp; mvn clean package -Pinclude-netty-tcnative-static -pl flink-shaded-netty-tcnative-static</code>
          </li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.authentication-enabled</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6253;&#x5F00;&#x7528;&#x4E8E;&#x901A;&#x8FC7;REST&#x7AEF;&#x70B9;&#x8FDB;&#x884C;&#x5916;&#x90E8;&#x901A;&#x4FE1;&#x7684;&#x76F8;&#x4E92;SSL&#x8EAB;&#x4EFD;&#x9A8C;&#x8BC1;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.cert.fingerprint</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5176;&#x4F59;&#x8BC1;&#x4E66;&#x7684;sha1&#x6307;&#x7EB9;&#x3002;&#x8FD9;&#x8FDB;&#x4E00;&#x6B65;&#x4FDD;&#x62A4;&#x4E86;&#x5176;&#x4F59;REST&#x7AEF;&#x70B9;&#x4EE5;&#x63D0;&#x4F9B;&#x4EC5;&#x7531;&#x4EE3;&#x7406;&#x670D;&#x52A1;&#x5668;&#x4F7F;&#x7528;&#x7684;&#x8BC1;&#x4E66;&#x8FD9;&#x5728;&#x66FE;&#x7ECF;&#x4F7F;&#x7528;&#x516C;&#x5171;CA&#x6216;&#x5185;&#x90E8;&#x516C;&#x53F8;&#x8303;&#x56F4;CA&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x662F;&#x5FC5;&#x8981;&#x7684;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.enabled</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5F00;&#x542F;SSL&#x4EE5;&#x901A;&#x8FC7;REST&#x7AEF;&#x70B9;&#x8FDB;&#x884C;&#x5916;&#x90E8;&#x901A;&#x4FE1;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.key-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5728;Flink&#x7684;&#x5916;&#x90E8;REST&#x7AEF;&#x70B9;&#x7684;&#x5BC6;&#x94A5;&#x5E93;&#x4E2D;&#x89E3;&#x5BC6;&#x5BC6;&#x94A5;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.keystore</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5E26;&#x6709;SSL&#x5BC6;&#x94A5;&#x548C;&#x8BC1;&#x4E66;&#x7684;Java&#x5BC6;&#x94A5;&#x5E93;&#x6587;&#x4EF6;&#xFF0C;&#x5C06;&#x7528;&#x4E8E;Flink&#x7684;&#x5916;&#x90E8;REST&#x7AEF;&#x70B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.keystore-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x4E3A;Flink&#x7684;&#x5916;&#x90E8;REST&#x7AEF;&#x70B9;&#x89E3;&#x5BC6;Flink&#x7684;&#x5BC6;&#x94A5;&#x5E93;&#x6587;&#x4EF6;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.truststore</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5305;&#x542B;&#x516C;&#x5171;CA&#x8BC1;&#x4E66;&#x7684;&#x4FE1;&#x4EFB;&#x5E93;&#x6587;&#x4EF6;&#xFF0C;&#x7528;&#x4E8E;&#x9A8C;&#x8BC1;Flink&#x7684;&#x5916;&#x90E8;REST&#x7AEF;&#x70B9;&#x7684;TrustStore&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.rest.truststore-password</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x7528;&#x4E8E;&#x89E3;&#x5BC6;Flink&#x7684;&#x5916;&#x90E8;REST&#x7AEF;&#x70B9;&#x7684;&#x4FE1;&#x4EFB;&#x5E93;&#x7684;&#x5BC6;&#x7801;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.verify-hostname</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6807;&#x8BB0;&#x4EE5;&#x5728;ssl&#x63E1;&#x624B;&#x671F;&#x95F4;&#x542F;&#x7528;&#x5BF9;&#x7B49;&#x65B9;&#x7684;&#x4E3B;&#x673A;&#x540D;&#x9A8C;&#x8BC1;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>zookeeper.sasl.disable</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><b>zookeeper.sasl.login-context-name</b>
      </td>
      <td style="text-align:left">&#x201C;Client&#x201D;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><b>zookeeper.sasl.service-name</b>
      </td>
      <td style="text-align:left">&#x201C;zookeeper&#x201D;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>## 创建和部署密钥库和信任库

可以使用[keytool实用程序](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)生成密钥，证书以及密钥库和信任库。需要Flink群集中的每个节点都具有适当的Java密钥库和信任库。

* 对于独立配置，这意味着将文件复制到每个节点，或将它们添加到共享的挂载目录。
* 对于基于容器的设置，请将密钥库和信任库文件添加到容器映像。
* 对于Yarn / Mesos配置，集群部署阶段可以自动分发密钥库和信任库文件。

对于面向外部的REST端点，证书中的通用名称或主题备用名称应与节点的主机名和IP地址匹配。

## 设置独立版和Kubernetes ****SSL示例

**内部连接**

执行keytool命令在密钥库中创建密钥对：

```text
keytool -genkeypair -alias flink.internal -keystore internal.keystore -dname "CN=flink.internal" -storepass internal_store_password -keyalg RSA -keysize 4096 -storetype PKCS12
```

服务器和客户端端点以相同的方式使用密钥库中的单个密钥/证书（相互认证）。密钥对充当内部安全的共享机密，我们可以直接将其用作密钥库和信任库。

```text
security.ssl.internal.enabled: true
security.ssl.internal.keystore: /path/to/flink/conf/internal.keystore
security.ssl.internal.truststore: /path/to/flink/conf/internal.keystore
security.ssl.internal.keystore-password: internal_store_password
security.ssl.internal.truststore-password: internal_store_password
security.ssl.internal.key-password: internal_store_password
```

**REST端点**

REST端点可以接收来自外部进程的连接，包括不属于Flink的工具（例如，对REST API的curl请求）。设置通过CA层次结构签名的正确证书对于REST端点可能是有意义的。

但是，如上所述，REST端点不对客户端进行身份验证，因此通常无论如何都需要通过代理进行保护。

**REST端点（简单的自签名证书）**

本示例说明如何创建简单的密钥库/信任库对。信任库不包含主键，可以与其他应用程序共享。在此示例中，_`myhost.company.org/ip:10.0.2.15`_是Flink主服务器的节点（或服务）。

```text
keytool -genkeypair -alias flink.rest -keystore rest.keystore -dname "CN=myhost.company.org" -ext "SAN=dns:myhost.company.org,ip:10.0.2.15" -storepass rest_keystore_password -keyalg RSA -keysize 4096 -storetype PKCS12

keytool -exportcert -keystore rest.keystore -alias flink.rest -storepass rest_keystore_password -file flink.cer

keytool -importcert -keystore rest.truststore -alias flink.rest -storepass rest_truststore_password -file flink.cer -noprompt
```

```text
security.ssl.rest.enabled: true
security.ssl.rest.keystore: /path/to/flink/conf/rest.keystore
security.ssl.rest.truststore: /path/to/flink/conf/rest.truststore
security.ssl.rest.keystore-password: rest_keystore_password
security.ssl.rest.truststore-password: rest_truststore_password
security.ssl.rest.key-password: rest_keystore_password
```

**REST端点（带有自签名的CA）**

执行keytool命令创建具有自签名CA的信任库。

```text
keytool -genkeypair -alias ca -keystore ca.keystore -dname "CN=Sample CA" -storepass ca_keystore_password -keyalg RSA -keysize 4096 -ext "bc=ca:true" -storetype PKCS12

keytool -exportcert -keystore ca.keystore -alias ca -storepass ca_keystore_password -file ca.cer

keytool -importcert -keystore ca.truststore -alias ca -storepass ca_truststore_password -file ca.cer -noprompt
```

现在，使用上述CA签名的证书为REST端点创建密钥库。假设_f`link.company.org/ip:10.0.2.15`_为Flink主服务器（_JobManager）_的主机名。

```text
keytool -genkeypair -alias flink.rest -keystore rest.signed.keystore -dname "CN=flink.company.org" -ext "SAN=dns:flink.company.org" -storepass rest_keystore_password -keyalg RSA -keysize 4096 -storetype PKCS12

keytool -certreq -alias flink.rest -keystore rest.signed.keystore -storepass rest_keystore_password -file rest.csr

keytool -gencert -alias ca -keystore ca.keystore -storepass ca_keystore_password -ext "SAN=dns:flink.company.org,ip:10.0.2.15" -infile rest.csr -outfile rest.cer

keytool -importcert -keystore rest.signed.keystore -storepass rest_keystore_password -file ca.cer -alias ca -noprompt

keytool -importcert -keystore rest.signed.keystore -storepass rest_keystore_password -file rest.cer -alias flink.rest -noprompt
```

现在将以下配置添加到`flink-conf.yaml`：

```text
security.ssl.rest.enabled: true
security.ssl.rest.keystore: /path/to/flink/conf/rest.signed.keystore
security.ssl.rest.truststore: /path/to/flink/conf/ca.truststore
security.ssl.rest.keystore-password: rest_keystore_password
security.ssl.rest.key-password: rest_keystore_password
security.ssl.rest.truststore-password: ca_truststore_password
```

**使用curl实用工具查询REST Endpoint的提示**

可以使用`openssl`命令将密钥库转换为`PEM`格式：

```text
openssl pkcs12 -passin pass:rest_keystore_password -in rest.keystore -out rest.pem -nodes
```

然后，可以使用`curl`命令查询REST Endpoint ：

```text
curl --cacert rest.pem flink_url
```

如果启用了双向SSL：

```text
curl --cacert rest.pem --cert rest.pem flink_url
```

## YARN / Mesos部署建议

对于YARN和Mesos，可以使用Yarn和Mesos的工具来帮忙：

* 配置内部通信的安全性与上面的示例完全相同。
* 为了保护REST端点，需要颁发REST端点的证书，以使其对于Flink主服务器可能部署到的所有主机均有效。可以使用通配符DNS名称或添加多个DNS名称来完成。
*  部署密钥库和信任库的最简单方法是使用YARN客户端的选项（`-yt`）。将密钥库和信任库文件复制到本地目录（例如`deploy-keys/`），并按如下所示启动YARN会话：`flink run -m yarn-cluster -yt deploy-keys/ flinkapp.jar`
* 使用YARN部署时，可通过YARN代理的跟踪URL访问Flink的Web仪表板。为了确保YARN代理能够访问Flink的HTTPS URL，需要配置YARN代理以接受Flink的SSL证书。为此，请将自定义CA证书添加到YARN代理节点上的Java的默认信任库中。



