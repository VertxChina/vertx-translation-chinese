# Zookeeper Cluster Manager

> 请注意，当前版本还处于预览版，请慎重在生产环境中使用
> 翻译：Ranger Tsao，校对 宋子豪、赵亮

`ZookeeperClusterManager` 是基于 [Apache Zookeeper](https://zookeeper.apache.org) 实现。由于 Vert.x 集群管理的可插拔性，也可轻易切换至其它的集群管理器。

`ZookeeperClusterManager` 在组件 `vertx-zookeeper` 中，通过构建工具可以轻松引入：

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-zookeeper</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(gradle.xml)

```groovy
compile 'io.vertx:vertx-zookeeper:3.4.1'
```

Vert.x 集群管理器包含以下几个功能：

1. 发现并管理集群中的节点
2. 管理集群端的主题订阅清单（这样就可以轻松得知集群中的那些节点订阅了那些 EventBus 地址）
3. 分布式 Map 支持
4. 分布式锁
5. 分布式计数器

**注意事项**
Vert.x 集群器并不处理节点之间的通信，在 Vert.x 中节点中的通信是直接由 TCP 链接处理的。

## 工作原理

`ZookeeperClusterManager` 使用 [Apache Curator](http://curator.apache.org/) 框架而不是原生Zookeeper 客户端，因此需要依赖 `guava` 、 `slf4j` 、 `zookeeper` 等其他第三方 jar 包。

由于 Zookeeper 采用字典树来存储数据，便可以以 `root` 路径作为命名空间，在默认的 `zookeeper.json` 中定义默认的根路径是 `io.vertx`，同时还有 5 个 子路径用来存储用于管理 Vert.x 集群的相关信息。所有的路径中，只有跟路径可以自定义配置。

各路径作用为：

- /io.vertx/cluster/nodes/ 对应 Vert.x 节点信息
- /io.vertx/asyncMap/$name/ 存储通过接口 `io.vertx.core.shareddata.AsyncMap` 创建的 `AsyncMap` 记录
- /io.vertx/asyncMultiMap/$name/ 存储通过接口 `io.vertx.core.spi.cluster.AsyncMultiMap` 创建的 `AsyncMultiMap` 记录
- /io.vertx/locks/ 存储分布式锁
- /io.vertx/counters/ 存储分布式计数器

## 使用 Zookeeper cluster manager

Vert.x 能够从 classpath 路径的 jar 自动检测并使用出 `ClusterManager` 的实现。不过需要确保在 classpath 没有其他的 `ClusterManager` 实现。

### 通过命令行使用

确保 `vertx-zookeeper-3.4.1.jar` 在 Vert.x 的安装路径中的 lib 目录下。

### 通过 Maven 或 Gradle 构建工具使用

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-zookeeper</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(gradle.xml)

```groovy
compile 'io.vertx:vertx-zookeeper:3.4.1'
```

### 编程形式调用

通过编码的形式，设置集群管理器实现，例子：

```java
ClusterManager mgr = new ZookeeperClusterManager();//创建 ZookeeperClusterManager
VertxOptions options = new VertxOptions().setClusterManager(mgr);// 设置集群管理器
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

## 配置 Zookeeper cluster manager

通常情况下，`ZookeeperClusterManager` 使用 jar 包中内嵌的 [default-zookeeper.json](https://github.com/vert-x3/vertx-zookeeper/blob/master/src/main/resources/default-zookeeper.json) 设置相应的配置。

如果要覆盖此配置，可以在 `classpath` 中添加一个 `zookeeper.json` 文件。如果想在 fat jar 中内嵌 `zookeeper.json` ，此文件必须在 fat jar 的根目录中。如果此文件是一个外部文件，则必须将其添加至 `classpath` 中。举个例子：

```bash
# zookeeper.json 在当前路径中
java -jar ... -cp . -cluster
vertx run MyVerticle -cp . -cluster

# zookeeper.json 在 conf 目录中
java -jar ... -cp conf -cluster
```

还有一种方式来覆盖默认的配置文件，那就是利用系统配置 `vertx.zookeeper.config` 来实现：

```bash
# 指定一个外部文件为自定义配置文件
java -Dvertx.zookeeper.config=./config/my-zookeeper-conf.json -jar ... -cluster

# 从 classpath 中加载一个文件为自定义配置文件
java -Dvertx.zookeeper.config=classpath:my/package/config/my-cluster-config.json -jar ... -cluster
```

如果系统变量 `vertx.zookeeper.config` 值不为空时，将覆盖 `classpath` 中所有的 `zookeeper.json` 文件，但是如果加载 `vertx.zookeeper.config` 失败时，系统将选取 `classpath` 任意一个 `zookeeper.json` ，甚至直接使用默认配置。

在配置文件 `default-zookeeper.json` 中已经通过注释的形式，详细说明每个配置项的作用。

同其他集群管理器，亦可通过编程的形式来进行配置，举例：

```java
JsonObject zkConfig = new JsonObject();
// 设置相关配置项
zkConfig.put("zookeeperHosts", "127.0.0.1");// zk host 地址
zkConfig.put("rootPath", "io.vertx");// 根路径
zkConfig.put("retry", new JsonObject() // 重试策略
    .put("initialSleepTime", 3000)
    .put("maxTimes", 3));

ClusterManager mgr = new ZookeeperClusterManager(zkConfig);
VertxOptions options = new VertxOptions().setClusterManager(mgr);

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

> 注意：通过系统变量 `vertx.zookeeper.hosts` 也可以达到配置 zookeeper `hosts` 的目的。

### 开启日志

在排除故障时，开启 Zookeeper 日志，将会给予很大的帮助。在 `classpath` 中添加 `vertx-default-jul-logging.properties` 文件（默认的JUL记录时），这是一个标准 java.util.loging（JUL） 配置文件。具体配置如下：

```
org.apache.ignite.level=INFO
java.util.logging.ConsoleHandler.level=INFO
java.util.logging.FileHandler.level=INFO
```

## Zookeeper 版本

Vert.x 使用 2.1.1 版本的 Curator ，其使用 3.4.8 版本的 Zookeeper，因此不支持 3.5.x 中的最新特性。