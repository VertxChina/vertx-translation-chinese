# Vert.x Redis

> 原文档：[Vert.x Redis](http://vertx.io/docs/vertx-redis-client/java/)

**Vert.x Redis Client 是 Vert.x 配套的 Redis 客户端实现。**

您可以通过 Vert.x Redis Client 来对 Redis 中的数据进行保存、获取、搜索和删除。Redis 是一个基于BSD协议开源的高性能Key-Value数据库。它可以存储字符串、哈希(hashes)、列表、无序集合(sets)和有序集合(sorted sets)，所以通常被用做结构化数据存储服务器。要使用本组件，您必须有一个运行中的 Redis 实例。

Redis 有着丰富的 API，总结如下：

- Cluster - 与集群相关的命令。要使用这些命令，Redis 服务器的版本必须 >=3.0.0
- Connection - 切换 DB、建立连接、断开连接、授权相关的命令
- Hashes - 操作 hash 相关的命令
- HyperLogLog - 使用 HyperLogLog 算法来快速计算元素基数的相关命令
- Keys - 操作 Redis key 的相关命令
- List - 操作 List 的相关命令
- Pub/Sub - 发布／订阅模式的消息队列相关命令
- Scripting - 解释运行 Lua 脚本的相关命令
- Server - 管理 Redis 服务器的相关命令
- Sets - 操作无序集合的相关命令
- Sorted Sets - 操作有序集合的相关命令
- Strings - 操作字符串的相关命令
- Transactions - 处理事务的相关命令

## 使用 Vert.x Redis

要使用 Vert.x Redis 客户端，需要添加下列依赖：

- Maven (在 `pom.xml` 文件中):

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-redis-client</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-redis-client:3.4.1'
```

## 连接 Redis 服务器

要连接到 Redis 服务器，需要配置一些参数。配置参数包装在 [`RedisOptions`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisOptions.html) 对象中，有以下这些参数：

- `host`: 默认是 `localhost`
- `port`:  默认是 `6379`
- `encoding`:  默认是 `UTF-8`
- `tcpKeepAlive`:  默认是 `true`
- `tcpNoDelay`:  默认是 `true`

例如：

```java
RedisOptions config = new RedisOptions()
    .setHost("127.0.0.1");

RedisClient redis = RedisClient.create(vertx, config);
```

Vert.x Redis 客户端在连接 Redis 服务器失败时，会尝试重连。所以，如果您连接的 Redis 服务器需要授权，或者您连接的 Redis 服务器的 DB 不是默认 DB，您需要提供授权的密码，或者 DB 的 ID，配置参数如下：

- `auth`
- `select`

如果您不这样做，而是手动调用  [`auth`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html#auth-java.lang.String-io.vertx.core.Handler-) 或者 [`select`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html#select-int-io.vertx.core.Handler-) 方法，那么当出现 socket 错误的时候，Vert.x Redis 客户端将不知道如何修复错误。

## 执行命令

当使用 Vert.x Redis 客户端连接到 Redis 服务器后，就可以使用 Vert.x Redis 客户端执行 Redis 命令。Vert.x Redis 客户端为执行 Redis 命令提供了一套十分简洁的 API，省去了手写命令的麻烦。例如，如果想通过 key 获得 value的话，可以这样：

```java
RedisClient redis = RedisClient.create(vertx, new RedisOptions());

redis.get("mykey", res -> {
  if (res.succeeded()) {
    // so something...
  }
});
```

要想了解更多 Redis 命令，请参考 [Redis 官方文档](http://redis.io/commands)。

## 发布／订阅模式

Redis 支持发布／订阅模式的消息队列。需要注意，一旦一个 Redis 连接被注册为订阅者时，此连接将不能再执行其他命令，直到这个连接使用取消订阅者的命令。

可以这样注册一个订阅者：

```java
vertx.eventBus().<JsonObject>consumer("io.vertx.redis.channel1", received -> {
  // 操作接收到的消息
  JsonObject value = received.body().getJsonObject("value");
  // value 是一个包含下面数据的 JSON 对象
  // channel - 发布消息的 channel
  // pattern - 若您使用匹配符来匹配消息 channel，Pattern 代表的是匹配符
  // message - 发布的消息载体
});

RedisClient redis = RedisClient.create(vertx, new RedisOptions());

redis.subscribe("channel1", res -> {
    if (res.succeeded()) {
        // so something...
    }
});
```

发布消息：

```java
RedisClient redis = RedisClient.create(vertx, new RedisOptions());

redis.publish("channel1", "Hello World!", res -> {
    if (res.succeeded()) {
        // so something...
    }
});
```

## 更简洁的 hash 命令

大部分 Redis 命令使用单个字符串或者字符串数组来作为参数，并且返回的也是单个字符串或者字符串数组。不过在处理 hash 值时，有其他一些更简洁的方式。

### hgetall 命令

`hgetall` 命令返回的结果将被转换成 JSON 对象。这样，您就可以使用 JSON 的语法来进行交互，这在与 Event Bus 通信时十分方便。

### mset 命令

您可以向 `mset` 命令传入一个 JSON 对象以在 hash 中设置多个值。需要注意 key 和 value 都将被转换成字符串。

```text
{
  keyName: "value",
  otherKeyName: "other value"
}
```

### msetnx 命令

您可以向 `msetnx` 命令传入一个 JSON 对象以在 hash 中设置多个值（译者注：msetnx 命令，必须当且仅当所有给定 key 都不存在）。需要注意 key 和 value 都将被转换成字符串。

```text
{
  keyName: "value",
  otherKeyName: "other value"
}
```

### hmset 命令

您可以向 `hmset` 命令传入一个 JSON 对象以在 hash 中设置多个值（译者注：hmset 命令，如果给定 key 不存在，将创建新的 key）。需要注意 key 和 value 都将被转换成字符串。

```text
{
  keyName: "value",
  otherKeyName: "other value"
}
```

### zadd 命令

调用 [`zaddMany`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html#zaddMany-java.lang.String-java.util.Map-io.vertx.core.Handler-) 方法可以同时向有序集合中添加多个member。需要注意 key 和 value 都将被转换成字符串。

> 译者注：在 Vert.x Redis Client 中，`zadd` 方法和 `zaddMany` 方法都对应 Redis 中的 `zadd` 命令。不同之处在于，`zaddMany` 方法可以添加多个 score-member。

> 译者注：实际在 `zaddMany` 方法中，传入的是 `Map<String, Double>` 类型的参数。

```text
{
  score: "member",
  otherScore: "other member"
}
```

## 服务器信息

为让返回的服务器信息易于操作，Vert.x Redis 客户端将会把服务器信息转换成利于理解的 JSON 格式。格式为：JSON 对象的每个部分都包装着属于着这部分的属性。不在这个部分的属性，将会以其他的顶级对象部分展现：

```text
{
  server: {
    redis_version: "2.5.13",
    redis_git_sha1: "2812b945",
    redis_git_dirty: "0",
    os: "Linux 2.6.32.16-linode28 i686",
    arch_bits: "32",
    multiplexing_api: "epoll",
    gcc_version: "4.4.1",
    process_id: "8107",
    ...
  },
  memory: {...},
  client: {...},
  ...
}
```

## Eval 和 Evalsha 命令

`eval` 和 `evalsha` 命令十分特殊，因为它们可以返回任意类型。Vert.x 基于 Java 语言，而 Java 是强类型语言，并且我们又要避免使用 `Object` 类型，这使得返回任意类型变得比较困难。避免使用 `Object` 类型的原因是因为 Vert.x 也是多语言的，而不同语言之间的类型转换实现起来十分的复杂和困难。所以，我们会用 `JsonArray` 来包装 `eval` 和 `evalsha` 命令的返回值，即便是像下面这样简单的脚本：

```javascript
return 10
```

执行上面的脚本，将返回一个 `JsonArray` 对象，且下标为 0 的值为 10。

---

> [原文档](http://vertx.io/docs/vertx-redis-client/java/)更新于2017-03-15 15:54:14 CET
