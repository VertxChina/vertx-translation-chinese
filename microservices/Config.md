# Vert.x Config

## 中英文对照表


- Overloading：重载
- Configuration Store：配置源

## 正文

Vert.x Config 为你的 Vert.x 应用提供配置服务。
包含：

- 支持多种配置语法(json, properties, yaml (extension), hocon (extension)…
- 支持多种存储方式(files, directories, http, git (extension), redis (extension), system properties, environment properties)…​
- 支持自定义选择顺序和重载行为
- 支持运行时重新配置

## 概念
本模块围绕着一下的几个概念：

- **Config Retriever** 是一系列配置文件源(Configuration store)的配置，由 Vert.x 实例化并管理。
- **Configuration store** 的定义是配置文件的数据源和它的语法(默认 json)。

配置文件最终会以 JSON Object 格式取出。

---

## 使用 Config Retriever
使用 Config Retriever，只需要在依赖中增加以下代码片段：

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

一旦完成以上步骤，我们第一步所需要做的就是实例化 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html):

```java
ConfigRetriever retriever = ConfigRetriever.create(vertx);
```

默认配置下，Config Retriever 会按照以下的顺序配置：

- Vert.x verticle 的 `config()` 函数
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

在后续章节会将更加详细的重载规则和其他可用的配置源设置一一展现。

一但成功实例化 Config Retriever, 可用如下代码使用配置：

```java
retriever.getConfig(ar -> {
  if (ar.failed()) {
    // Failed to retrieve the configuration
  } else {
    JsonObject config = ar.result();
  }
});
```
---

## 重载规则
明确配置源的顺序对于重载极为重要，对于冲突的键名，将以最终获得的配置为准，换言之将会替换掉之前获得的变量值。举个例子，比如我们有两个配置源：

- `A` 含有 `{a:value, b:1}`
- `B` 含有 `{a:value2, c:2}`

如果我们配置的顺序是 (A, B)，那我们的配置结果将是： `{a:value2, b:1, c:2}`。  
如果我们将配置顺序颠倒下 (B, A)，我们最终得到的结果是： `{a:value, b:1, c:2}`。

---


## 支持的配置源
Config Retriever 提供了一系列的配置存储支持和格式支持，当然你可以实现自己的拓展。

### 配置的数据结构
申明每一个数据源就必须要指明 `类型` ,它可以定为 `格式`, 默认将使用 JSON。  
一些配置源需要一些额外的配置项（比如路径……），这些配置可以通过 [`setConfig`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigStoreOptions.html#setConfig-io.vertx.core.json.JsonObject-) 函数传入 Json Object 对象实现。

### File
此配置仅仅从文件中读取配置，支持所有的格式。

```java
ConfigStoreOptions file = new ConfigStoreOptions()
  .setType("file")
  .setFormat("properties")
  .setConfig(new JsonObject().put("path", "path-to-file.properties"));
```
`path` 参数是必须的。

### JSON
JSON配置仅仅支持JSON配置。 

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("json")
  .setConfig(new JsonObject().put("key", "value"));
```
本配置仅支持JSON格式。

### Environment Variables
此配置将环境变量中的键值对作为JSON对象传入。

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("env");
```
此配置不支持 `format` 配置，
### System Properties
此配置系统属性的键值对作为JSON对象全局参数。

```java
ConfigStoreOptions json = new ConfigStoreOptions()
  .setType("sys")
  .setConfig(new JsonObject().put("cache", "false"));
```
此配置不支持 `format` 配置，

### HTTP
此方式从HTTP地址中获得配置，可以支持任何格式。

```java
ConfigStoreOptions http = new ConfigStoreOptions()
  .setType("http")
  .setConfig(new JsonObject()
    .put("host", "localhost")
    .put("port", 8080)
    .put("path", "/A"));
```

It creates a Vert.x HTTP Client with the store configuration (see next snippet). To ease the configuration, you can also configure the host, port and path with the host, port and path properties.

创建一个 Vert.x HTTP 用使用独立配置的客户端（请看下面代码）。此配置方式可以自行设定参数包括：`host`, `port` 和 `path `。 

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
此方式从 Event Bus 中获得配置，此方式可以方便在分布式系统中发布配置。

```java
ConfigStoreOptions eb = new ConfigStoreOptions()
  .setType("event-bus")
  .setConfig(new JsonObject()
    .put("address", "address-getting-the-conf")
  );
```
此方式也支持所有的格式。

### Directory
此种方式非常像 `file` 配置方式，但是不是从单一的文件中读取配置，而是从目录中读取。

此方式有两个必要的参数：

- `path` - 文件所在的根结点路径
- 至少一个`fileset` - 需要读取的文件列表

Each fileset contains: * a pattern : a Ant style pattern to select files. The pattern is applied on the relative path of the files location in the directory. * an optional format indicating the format of the files (each fileset can use a different format, BUT files in a fileset must share the same format).

每一个 `fileset` 都包含：`pattern（匹配模式）`：Ant风格的匹配用来制定文件。此匹配模式使用相对路径确定配置文件位置。`format（格式）`格式作为可选的参数（每一个fileset都可以使用不同的格式，但是在一个fileset内的文件必须是同一种格式）。


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

---

## 监听配置改变
Configuration Retriever 会定期的从配置源处读取配置，如果读取的结果和当前的不一样，那么应用就会重新配置，默认配置下，配置的刷新时间是 5 秒钟。

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

---

## 检索最近一次配置内容
可以快速无需“等待”的获得最近一次的配置内容：

```java
JsonObject last = retriever.getCachedConfig();
```
---

## 使用 stream 方式读取配置
[`ConfigRetriever `](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html)提供stream的方式去获得数据，是[`JsonObject`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)的[`ReadStream`](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)。使用正确的注册方式，就可以在恰当的时间获得通知。

- 检索到新的配置
- 获取新配置时发生错误
- 当 configuration retriever 关闭时（[`endHandler`](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#endHandler-io.vertx.core.Handler-)会被调用）

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
---

## 拓展 Config Retriever

你可以通过以下SPI对其进行拓展：

- 实现[`ConfigProcessor`](http://vertx.io/docs/apidocs/io/vertx/config/spi/ConfigProcessor.html)SPI用来增加对新格式的支持。
- 实现[`ConfigStoreFactory`](http://vertx.io/docs/apidocs/io/vertx/config/spi/ConfigStoreFactory.html)SPI用来增加对配置源的支持（配置哪里去读取配置）。

---

## 额外的格式
除了上文所提到的一系列格式的支持以外，Vert.x 也提供额外的格式支持，并可以在你的应用中使用。

### Hocon 配置格式

Hocon 是对 Vert.x Configuration Retriever 的拓展，并且支持 HOCON([`https://github.com/typesafehub/config/blob/master/HOCON.md`](https://github.com/typesafehub/config/blob/master/HOCON.md)) 格式。

此格式支持 includes, json, properties, macros…​

#### 使用 Hocon 配置格式
为了使用 Hocon 配置格式，你需要首先增加你的依赖：

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

#### 配置使用 Hocon 格式
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此格式。

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


### Yaml 配置格式

Yaml 是对 Vert.x Configuration Retriever 的拓展，并且支持 Yaml 格式。

#### 使用 Yaml 配置格式
为了使用 Yaml 配置格式，你需要首先增加你的依赖：

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
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此格式。

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

---

## 额外的配置源
除了上文所提到的一系列配置源的支持以外，Vert.x 也提供额外的配置源支持，并可以在你的应用中使用。

### Git 配置源
Git 配置源是对  Vert.x Configuration Retriever 获得配置的拓展，支持从 Git 仓库中获得配置。

#### 使用 Git 配置源
为了使用 Git 配置源，你需要首先增加你的依赖：

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

#### 配置源
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置源。

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
此配置源需要以下几个参数：  

- 仓库的`url(地址)`
- 仓库克隆的 `path` （本地路径）
- 至少一个`fileset` - 需要读取的文件列表（等同于 directory 的配置方式）

你也可以配置 `branch（分支）`(默认的是 `master`)，并且可以指定 `remote（远程）`仓库地址（默认读取 `origin`）

#### 它是如何工作的
如果本地`路径`不存在，那首先将配置的 Git 仓库克隆下来，然后从中读取匹配的文件集。   
如果本地`路径`存在，那首先尝试更新（如果需要切换分支就切换分支），如果更新失败，那读取也会失败。  
并且会周期性的去检测更新本地仓库。


### Kubernetes ConfigMap 配置源
Kubernetes ConfigMap 配置源是对  Vert.x Configuration Retriever 获得配置的拓展，支持从 Kubernetes Config Map 和 Secrets。

#### 使用 Kubernetes ConfigMap 配置源

为了使用 Kubernetes ConfigMap 配置源，你需要首先增加你的依赖：

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

#### 配置源
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置源。
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

你需要的是正确的配置用来找到正确的configmap：

- `namespace` 项目的命名空间，默认是 `default`。如果环境变量中有 `KUBERNETES_NAMESPACE` 那会优先使用此值。
- `name` config map 的名称。

如果  config map 由多个元素部分构成，你可以使用 `key` 确定哪一个才是我们要读取的。  
另外，应用必须要拥有对config map的权限。  
为了从 secret 读取数据，必须设置 `secret`属性为 `true`:

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


### Redis 配置源
Redis 配置源是对  Vert.x Configuration Retriever 获得配置的拓展，支持从 Redis 服务器中获得配置。

#### 使用 Redis 配置源
为了使用 Redis 配置源，你需要首先增加你的依赖：

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

#### 配置源
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置源。

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

使用此配置源需要实例化 [`RedisClient`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html)。关于此部分可以查考 Vert.x Redis Client [文档](http://vertx.io/docs/vertx-redis-client/java/)

除此之外，你可以设置 `key` 用来定位所需要 *field（字段）*。默认值是 `configuration`。
最终创建的 RedisClient 对象检索数据将使用 `HGETALL` 配置。




### Zookeeper 配置源
Zookeeper 配置源是对  Vert.x Configuration Retriever 获得配置的拓展，支持从 Zookeeper 服务器中获得配置。

#### 使用 Zookeeper 配置源
为了使用 Zookeeper 配置源，你需要首先增加你的依赖：

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

#### 配置源
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置源。

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

使用此配置源通常需要配置 [Apache Curator](http://curator.apache.org/) 客户端，和包含我们需数据的 *Node Path*，此节点的数据格式可是是JSON或者其他受支持的类型。

此配置源需要传入一下一系列参数：`configuration` 属性包含 连接Zookeeper服务器的地址和包含我们所配置的节点 `path` 路径。

额外可以配置的有：

- `maxRetries`：尝试重新连接的次数，默认是3。
- `baseSleepTimeBetweenRetries`： 在每一次重新尝试连接的间隔（[exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) 策略），默认是 1000 ms。


### Spring Config Server 配置源
Spring Config Server 配置源是对  Vert.x Configuration Retriever 获得配置的拓展，支持从 Spring Server 中获得配置。

#### 使用 Spring Config Server 配置源
为了使用 Spring Config Server 配置源，你需要首先增加你的依赖：

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

#### 配置源
一但添加完依赖，我们需要配置 [`ConfigRetriever`](http://vertx.io/docs/apidocs/io/vertx/config/ConfigRetriever.html) 来使用此配置源。

```java
ConfigStoreOptions store = new ConfigStoreOptions()
    .setType("spring-config-server")
    .setConfig(new JsonObject().put("url", "http://localhost:8888/foo/development"));

ConfigRetriever retriever = ConfigRetriever.create(vertx,
    new ConfigRetrieverOptions().addStore(store));
```

以下的参数可供配置：

- `url`: 读取配置的 `url` （必填）
- `timeout`: 读取配置的超时时间，单位是毫秒
- `user`: 用户 （默认是无鉴权）
- `password`: 密码
- `httpClientConfiguration`: 底层所使用的 HTTP 客户端

> 最后更新时间 2017-03-15 15:54:14 CET


