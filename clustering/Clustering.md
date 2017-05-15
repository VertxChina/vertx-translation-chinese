# Vert.x Clustering

在 Vert.x 中，集群化与高可用均是开箱即用的。Vert.x 通过可插拔的集群管理器（cluster manager）来实现集群管理。在 Vert.x 中，采用 [Hazelcast](http://hazelcast.org) 作为默认的集群管理器。

## 集群管理器

集群管理器的介绍见 [Vert.x Core 文档手册中的相关章节](http://vertx.io/docs/vertx-core/java/#_cluster_managers)。

## 集群管理器实现

目前 Vert.x 提供了四种集群管理器实现：

- [Hazelcast](http://hazelcast.org)（默认实现）
  - [文档手册](Hazelcast.md)
  - [组件源码](https://github.com/vert-x3/vertx-hazelcast)
- [Infinispan](http://infinispan.org/)（预览版）
  - [文档手册](Infinispan.md)
  - [组件源码](https://github.com/vert-x3/vertx-infinispan)
- [Apache Ignite](http://ignite.apache.org/)
  - [文档手册](ApacheIgnite.md)
  - [组件源码](https://github.com/vert-x3/vertx-ignite)
- [Apache Zookeeper](http://zookeeper.apache.org/)（预览版）
  - [文档手册](ApacheZookeeper.md)
  - [组件源码](https://github.com/vert-x3/vertx-zookeeper)
