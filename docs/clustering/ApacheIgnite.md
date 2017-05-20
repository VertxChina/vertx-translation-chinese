# Ignite Cluster Manager

> 翻译：Ranger Tsao，校对 宋子豪、赵亮

`IgniteClusterManager` 是基于 [Apache Ignite](https://ignite.apache.org) 实现。由于 Vert.x 集群管理的可插拔性，也可轻易切换至其它的集群管理器。

Vert.x 集群管理器包含以下几个功能：

1. 发现并管理集群中的节点
2. 管理集群端的主题订阅清单（这样就可以轻松得知集群中的那些节点订阅了那些 EventBus 地址）
3. 分布式 Map 支持
4. 分布式锁
5. 分布式计数器

**注意事项**
Vert.x 集群器并不处理节点之间的通信，在 Vert.x 中节点中的通信是直接由 TCP 链接处理的。

## 使用 Ignite cluster manager

Vert.x 能够从 classpath 路径的 jar 自动检测并使用出 `ClusterManager` 的实现。不过需要确保在 classpath 没有其他的 `ClusterManager` 实现。

另外 Vert.x 可以通过设置 `-Dvertx.clusterManagerFactory=io.vertx.spi.cluster.ignite.IgniteClusterManager` 来使用指定的 集群管理器实现。

### 通过命令行使用

确保 `vertx-ignite-3.4.1.jar` 在 Vert.x 的安装路径中的 lib 目录下。

### 通过 Maven 或 Gradle 构建工具使用

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-ignite</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(gradle.xml)

```groovy
compile 'io.vertx:vertx-ignite:3.4.1'
```

### 编程形式调用

通过编码的形式，设置集群管理器实现，例子：

```java
ClusterManager clusterManager = new IgniteClusterManager();//创建 Ignite 集群管理器

VertxOptions options = new VertxOptions().setClusterManager(clusterManager);//设置集群管理器实现
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

## 配置 Ignite cluster manager

### 使用配置文件

在 `vertx-ignite` jar 中内嵌了一个默认配置文件 `default-ignite.xml`。
如果需要覆盖此配置文件，需要在 classpath 路径中添加 `ignite.xml`

`default-ignite.xml` 与 `ignite.xml` 必须是一个 `Apache Ignite` 配置文件，在 [Apache Ignite](https://www.zybuluo.com/liyuj/note/230739) 的官方中文文档中，可以找到具体的配置描述。

### 编程配置

```java
IgniteConfiguration cfg = new IgniteConfiguration();
// 配置 Ignite ，此处省略配置代码

ClusterManager clusterManager = new IgniteClusterManager(cfg);

VertxOptions options = new VertxOptions().setClusterManager(clusterManager);
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

### 自动发现及网络传输配置

在默认配置中使用 `TcpDiscoveryMulticastIpFinder` 实现，这网络发现实现，需要开启组播。如果组播被禁用，可以采用 `TcpDiscoveryVmIpFinder` 替代，前提是在 `ignite.xml` 事先配置好 IP 地址列表。具体 Ignite 集群配置，可以参考文档 [Apache Ignite](https://www.zybuluo.com/liyuj/note/230739)。

## 故障排除

### 组播未正常开启

MacOS 默认禁用组播。Google一下启用组播。

### 使用错误的网络接口

如果机器上有多个网络接口（也有可能是在运行 VPN 的情况下），那么 Ignite 很有可能是使用了错误的网络接口。

为了确保 Ignite 使用正确的网络接口，在配置文件中将 `IgniteConfiguration` 设置为指定IP地址。 例如：

```xml
<bean class="org.apache.ignite.configuration.IgniteConfiguration">
  <property name="localHost" value="192.168.1.20"/>
</bean>
```

当运行集群模式时，需要确保 Vert.x 使用正确的网络接口。当通过命令行模式时，可以设置 `cluster-host` 参数：

```bash
vertx run myverticle.js -cluster -cluster-host your-ip-address
```

其中 `your-ip-address` 必须与 Ignite 中的配置保持一致。

当通过编程模式使用 Vert.x 时，可以调用方法 `setClusterHost` 来设置参数

### 使用VPN

VPN 软件通常通过创建不支持组播的虚拟网络接口来进行工作。在 VPN 环境中，如果 Ignite 与 Vert.x 不正确配置的话，VPN 接口将被选择，而不是正确的接口。

所以，如果你的软件运行在 VPN 环境中，参考上述章节，设置正确的网络接口。

### 组播被禁用

在一些情况下，运行环境中，无法开启组播。在这种情况下，需要配置合适的 `IP finder`。TCP sockets 发现器  `TcpDiscoveryVmIpFinder` ，或者 Amazon S3 发现器 `TcpDiscoveryS3IpFinder` 。

具体 Ignite 集群配置，可以参考文档 [Apache Ignite](https://www.zybuluo.com/liyuj/note/230739)。

### 开启日志

在排除故障时，开启 Ignite 日志，将会给予很大的帮助。在 `classpath` 中添加 `vertx-default-jul-logging.properties` 文件（默认的JUL记录时），这是一个标准 java.util.loging（JUL） 配置文件。具体配置如下：

```
org.apache.ignite.level=INFO
java.util.logging.ConsoleHandler.level=INFO
java.util.logging.FileHandler.level=INFO
```
