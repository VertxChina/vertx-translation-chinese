# Infinispan Cluster Manager

> 请注意，当前版本还处于预览版，请慎重在生产环境中使用
> 翻译：Ranger Tsao，校对 宋子豪、赵亮

`InfinispanClusterManager` 是基于 [Infinispan](http://infinispan.org/) 实现。由于 Vert.x 集群管理的可插拔性，也可轻易切换至其它的集群管理器。

`InfinispanClusterManager` 在组件 `vertx-infinispan` 中，通过构建工具可以轻松引入：

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-infinispan</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(gradle.xml)

```groovy
compile 'io.vertx:vertx-infinispan:3.4.1'
```

Vert.x 集群管理器包含以下几个功能：

1. 发现并管理集群中的节点
2. 管理集群端的主题订阅清单（这样就可以轻松得知集群中的那些节点订阅了那些 EventBus 地址）
3. 分布式 Map 支持
4. 分布式锁
5. 分布式计数器

**注意事项**
Vert.x 集群器并不处理节点之间的通信，在 Vert.x 中节点中的通信是直接由 TCP 链接处理的。

## 使用 Infinispan cluster manager

Vert.x 能够从 classpath 路径的 jar 自动检测并使用出 `ClusterManager` 的实现。不过需要确保在 classpath 没有其他的 `ClusterManager` 实现。

### 通过命令行使用

确保 `vertx-infinispan-3.4.1.jar` 在 Vert.x 的安装路径中的 lib 目录下。

### 通过 Maven 或 Gradle 构建工具使用

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-infinispan</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(gradle.xml)

```groovy
compile 'io.vertx:vertx-infinispan:3.4.1'
```

### 编程形式调用

通过编码的形式，设置集群管理器实现，例子：

```java
ClusterManager mgr = new InfinispanClusterManager();//创建 InfinispanClusterManager
VertxOptions options = new VertxOptions().setClusterManager(mgr);// 设置集群管理器
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

## 配置 Infinispan cluster manager

通常情况下，`InfinispanClusterManager` 使用 jar 包中内嵌的两个文件来配置自身属性：

1. [jgroup.xml](https://github.com/vert-x3/vertx-infinispan/blob/master/src/main/resources/jgroups.xml)
2. [infinispan.xml](https://github.com/vert-x3/vertx-infinispan/blob/master/src/main/resources/infinispan.xml)

如果要覆盖其中任意一个配置文件，可以在 `classpath` 中添加对应的文件。如果想在 fat jar 中内嵌相应的配置文件 ，此文件必须在 fat jar 的根目录中。如果此文件是一个外部文件，则必须将其添加至 `classpath` 中。举个例子：

```bash
# infinispan.xml 在当前路径中
java -jar ... -cp . -cluster
vertx run MyVerticle -cp . -cluster

# infinispan.xml 在 conf 目录中
java -jar ... -cp conf -cluster
```

还有一种方式来覆盖默认的配置文件，那就是利用系统配置 `vertx.infinispan.config` 来实现：

```bash
# 指定一个外部文件为自定义配置文件
java -Dvertx.infinispan.config=./config/my-infinispan.xml -jar ... -cluster

# 从 classpath 中加载一个文件为自定义配置文件
java -Dvertx.infinispan.config=my/package/config/my-infinispan.xml -jar ... -cluster
```

如果系统变量 `vertx.infinispan.config` 值不为空时，将覆盖 `classpath` 中所有的 `infinispan.xml` 文件，但是如果加载 `vertx.infinispan.config` 失败时，系统将选取 `classpath` 任意一个 `infinispan.xml` ，甚至直接使用默认配置。

`jgroup.xml` 与 `infinispan.xml` 分别是 JGroups 、 Infinispan 配置文件。在对应的官方可以网站可以详细的配置攻略。

同其他集群管理器，亦可通过编程的形式来进行配置，举例：

```java
ClusterManager mgr = new InfinispanClusterManager("custom-infinispan.xml");//加载自定义配置文件

VertxOptions options = new VertxOptions().setClusterManager(mgr);

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

## 使用已有 Infinispan Cache Manager

开发者可以通过 `DefaultCacheManager` 来复用已经存在的 `cache manager`。

```java
ClusterManager mgr = new InfinispanClusterManager(cacheManager);
VertxOptions options = new VertxOptions().setClusterManager(mgr);
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

在这种情况下，Vert.x 并不是 `cache manager` 的所有者，因此不能在关闭 Vert.x 时，停止 Infinispan 。

需要注意的是，需要通过如下配置来自定义 Infinispan 实例：

```xml
<cache-container default-cache="__vertx.distributed.cache">

  <replicated-cache name="__vertx.subs">
    <expiration interval="-1"/>
  </replicated-cache>

  <replicated-cache name="__vertx.haInfo">
    <expiration interval="-1"/>
  </replicated-cache>

  <distributed-cache name="__vertx.distributed.cache">
    <expiration interval="-1"/>
  </distributed-cache>

</cache-container>
```

## 适配 Openshift 3

为了使 Vert.x 集群能够正常运行在 Openshift 3，需要在配置以及依赖上做一些改动。

首先，添加 JGroups `KUBE_PING` 协议依赖：

```xml
<dependency>
  <groupId>org.jgroups.kubernetes</groupId>
  <artifactId>kubernetes</artifactId>
  <version>0.9.0</version>
  <exclusions>
    <exclusion>                                          (1)
      <artifactId>undertow-core</artifactId>
      <groupId>io.undertow</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

> 1. 去掉 `undertow-core` 依赖, 确保 `KUBE_PING` 能够在 JDK Http server 正常运行

然后覆盖默认的 JGroups 配置，确保 `KUBE_PING` 成为其发现协议，代码如下：

```xml
<config xmlns="urn:org:jgroups"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/JGroups-3.6.xsd">

  <TCP bind_addr="${jgroups.tcp.address:match-interface:eth.*}"
       bind_port="${jgroups.tcp.port:7800}"
       enable_diagnostics="false"
       thread_naming_pattern="pl"
       send_buf_size="640k"
       sock_conn_timeout="300"
       bundler_type="transfer-queue"

       thread_pool.min_threads="${jgroups.thread_pool.min_threads:2}"
       thread_pool.max_threads="${jgroups.thread_pool.max_threads:30}"
       thread_pool.keep_alive_time="60000"
       thread_pool.queue_enabled="false"

       internal_thread_pool.min_threads="${jgroups.internal_thread_pool.min_threads:5}"
       internal_thread_pool.max_threads="${jgroups.internal_thread_pool.max_threads:20}"
       internal_thread_pool.keep_alive_time="60000"
       internal_thread_pool.queue_enabled="true"
       internal_thread_pool.queue_max_size="500"

       oob_thread_pool.min_threads="${jgroups.oob_thread_pool.min_threads:20}"
       oob_thread_pool.max_threads="${jgroups.oob_thread_pool.max_threads:200}"
       oob_thread_pool.keep_alive_time="60000"
       oob_thread_pool.queue_enabled="false"
  />
  <kubernetes.KUBE_PING
  />
  <MERGE3 min_interval="10000"
            max_interval="30000"
  />
  <FD_SOCK/>
  <FD_ALL timeout="60000"
           interval="15000"
           timeout_check_interval="5000"
  />
  <VERIFY_SUSPECT timeout="5000"/>
  <pbcast.NAKACK2 use_mcast_xmit="false"
                  xmit_interval="1000"
                  xmit_table_num_rows="50"
                  xmit_table_msgs_per_row="1024"
                  xmit_table_max_compaction_time="30000"
                  max_msg_batch_size="100"
                  resend_last_seqno="true"
  />
  <UNICAST3 xmit_interval="500"
             xmit_table_num_rows="50"
             xmit_table_msgs_per_row="1024"
             xmit_table_max_compaction_time="30000"
             max_msg_batch_size="100"
             conn_expiry_timeout="0"
  />
  <pbcast.STABLE stability_delay="500"
                 desired_avg_gossip="5000"
                 max_bytes="1M"
  />
  <pbcast.GMS print_local_addr="false"
              join_timeout="${jgroups.join_timeout:5000}"
  />
  <MFC max_credits="2m"
        min_threshold="0.40"
  />
  <FRAG3/>
</config>
```

`KUBE_PING` 默认监听 8080 端口, 因此在构建容器镜像时，需要显式声明：

```bash
EXPOSE 8888
```

同样需要设置项目命名空间作为发现范围：

```bash
ENV OPENSHIFT_KUBE_PING_NAMESPACE my-openshift3-project
```

然后，通过设置系统变量，确保 JVM 采用 `IPv4`

```bash
-Djava.net.preferIPv4Stack=true
```

最后, 所有的设置需要一个服务账号：

```bash
oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default -n $(oc project -q)
```

有关更多配置详情，请参考 [Kubernetes discovery protocol for JGroups](https://github.com/jgroups-extras/jgroups-kubernetes) 。

## 适配 Docker Compose

确认 JVM 在启动时 设置了下面两项配置：

```
-Djava.net.preferIPv4Stack=true -Djgroups.tcp.address=NON_LOOPBACK
```
通过上述两项系统配置，JGroups 才能正确的挑选出 Docker 创建的虚拟网络接口。

## 故障排除

### 组播未正常开启

MacOS 默认禁用组播。Google一下启用组播。

### 使用错误的网络接口

如果机器上有多个网络接口（也有可能是在运行 VPN 的情况下），那么 JGroups 很有可能是使用了错误的网络接口。

为了确保 JGroups 使用正确的网络接口，在配置文件中将 `bind_addr` 设置为指定IP地址。 例如：

```xml
<TCP bind_addr="192.168.1.20" />
<MPING bind_addr="192.168.1.20" />
```

另外，如果需要修改打包好的 `jgoups.xml` 文件，可以通过设置 `jgroups.tcp.address` 系统变量来达到目的

```bash
-Djgroups.tcp.address=192.168.1.20
```

当运行集群模式时，需要确保 Vert.x 使用正确的网络接口。当通过命令行模式时，可以设置 `cluster-host` 参数：

```bash
vertx run myverticle.js -cluster -cluster-host your-ip-address
```

其中 `your-ip-address` 必须与 JGroup 中的配置保持一致。

当通过编程模式使用 Vert.x 时，可以调用方法 `setClusterHost` 来设置参数

### 使用VPN

VPN软件通常通过创建不支持多播虚拟网络接口来进行工作。如果有一个 VPN 运行，如果 JGroups与 Vert.x 不正确配置的话，VPN接口将被选择，而不是正确的接口。

所以，如果你运行在 VPN 环境中，参考上述章节，设置正确的网络接口。

### 组播被禁用

在某些情况下，因为特殊的运行环境，可能无法使用组播。在这种情况下，应该配置其他网络传输协议，例如在 TCP 上使用 `TCPPING` ，在亚马逊云上使用 `S3_PING` 。

有关 JGroups 更多传输方式，以及如何配置它们，请咨询 [JGroups文档](http://www.jgroups.org/manual/index.html#Discovery) 。

### IPv6 错误

如果在 IPv6 地址配置有难点，请强制使用 IPv4:

```bash
-Djava.net.preferIPv4Stack=true
```

## Infinispan 日志配置

Infinispan 依赖与 JBoss Logging 。JBoss Logging 是一个与多种日志框架的桥接器。

JBoss Logging 能够自动检测使用 classpath 中 JARS 中的日志框架实现。

如果在 classpath 有多种日志框架，可以通过设置系统变量 `org.jboss.logging.provider` 来指定具体的实现，例子：

```bash
-Dorg.jboss.logging.provider=log4j2
```

更多配置信息请参考 [JBoss Logging](http://docs.jboss.org/hibernate/orm/4.3/topical/html/logging/Logging.html)

## JGroups 日志配置

JGroups 默认采用 JDK Logging 实现。同时也支持 log4j 与 log4j2 ，如果相应的 jar 包 在 classpath 中。

如果想查阅更详细的信息，或实现自己的日志后端，请参考 [JGroups 日志文档](http://www.jgroups.org/manual/index.html#Logging)