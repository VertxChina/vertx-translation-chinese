# Vert.x Config

- [原文档][1]
- [组件源码][2]

## 中英文对照表

- Overloading：重载
- Configuration Store：配置仓库

## 组件介绍

Vert.x Config 提供了一种配置 Vert.x 应用的方式。

它包含：

- 支持多种配置语法（JSON、properties、YAML（扩展）、HOCON（扩展）等）
- 支持多种存储方式（文件、目录、HTTP、Git（扩展）、Redis（扩展）、系统属性、环境变量等）
- 支持自定义配置处理顺序和配置重载(overloading)
- 支持运行时重新配置(runtime reconfiguration)

## 概念

本组件基于如下几个概念：

- **Config Retriever** 由Vert.x 应用实例化以及使用；它配置了一系列的配置仓库
- **Configuration Store** 定义了配置项的数据源和语法（默认是 JSON）

配置文件最终会以 JSON 对象(`JsonObject`)的形式取出。

## 使用 Config Retriever

要使用 Config Retriever，只需要在依赖中增加以下代码片段：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
```

一旦完成以上步骤，我们第一步所需要做的就是实例化 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html)：

```java
ConfigRetriever retriever = ConfigRetriever.create(vertx);
```

默认配置下，Config Retriever 会按照以下的顺序进行配置：

- Vert.x Verticle 中的 `config()` 函数
- 系统配置
- 环境变量

你可以通过下列示例自定义配置：

```java
ConfigStoreOptions httpStore = new ConfigStoreOptions()
  .setType("http")
  .setConfig(new JsonObject()
    .put("host", "localhost").put("port", 8080).put("path", "/conf"));

ConfigStoreOptions fileStore = new ConfigStoreOptions()
  .setType("file")
  .setConfig(new JsonObject().put("path", "my-config.json"));

ConfigStoreOptions sysPropsStore = new ConfigStoreOptions().setType("sys");


ConfigRetrieverOptions options = new ConfigRetrieverOptions()
  .addStore(httpStore).addStore(fileStore).addStore(sysPropsStore);

ConfigRetriever retriever = ConfigRetriever.create(vertx, options);
```

在后续章节会将更加详细的重载规则和其他可用的配置仓库设置一一展现。

一旦成功实例化 Config Retriever, 可用如下代码获取配置：

```java
retriever.getConfig(ar -> {
  if (ar.failed()) {
    // Failed to retrieve the configuration
  } else {
    JsonObject config = ar.result();
  }
});
```

## 重载规则

明确配置仓库的顺序对于重载极为重要。对于冲突的键名，后面的配置值将替换掉之前的。举个例子，比如我们有两个配置仓库：

- `A` 含有 `{a:value, b:1}`
- `B` 含有 `{a:value2, c:2}`

如果我们配置的顺序是 (A, B)，那我们的配置结果将是：`{a:value2, b:1, c:2}`。  
如果我们将配置顺序颠倒为 (B, A)，我们最终得到的结果是：`{a:value, b:1, c:2}`。

## 支持的配置仓库

Vert.x Config 提供了一系列的配置存储支持和格式支持。更多的种类支持以扩展组件的形式提供，当然你也可以实现自己的扩展组件。

### 配置的数据结构

声明每一个数据存储源就必须要指明存储类型（`type`）。它可以指定格式（`format`），默认将使用 JSON。

一些配置仓库需要一些额外的配置项（比如路径）。我们可以通过 [`setConfig`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigStoreOptions.html#setConfig-io.vertx.core.json.JsonObject-) 方法传入一个 `JsonObject` 对象来进行上述的配置。

### 文件

此配置仓库仅仅从文件中读取配置，支持所有的格式。

```java
ConfigStoreOptions file = new ConfigStoreOptions()
  .setType("file")
  .setFormat("properties")
  .setConfig(new JsonObject().put("path", "path-to-file.properties"));
```

其中 `path` 参数是必须的。

### JSON

JSON 配置仓库仅支持通过给定的 JSON 对象（`JsonObject`）进行配置。

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("json")
  .setConfig(new JsonObject().put("key", "value"));
```

本配置仓库仅支持 JSON 格式。

### 环境变量

此配置仓库将环境变量中的键值对映射成 JSON 对象传入，作为全局配置。

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("env");
```

此配置仓库不支持 `format` 配置项。

### 系统属性

此配置仓库将系统属性中的键值对映射成 JSON 对象传入，作为全局配置。

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("sys")
  .setConfig(new JsonObject().put("cache", "false"));
```

此配置仓库不支持 `format` 配置项。

您可以对 `cache` 属性进行配置（默认为 `true`）。此配置项决定是否在初次获取系统属性时对其进行缓存，并且在后面不会重新加载。

### HTTP

此配置仓库会通过给定的 HTTP 地址获得配置，可以使用任何支持的格式。

```java
ConfigStoreOptions http = new ConfigStoreOptions()
  .setType("http")
  .setConfig(new JsonObject()
    .put("host", "localhost")
    .put("port", 8080)
    .put("path", "/A"));
```

它会创建一个Vert.x HTTP 客户端（`HttpClient`）来访问配置仓库（请看下面的代码）。你也可以通过配置 `host`、`port` 和 `path` 这些属性来简化这个过程。

```java
ConfigStoreOptions http = new ConfigStoreOptions()
  .setType("http")
  .setConfig(new JsonObject()
    .put("defaultHost", "localhost")
    .put("defaultPort", 8080)
    .put("ssl", true)
    .put("path", "/A"));
```

### Event Bus

此配置仓库会从 Event Bus 中获取配置。这种方式让你可以在本地和分布式组件中分发你的配置。

```java
ConfigStoreOptions eb = new ConfigStoreOptions()
  .setType("event-bus")
  .setConfig(new JsonObject()
    .put("address", "address-getting-the-conf")
  );
```

此配置仓库也支持所有的格式。

### 目录

这种配置仓库非常像 `file` 配置方式，但是不同的是，`file` 只读取一个配置文件，而这种方式会从一个目录中读取一批配置文件。

此方式有两个必要的参数：

- `path` - 文件所在的根结点路径
- 至少一个 `fileset` - 需要读取的文件列表（模式）

每一个 `fileset` 都包含：`pattern`（匹配模式），即用于选取文件的Ant风格的匹配模式。此匹配模式使用相对路径确定配置文件位置。`format`（格式）作为可选的参数（每一个 `fileset` 都可以使用不同的格式，但是在一个 `fileset` 内的文件必须是同一种格式）。

```java
ConfigStoreOptions dir = new ConfigStoreOptions()
  .setType("directory")
  .setConfig(new JsonObject().put("path", "config")
    .put("filesets", new JsonArray()
      .add(new JsonObject().put("pattern", "dir/*json"))
      .add(new JsonObject().put("pattern", "dir/*.properties")
        .put("format", "properties"))
    ));

```

## 监听配置变更

Configuration Retriever 会定期地从配置仓库处读取配置。如果读取的结果和当前的配置不一样，那么应用就会重新进行配置，默认情况下，配置的刷新时间是 5 秒钟。

```java
ConfigRetrieverOptions options = new ConfigRetrieverOptions()
  .setScanPeriod(2000)
  .addStore(store1)
  .addStore(store2);

ConfigRetriever retriever = ConfigRetriever.create(Vertx.vertx(), options);
retriever.getConfig(json -> {
  // Initial retrieval of the configuration
});

retriever.listen(change -> {
  // Previous configuration
  JsonObject previous = change.getPreviousConfiguration();
  // New configuration
  JsonObject conf = change.getNewConfiguration();
});
```

## 检索最近一次获取的配置内容

可以快速、无需“等待”地获得最近一次获取的配置内容：

```java
JsonObject last = retriever.getCachedConfig();
```

## 使用流的方式读取配置

[`ConfigRetriever `](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 提供流(Stream)的方式去获取配置。对应的流是 [`JsonObject`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html) 类型的可读流([`ReadStream`](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html))。你可以注册相应的处理器(`Handler`)，来获得下面的事件通知：

- 检索到新的配置
- 获取新配置时发生错误
- 当 `ConfigRetriever` 关闭时（[`endHandler`](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#endHandler-io.vertx.core.Handler-) 方法会被调用）

```java
ConfigRetrieverOptions options = new ConfigRetrieverOptions()
  .setScanPeriod(2000)
  .addStore(store1)
  .addStore(store2);

ConfigRetriever retriever = ConfigRetriever.create(Vertx.vertx(), options);
retriever.configStream()
  .endHandler(v -> {
    // 关闭时
  })
  .exceptionHandler(t -> {
    // 获取配置发生错误
  })
  .handler(conf -> {
    // 配置内容
  });

```

## 使用 Future 方式读取配置

[`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 也支持采用 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html) 作为返回值的方式获取配置：

```java
Future<JsonObject> future = ConfigRetriever.getConfigAsFuture(retriever);
future.setHandler(ar -> {
  if (ar.failed()) {
    // Failed to retrieve the configuration
  } else {
    JsonObject config = ar.result();
  }
});
```

## 扩展 Config Retriever

你可以通过以下SPI对其进行扩展：

- 实现 [`ConfigProcessor`](http://vertx.io/docs/apidocs/io/vertx/config/spi/ConfigProcessor.html) SPI，用来增加对新格式的支持
- 实现 [`ConfigStoreFactory`](http://vertx.io/docs/apidocs/io/vertx/config/spi/ConfigStoreFactory.html) SPI，用来增加对配置仓库（读取配置的位置）的支持

## 额外的配置格式

除了上文所提到的一系列现成的格式支持以外，Vert.x Config 也提供额外的格式支持组件，并可以在你的应用中使用。

### HOCON 配置格式

HOCON 配置格式组件是对 Vert.x Config 的扩展，并且提供 [HOCON](https://github.com/typesafehub/config/blob/master/HOCON.md)) 格式的支持。

此格式支持 includes, json, properties, macros等。

#### 使用 HOCON 配置格式

为了使用 HOCON 配置格式，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-hocon</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-hocon:3.4.1'
```

#### 配置使用 HOCON 格式

一旦添加完依赖，我们就需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此格式：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
  .setType("file")
  .setFormat("hocon")
  .setConfig(new JsonObject()
    .put("path", "my-config.conf")
  );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

你仅仅需要将 `format` 设置为 `hocon`。


### YAML 配置格式

YAML 配置格式组件是对 Vert.x Config 的扩展，并且提供 YAML 格式的支持。

#### 使用 YAML 配置格式

为了使用 YAML 配置格式，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-yaml</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-yaml:3.4.1'
```

#### 配置使用 YAML 格式

一旦添加完依赖，我们就需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此格式：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
  .setType("file")
  .setFormat("yaml")
  .setConfig(new JsonObject()
    .put("path", "my-config.yaml")
  );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

你仅仅需要将 `format` 设置为 `yaml`。

## 额外的配置仓库

除了上文所提到的一系列现成的配置仓库支持以外，Vert.x Config 也提供额外的配置仓库支持，并可以在你的应用中使用。

### Git 配置仓库

Git 配置仓库组件是对 Vert.x Config 的扩展，支持从 Git 仓库(repository)中获取配置。

#### 使用 Git 配置仓库

为了使用 Git 配置仓库，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-git</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-git:3.4.1'
```

#### 对配置仓库进行配置

一旦添加完依赖，我们就需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置仓库：

```java
ConfigStoreOptions git = new ConfigStoreOptions()
    .setType("git")
    .setConfig(new JsonObject()
        .put("url", "https://github.com/cescoffier/vertx-config-test.git")
        .put("path", "local")
        .put("filesets",
            new JsonArray().add(new JsonObject().put("pattern", "*.json"))));

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(git));
```

此配置仓库需要以下几个参数：  

- 仓库(Git repository)的 `url`（地址）
- 仓库克隆的 `path`（本地路径）
- 至少一个 `fileset` —— 需要读取的文件列表（模式）（等同于目录配置仓库的配置方式）

你也可以配置 `branch`(分支，默认为 `master`)，并且可以指定 `remote`（远程）仓库名称（默认读取 `origin`）

#### 它是如何工作的

如果本地 `path` 不存在，那首先将配置的 Git 仓库克隆下来，然后从中读取匹配的文件集合。

如果本地 `path` 存在，那首先尝试更新（如果需要切换分支就切换分支）。如果更新失败，那配置读取也会失败。

Vert.x Config会周期性的去检查变更并更新本地仓库。

### Kubernetes ConfigMap 配置仓库

Kubernetes ConfigMap 配置仓库组件是对 Vert.x Config 的扩展。它提供了对 [Kubernetes ConfigMap](https://kubernetes.io/docs/user-guide/configmap/) 和 [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) 的支持。

#### 使用 Kubernetes ConfigMap 配置仓库

为了使用 Kubernetes ConfigMap 配置仓库，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-kubernetes-configmap</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-kubernetes-configmap:3.4.1'
```

#### 对配置仓库进行配置

一旦添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置仓库：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("configmap")
    .setConfig(new JsonObject()
        .put("namespace", "my-project-namespace")
        .put("name", "configmap-name")
    );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

你需要对配置仓库进行配置，来找到正确的 ConfigMap：

- `namespace` —— 项目的命名空间，默认是 `default`。如果环境变量中有 `KUBERNETES_NAMESPACE`，则会优先使用此值
- `name` —— ConfigMap 的名称。

如果 ConfigMap 由多个元素部分构成，你可以使用 `key` 参数确定哪一个才是我们要读取的。另外，应用必须要拥有对 ConfigMap 的读权限。

为了从 Secret 中读取数据，必须设置 `secret` 属性为 `true`：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("configmap")
    .setConfig(new JsonObject()
        .put("namespace", "my-project-namespace")
        .put("name", "my-secret")
        .put("secret", true)
    );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```


### Redis 配置仓库

Redis 配置仓库组件是对 Vert.x Config 的扩展，支持从 Redis 服务器中获取配置。

#### 使用 Redis 配置仓库

为了使用 Redis 配置仓库，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-redis</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-redis:3.4.1'
```

#### 对配置仓库进行配置

一旦添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置仓库：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("redis")
    .setConfig(new JsonObject()
        .put("host", "localhost")
        .put("port", 6379)
        .put("key", "my-configuration")
    );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

此配置仓库会创建一个 [`RedisClient`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html) 实例。关于此部分的配置可以参考 [Vert.x Redis Client 文档](http://vertx.io/docs/vertx-redis-client/java/)。

除此之外，你可以设置 `key` 参数用来指示存储配置对应的 *field（字段）*，默认值是 `configuration`。

最终创建的 [`RedisClient`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html) 实例将通过 `HGETALL` 命令来检索配置。

### ZooKeeper 配置仓库

ZooKeeper 配置仓库组件是对 Vert.x Config 的扩展，支持从 ZooKeeper 服务器中获得配置。

此组件底层使用 [Apache Curator](http://curator.apache.org/) 作为访问 ZooKeeper 的客户端。

#### 使用 ZooKeeper 配置仓库

为了使用 ZooKeeper 配置仓库，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-zookeeper</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-zookeeper:3.4.1'
```

#### 对配置仓库进行配置

一旦添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置仓库：

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("zookeeper")
    .setConfig(new JsonObject()
        .put("connection", "localhost:2181")
        .put("path", "/path/to/my/conf")
    );

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

使用此配置仓库通常需要配置 [Apache Curator](http://curator.apache.org/) 客户端以及存储我们配置内容的 ZNode 路径（`path`）。此节点的数据格式可以是 JSON 或者其它任意支持的类型。

此配置仓库需要传入一系列参数：`configuration` 属性包含连接 ZooKeeper 服务器的地址和包含我们所配置的节点 `path` 路径。

额外可以配置的有：

- `maxRetries`：尝试重新连接的次数，默认是 3
- `baseSleepTimeBetweenRetries`： 在每一次重新尝试连接的间隔（[Exponential Backoff](https://en.wikipedia.org/wiki/Exponential_backoff) 策略），默认是 1000 ms

### Spring Config Server 配置仓库

Spring Config Server 配置仓库组件是对 Vert.x Config 的扩展，支持从 Spring Config Server 中获得配置。

#### 使用 Spring Config Server 配置仓库

为了使用 Spring Config Server 配置仓库，您需要添加以下依赖：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config-spring-config-server</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-config</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-config:3.4.1'
compile 'io.vertx:vertx-config-spring-config-server:3.4.1'
```

#### 对配置仓库进行配置

一旦添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置仓库。

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("spring-config-server")
    .setConfig(new JsonObject().put("url", "http://localhost:8888/foo/development"));

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

以下的参数可供配置：

- `url`: 读取配置的 `url`（必填）
- `timeout`: 读取配置的超时时间，单位是毫秒，默认为 3000 ms
- `user`: 用户（默认无权限）
- `password`: 密码
- `httpClientConfiguration`: 底层所使用的 HTTP 客户端的配置

---

> [原文档](http://vertx.io/docs/vertx-config/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-config/java/
[2]: https://github.com/vert-x3/vertx-config
