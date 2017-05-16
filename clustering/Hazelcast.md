# Hazelcast Cluster Manager

> 翻译：Ranger Tsao，校对 宋子豪、赵亮

`HazelcastClusterManager` 是基于 [Hazelcast](http://hazelcast.org) 实现 ,是Vert.x 中集群管理器中的默认实现。由于 Vert.x 集群管理的可插拔性，也可轻易切换至其它的集群管理器。

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-hazelcast</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(build.gradle)

```groovy
compile 'io.vertx:vertx-hazelcast:3.4.1'
```

Vert.x 集群管理器包含一下几个功能：

1. 发现并管理集群中的节点
2. 管理集群端的主题订阅清单（这样就可以轻松得知集群中的那些节点订阅了那些 EventBus 地址）
3. 分布式 Map 支持
4. 分布式锁
5. 分布式计数器

**注意事项**
Vert.x 集群器并不处理节点之间的通信，在 Vert.x 中节点中的通信是直接由 TCP 链接处理的。

## 使用 Hazelcast cluster manager

如果通过命令行来使用 Vert.x，对应集群管理器 `jar` 包( `vertx-hazelcast-${version}` )应该在 Vert.x 中安装包中。

如果在 `Maven` 或者 `Gradle` 工程中使用 Vert.x ，只需要在工程依赖中加上相应的 `ClusterManager` 实现依赖：`io.vertx:vertx-hazelcast:${version}`。

如果 `vertx-hazelcast-${version}` 在 `classpath` 中，Vert.x将自动检测到，并将其作为集群管理。需要注意的是，要确保 Vert.x 的 `classpath` 中没有其它的 `ClusterManager` 实现 `jar` 包。

当然在内嵌 Vert.x 时，通过编程的方式创建 Vert.x 集群模式实例，调用 `setClusterManager` 方法显式指定集群管理器。

```java
ClusterManager mgr = new HazelcastClusterManager();//创建ClusterManger对象

VertxOptions options = new VertxOptions().setClusterManager(mgr);//设置到Vertx启动参数中

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

## 配置 Hazelcast cluster manager

通常情况下，集群管理器的相关配置是由打包的jar中的默认配置文件 `default-cluster.xml` 决定的。

> `default-cluster.xml`  还是下面需要提到的 `cluster.xml` 必须是一个 `Hazelcast` 配置文件，在 [Hazelcast](http://hazelcast.org) 的官方网站，可以找到具体的配置描述。

如果要覆盖此配置，可以在 `classpath` 中添加一个 `cluster.xml` 文件。如果想在 fat jar 中内嵌 `cluster.xml` ，此文件必须在 fat jar 的根目录中。如果此文件是一个外部文件，则必须将其添加至 `classpath` 中。举个例子：

```bash
# cluster.xml 在当前路径中
java -jar ... -cp . -cluster
vertx run MyVerticle -cp . -cluster

# cluster.xml 在 conf 目录中
java -jar ... -cp conf -cluster
```

还有一种方式来覆盖默认的配置文件，那就是利用系统配置 `vertx.hazelcast.config` 来实现：

```bash
# 指定一个外部文件为自定义配置文件
java -Dvertx.hazelcast.config=./config/my-cluster-config.xml -jar ... -cluster

# 从 classpath 中加载一个文件为自定义配置文件
java -Dvertx.hazelcast.config=classpath:my/package/config/my-cluster-config.xml -jar ... -cluster
```

如果 `vertx.hazelcast.config` 值不为空时，将覆盖 `classpath` 中所有的 `cluster.xml` 文件，但是如果加载 `vertx.hazelcast.config` 失败时，系统将选取 `classpath` 任意一个 `cluster.xml` ，甚至直接使用默认配置。

> 注意：Vert.x 并不支持 -Dhazelcast.config 设置方式，请不要使用。

同时也可以通过编程的形式达到配置的目的：

```java
Config hazelcastConfig = new Config(); //创建hazelcast配置

// 设置相关的hazlcast配置，在这里省略掉，不再赘述

ClusterManager mgr = new HazelcastClusterManager(hazelcastConfig);

VertxOptions options = new VertxOptions().setClusterManager(mgr);

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

Hazelcast支持多种不同的传输协议，包括组播和TCP。默认配置中采用组播传输协议，因此您必须在网络上启用组播才能使其工作。

具体详细配置，请参阅 [Hazelcast](http://hazelcast.org) 文档。

## 使用已存在的 Hazelcast 集群

可以在集群管理器通过设置 `HazelcastInstance` 来复用现有集群：

```java
HazelcastInstance instance = HazelcastClient.newHazelcastClient(
  new ClientConfig()
    .setGroupConfig(new GroupConfig("groupname","password")
    .setNetworkConfig(new ClientNetworkConfig().addAddress("hosts")))); //创建HazelcastClient
ClusterManager mgr = new HazelcastClusterManager(hazelcastInstance);
VertxOptions options = new VertxOptions().setClusterManager(mgr);
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

在这种情况下，Vert.x不是 Hazelcast 群集的所有者，所以不要关闭 Vert.x 时关闭 Hazlecast 集群。

请注意，自定义 Hazelcast 实例需要配置：

```xml
<properties>
  <property name="hazelcast.shutdownhook.enabled">false</property>
</properties>
<multimap name="__vertx.subs">
  <backup-count>1</backup-count>
</multimap>
<map name="__vertx.haInfo">
  <time-to-live-seconds>0</time-to-live-seconds>
  <max-idle-seconds>0</max-idle-seconds>
  <eviction-policy>NONE</eviction-policy>
  <max-size policy="PER_NODE">0</max-size>
  <eviction-percentage>25</eviction-percentage>
  <merge-policy>com.hazelcast.map.merge.LatestUpdateMapMergePolicy</merge-policy>
</map>
<semaphore name="__vertx.*">
  <initial-permits>1</initial-permits>
</semaphore>
```

**重要提醒**

- 当 Vert.x 集群使用 HA（高可用或故障转移）时，请不要使用 Hazelcast 客户端，因为他们不会通知他们何时离开集群，同时有可能丢失数据，还有可能将集群置于不一致的状态。更多情况请翻阅 [Issue 24](https://github.com/vert-x3/vertx-hazelcast/issues/24)
- 同时要确保 Hazelcast 集群 先于 Vert.x 集群启动，后于 Vert.x 集群关闭。同时需要禁用 `shutdownhook`。参考上述的 xml 配置，或者通过 系统变量来实现。

## 使用 Hazelcast async methods

Hazelcast 中的 `IMap` 、 `IAtomicLong` 接口(数据结构) 均有异步调用方法，其返回值为 `ICompletableFuture<V>`，这与 Vert.x 的线程模型完美契合。但是即使这些接口已经存在一段时间，却没有通过 `HazelcastInstance` 公共 API 暴露。

默认情况下，`HazelcastClusterManager` 使用公共 API。当在程序启动时，设置选项`-Dvertx.hazelcast.async-api=true` ，将代表系统在与 Hazelcast 集群通讯交互时，将采用 Hazelcast async API 。这意味着，`Counter` 计数操作、`AsyncMap`的 `get` `put` `remove` 操作都将通过 Vert.x EventLoop 线程来执行，而不是通过 Woker 线程的 `vertx.executeBlocking` 执行。

## 故障排除

如果默认的组播配置不能正常运行，通常有以下原因：

### 机器禁用组播

MacOS 默认禁用组播。Google一下启用组播。

### 使用错误的网络接口

如果机器上有多个网络接口（也有可能是在运行 VPN 的情况下），那么 Hazelcast 很有可能是使用了错误的网络接口。

为了确保 Hazelcast 使用正确的网络接口，在配置文件中将 `interface` 设置为指定IP地址，同时确保 enabled 属性设置为 true 。 例如：

```xml
<interfaces enabled="true">
  <interface>192.168.1.20</interface>
</interfaces>
```

当运行集群模式时，需要确保 Vert.x 使用正确的网络接口。当通过命令行模式时，可以设置 `cluster-host` 参数：

```bash
vertx run myverticle.js -cluster -cluster-host your-ip-address
```

其中 `your-ip-address` 必须与 Hazelcast 中的配置保持一致。

当通过编程模式使用 Vert.x 时，可以调用方法 `setClusterHost` 来设置参数

### 使用VPN

VPN 软件通常通过创建不支持组播的虚拟网络接口来进行工作。在 VPN 环境中，如果 Hazelcast 与 Vert.x 不正确配置的话，VPN 接口将被选择，而不是正确的接口。

所以，如果你的软件运行在 VPN 环境中，参考上述章节，设置正确的网络接口。

### 组播不可用

在某些情况下，因为特殊的运行环境，可能无法使用组播。在这种情况下，应该配置其他网络传输，例如在 TCP 上使用 TCP 套接字，在亚马逊云上使用 EC2 。

有关 Hazelcast 更多传输方式，以及如何配置它们，请咨询 Hazelcast 文档。

### 开启日志

在排除故障时，开启 Hazelcast 日志，将会给予很大的帮助。在 `classpath` 中添加 `vertx-default-jul-logging.properties` 文件（默认的JUL记录时），这是一个标准 java.util.loging（JUL） 配置文件。具体配置如下：

```
com.hazelcast.level=INFO
java.util.logging.ConsoleHandler.level=INFO
java.util.logging.FileHandler.level=INFO
```

## Hazelcast 日志配置

Hazelcast 的日志默认采用 JDK 实现（参考 JUL）。如果想切换至其他日志库，通过设置 `azelcast.logging.type` 即可达到目的。

```bash
-Dhazelcast.logging.type=slf4j
```

详细文档请参考 [hazelcast documentation](http://docs.hazelcast.org/docs/3.6.1/manual/html-single/index.html#logging-configuration)。

## 使用其他 Hazelcast 版本

当前的 Vert.x HazelcastClusterManager 使用的 Hazelcast 版本为 `3.6.3` 。如果开发者想使用其他版本的 Hazelcast，需要做以下工作：

- 将目标版本的 Hazelcast 依赖添加至 classpath 中 
- 如果是 fat jar 的形式，在构建工具中使用正确的版本

参考代码如下：

- Maven(pom.xml)

```xml
<dependency>
  <groupId>com.hazelcast</groupId>
  <artifactId>hazelcast</artifactId>
  <version>ENTER_YOUR_VERSION_HERE</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-hazelcast</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(build.gradle)

```groovy
dependencies {
 compile ("io.vertx:vertx-hazelcast:3.4.1"){
   exclude group: 'com.hazelcast', module: 'hazelcast'
 }
 compile "com.hazelcast:hazelcast:ENTER_YOUR_VERSION_HERE"
}
```