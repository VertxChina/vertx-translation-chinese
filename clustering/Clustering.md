# Vert.x Clustering

> 翻译：Ranger Tsao，校对 宋子豪、赵亮

在Vert.x中，集群化与高可用均是开箱即用的。集群群组管理器继承了集群管理器，是一个可插拔的实现。在Vert.x中，采用[Hazelcast](http://hazelcast.org)作为默认的集群管理器。

## 集群管理器

在Vert.x中，集群管理器有如下作用：

1. 发现并管理集群中的节点
2. 维护集群端主题订阅列表（这样Vert.x便能轻松的知道节点与时间总线的关系）
3. 支持分布式Map
4. 分布式锁
5. 分布式计数器

集群管理器并不负责事件总线在节点中的传输，这传输是直接由Vert.x通过TCP完成的。

在Vert.x分布式中，采用[Hazelcast](http://hazelcast.org)作为默认的集群管理器，因为集群管理器的实现是可插拔的，因此也可以轻易的换成其他的实现。

Vert.x的集群管理器必须实现[ClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)接口。Vert.x通过[Java Service Loader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)在`classpath`中定位[ClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)实例，从而在运行时，定位到具体的集群管理器。

如果通过命令行来使用Vert.x，又想使用集群化的话，就必须确保Vert.x的运行包的lib目录中必须有相应的`ClusterManager`实现jar文件.

如果在`Maven`或者`Gradle`工程中使用Vert.x，只需要在工程依赖中加上相应的`ClusterManager`实现依赖即可。

当然在内嵌Vert.x是，也可以通过`setClusterManager`显式指定集群管理器。

### 集群管理器实现

目前Vert.x提供了两个集群管理器实现

#### Hazelcast

基于[Hazelcast](http://hazelcast.org)实现，是一个默认的集群管理器

[文档手册]|(/clustering/Hazelcast.md)，[源代码](https://github.com/vert-x3/vertx-hazelcast)

#### JGroups

基于[JGroups](http://jgroups.org)实现的，目前还处于**技术预览版**

[文档手册]|(/clustering/JGroups.md)，[源代码](https://github.com/vert-x3/vertx-jgroups)
