# Hazelcast Cluster Manager

> 翻译：Ranger Tsao，校对 宋子豪、赵亮

`HazelcastClusterManager`是Vert.x中集群管理器基于[Hazelcast](http://hazelcast.org)的一个实现。 它是Vert.x集群管理器的默认实现，也可轻易切换至其它的实现，`HazelcastClusterManager`的maven坐标为：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-hazelcast</artifactId>
  <version>3.2.1</version>
</dependency>
```

## 使用`HazelcastClusterManager`

如果通过命令行来使用Vert.x，对应集群管理器`jar`包(`vertx-hazelcast-${version}`)应该在Vert.x中安装包中。

如果在`Maven`或者`Gradle`工程中使用Vert.x，只需要在工程依赖中加上相应的`ClusterManager`实现依赖：`io.vertx:vertx-hazelcast:${version}`。

如果`vertx-hazelcast-${version}`在`classpath`中，Vert.x将自动检测到，并将其作为集群管理。需要注意的是，要确保Vert.x的`classpath`中没有其它的`ClusterManager`实现`jar`包。

当然在内嵌Vert.x时，通过编程的方式创建Vert.x实例，调用`setClusterManager`方法显式指定集群管理器。

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

## 配置`HazelcastClusterManager`

通常情况下，集群管理器的相关配置是由打包的jar中的默认配置文件`cluster.xml`决定的。

如果要覆盖此配置，可以在`classpath`中添加一个`cluster.xml`文件。此配置文件必须是一个`Hazelcast`配置文件，在[Hazelcast](http://hazelcast.org)的官方网站，可以找到具体的配置描述。

同时也可以通过编程的形式来指定相应的配置：

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

## 使用已存在的Hazelcast集群

可以在集群管理器通过设置`HazelcastInstance`来重用现有的集群：

```java
HazelcastInstance instance = HazelcastClient.newHazelcastClient(
  new ClientConfig()
    .setGroupConfig(new GroupConfig("groupname","password")
    .setNetworkConfig(new ClientNetworkConfig().addAddress("hosts"))));//创建HazelcastClient
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

在这种情况下，vert.x不是Hazelcast群集的所有者，所以不要关闭Vert.x关闭Hazlecast集群。

请注意，自定义Hazelcast实例需要配置：

```xml
<map name="__vertx.subs">
  <backup-count>1</backup-count>
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

## 故障排除

如果默认的广播配置不能正常运行，通常有以下原因：

### 机器禁用广播

MacOS默认禁用广播。Google一下启用广播。

### 使用错误的网络接口

如果机器上上有多个网络接口（也有可能是在运行VPN的情况下），那么Hazelcast很有可能是使用了错误的网络接口。

告诉Hazelcast使用特定的接口，在配置文件中将`interface`设置为指定IP地址，同时确保enabled属性设置为true。 例如：

```xml
<interfaces enabled="true">
  <interface>192.168.1.20</interface>
</interfaces>
```

当运行集群模式时，需要确保Vert.x使用正确的网络接口。当通过命令行模式时，可以设置`cluster-host`参数：

```
vertx run myverticle.js -cluster -cluster-host your-ip-address
```

其中`your-ip-address`必须与Hazelcast中的配置保持一致。

当通过编程模式使用Vert.x时，可以调用方法`setClusterHost`来设置参数

### 使用VPN

VPN软件通常通过创建不支持广播虚拟网络接口来进行工作。如果有一个VPN运行，如果hazelcast与Vert.x不正确配置的话，VPN接口将被选择，而不是正确的接口。

所以，如果你有一个VPN运行，参考上述章节，设置正确的网络接口。

### 广播不可用

在某些情况下，因为特殊的运行环境，可能无法使用多播。在这种情况下，应该配置其他网络传输，例如在TCP上使用TCP套接字，在亚马逊云上使用EC2。

有关Hazelcast更多传输方式，以及如何配置它们，请咨询Hazelcast文档。

### 开启日志

在排除故障时，开启hazelcast日志，将会给予很大的帮助。在classpath中添加`vertx-default-jul-logging.properties`文件（默认的JUL记录时），这是一个标准java.util.loging（JUL）配置文件。具体配置如下：

```
com.hazelcast.level=INFO
java.util.logging.ConsoleHandler.level=INFO
java.util.logging.FileHandler.level=INFO
```
