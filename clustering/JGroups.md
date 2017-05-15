> 从 Vert.x 3.4.0 开始，Vert.x 已经弃用 JGoups 实现，已经由 [infinispan]|(/clustering/Infinispan.md) 。**不建议在生产或测试环境中使用**

# JGroups Cluster Manager

在构建工具中添加依赖即可：

- Maven(pom.xml)

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-jgroups</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(build.gradle)

```
compile 'io.vertx:vertx-jgroups:3.4.1'
```

如果通过命令行来使用Vert.x，jar包`vertx-jgroups-3.4.1.jar`，应该在Vert.x中安装包中。同时也要将`jgroups` 的jar加到lib目录中。

如果 `vertx-jgroups-3.4.1.jar` 在 `classpath` 中，Vert.x将自动检测到，并将其作为集群管理。需要注意的是，要确保Vert.x的 `classpath` 中没有其它的 `ClusterManager` 实现 `jar` 包。也可以通过设置启动参数: `-Dvertx.clusterManagerFactory=io.vertx.spi.cluster.jgroups.JGroupsClusterManager` ,来指定集群管理器。

同 [hazlcast]|(/clustering/Hazelcast.md)，也可以通过编程的方式来实现：

```java
ClusterManager mgr = new JGroupsClusterManager();
VertxOptions options = new VertxOptions().setClusterManager(mgr);
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
  } else {
    // failed!
  }
});
```

### 配置

JGroups 的配置文件 `default-jgroups.xml` 如下：

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="urn:org:jgroups"
xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/jgroups.xsd">

<UDP
  mcast_port="${jgroups.udp.mcast_port:45588}"
  ip_ttl="0"
  tos="8"
  ucast_recv_buf_size="5M"
  ucast_send_buf_size="5M"
  mcast_recv_buf_size="5M"
  mcast_send_buf_size="5M"
  max_bundle_size="64K"
  max_bundle_timeout="30"
  enable_diagnostics="true"
  thread_naming_pattern="cl"

  timer_type="new3"
  timer.min_threads="4"
  timer.max_threads="10"
  timer.keep_alive_time="3000"
  timer.queue_max_size="500"

  thread_pool.enabled="true"
  thread_pool.min_threads="2"
  thread_pool.max_threads="8"
  thread_pool.keep_alive_time="5000"
  thread_pool.queue_enabled="true"
  thread_pool.queue_max_size="100000"
  thread_pool.rejection_policy="discard"

  oob_thread_pool.enabled="true"
  oob_thread_pool.min_threads="1"
  oob_thread_pool.max_threads="8"
  oob_thread_pool.keep_alive_time="5000"
  oob_thread_pool.queue_enabled="false"
  oob_thread_pool.queue_max_size="100"
  oob_thread_pool.rejection_policy="discard"/>

<PING/>
<MERGE3 max_interval="30000" min_interval="10000"/>
<FD_SOCK/>
<FD_ALL/>
<VERIFY_SUSPECT timeout="1500"/>
<BARRIER/>
<pbcast.NAKACK2 xmit_interval="500"
  xmit_table_num_rows="100"
  xmit_table_msgs_per_row="2000"
  xmit_table_max_compaction_time="30000"
  max_msg_batch_size="500"
  use_mcast_xmit="true"
  discard_delivered_msgs="true"/>

<UNICAST3 xmit_table_num_rows="100" xmit_table_msgs_per_row="1000"
  xmit_table_max_compaction_time="30000" max_msg_batch_size="500"/>

<pbcast.STABLE stability_delay="1000" desired_avg_gossip="50000"
  max_bytes="8m"/>

<pbcast.GMS print_local_addr="true" join_timeout="2000"
  view_bundling="true"/>
<UFC max_credits="2M"min_threshold="0.4"/>
<MFC max_credits="2M" min_threshold="0.4"/>
<FRAG2 frag_size="60K"/>
<pbcast.STATE_TRANSFER/>

<COUNTER/>
<CENTRAL_LOCK use_thread_id_for_lock_owner="false"/>
</config>
```

`default-jgroups.xml` 内嵌在对应jar包中。

如果要覆盖此配置，可以在 `classpath` 中添加一个 `jgroups.xml` 文件。此配置文件必须是一个 `JGroups` 配置文件，在 [JGroups](http://jgroups.org) 的官方网站中，可以找到具体的配置描述。


JGroups支持多种传播方式，包括多播与TCP。默认配置时多播方式，所以需要确保网络是否启用多播。

## 故障排除

如果默认的多播配置不能正常运行，通常有以下原因：

### 机器禁用多播

当使用UDP时，IP多播是必需的，在一些系统中，多播路由需要被添加到路由表中，否则，缺省路由将被使用，请注意，有些系统并不适用路由表中的IP组播路由，只为单播路由

Mac系统例子：

```
# Adds a multicast route for 224.0.0.1-231.255.255.254
sudo route add -net 224.0.0.0/5 127.0.0.1

# Adds a multicast route for 232.0.0.1-239.255.255.254
sudo route add -net 232.0.0.0/5 192.168.1.3
```

### 错误配置的IPV6

默认情况下，JVM 使用 IPv6，但路由表配置不正确，或使用 IPv4 解决方法：查看 IPv6 路由或强制使用 IPv4 （`-Djava.net.preferIPv4Stack=TRUE`）。有关这方面更多的细节可在https://developer.jboss.org/wiki/IPv6。

### 使用错误的网络接口

如果机器上上有多个网络接口（也有可能是在运行VPN的情况下），那么JGroups很有可能是使用了错误的网络接口。

配置参数 `jgroups.bind_addr` 用来确定绑定的网络接口, 例如：`jgroups.bind_addr=192.168.1.5`。

下面这些配置参数同样有效：

- `global`:挑选一个可用的全局地址。如果不能，使用 `site_local` 地址。
- `site_local`:挑选一个本地（非路由）的IP地址，例如从192.168.0.0或10.0.0.0地址
- `link_local`:挑选一个链接本地IP地址从169.254.1.0到169.254.254.255
- `non_loopback`:挑选任何非回送地址
- `loopback`:挑选一个环回地址，例如127.0.0.1
- `match-interface`:挑选任何匹配网络接口名的地址，例如匹配接口：ETH\ *
- `match-host`:挑选任何符合域名规则的地址，例如：linux.\*
- `match-address`:挑选任务符合IP地址规则的地址，例如：192.168.\*

当运行集群模式时，需要确保Vert.x使用正确的网络接口。当通过命令行模式时，可以设置`cluster-host`参数：

```
vertx run myverticle.js -cluster -cluster-host your-ip-address
```

其中 `your-ip-address` 必须与 JGroups 中的配置保持一致。

当通过编程模式使用 Vert.x 时，可以调用方法 `setClusterHost` 来设置参数

### 使用VPN

VPN软件通常通过创建不支持多播虚拟网络接口来进行工作。如果有一个 VPN 运行，如果 JGroups与 Vert.x 不正确配置的话，VPN接口将被选择，而不是正确的接口。

所以，如果你运行在 VPN 环境中，参考上述章节，设置正确的网络接口。


### 多播不可用

在某些情况下，因为特殊的运行环境，可能无法使用多播。在这种情况下，应该配置其他网络传输，例如在 TCP 上使用 TCP 套接字，在亚马逊云上使用 EC2。

有关 JGroups 更多传输方式，以及如何配置它们，请咨询JGroups文档。

### 开启日志

在排除故障时，开启JGroups日志，将会给予很大的帮助。在 classpath 中添加 `vertx-default-jul-logging.properties` 文件（默认的JUL记录时），这是一个标准 java.util.loging（JUL）配置文件。具体配置如下：

```
org.jgroups.level=INFO
java.util.logging.ConsoleHandler.level=INFO
java.util.logging.FileHandler.level=INFO
```
