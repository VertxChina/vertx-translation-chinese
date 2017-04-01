# Vert.x JDBC Client

> 原文档：[Vert.x JDBC Client](http://vertx.io/docs/vertx-jdbc-client/java/) 

**使用 Vert.x JDBC Client，可以让我们的 Vert.x 应用程序通过异步的方式，与任何支持 JDBC 的数据库进行交互。**

Vert.x JDBC Client 的接口定义为 [`JDBCClient`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html)。

要使用Vert.x JDBC Client，需要添加下面的依赖：

- Maven (在 `pom.xml` 文件中):

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-jdbc-client</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-jdbc-client:3.4.1'
```

## 创建 Vert.x JDBC Client 对象

接下来，我们一起来看下创建 Vert.x JDBC Client 对象的几种方式。

### 默认使用共享的数据源

大部分情况下，我们希望在不同的 Vert.x JDBC Client 对象之间，共享一个数据源。

考虑这样一种情况：我们在部署 Verticle 时，设置了 Verticle 拥有多个实例化的对象，但是我们希望每个 Verticle 实例能够共享同一个数据源，而不是单独为每个 Verticle 实例设置不同的数据源。

要解决上面的问题，我们可以这么做：

```java
JDBCClient client = JDBCClient.createShared(vertx, config);
```

只有在第一次调用 [`JDBCClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法的时候，才会真正的根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的 Vert.x JDBC Client 对象，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

### 指定数据源名称

我们还可以像下面这样，在创建一个 Vert.x JDBC Client 对象的时候指定数据源的名称：

```java
JDBCClient client = JDBCClient.createShared(vertx, config, "MyDataSource");
```

如果不同的 Vert.x JDBC client 对象使用了相同的Vert.x 对象和相同的数据源名称，那么它们将共享数据源。

同样的（与默认使用共享的数据源），只有在第一次调用[`JDBCClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-)方法的时候，才会真正的根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的 Vert.x JDBC Client 对象，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

当我们希望不同含义的 Vert.x JDBC Client 对象拥有不同的数据源时，可以采用这种方式来创建它的对象。比如它们要与不同的数据源进行交互。

### 创建不共享数据源的 Vert.x JDBC Client 对象

在大部分情况下，我们会希望在不同的 Vert.x JDBC Client 对象之间共享数据源。但有时候，却恰恰相反。

这时，可以调用 [`JDBCClient.createNonShared`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html#createNonShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法：

```java
JDBCClient client = JDBCClient.createNonShared(vertx, config);
```

每次调用此方法，就相当于在调用 [`JDBCClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法时加上了具有唯一名称的数据源参数。

### 指定数据源

如果我们已经存在一个数据源，也可以在创建 Vert.x JDBC Client 对象的时候就直接指定它：

```java
JDBCClient client = JDBCClient.create(vertx, dataSource);
```

## 关闭客户端

我们可以较长时间的持有 Vert.x JDBC Client 对象（比如在 Verticle 的整个生命周期里），可一旦不再使用它后，就应该关闭它。

在多个 Vert.x JDBC Client 对象共享数据源的情况下，这个数据源对象维护着一个引用计数器。一旦此数据源最后一个引用关闭后，这个数据源也就关闭了。

### 在 Verticle 中自动关闭

我们在 Verticle 中创建的 Vert.x JDBC Client 对象，会在这些 Verticle 卸载（undeploy）的时候自动关闭。

## 获取数据库连接

在创建 Vert.x JDBC Client 对象后，我们可以通过 [`getConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html#getConnection-io.vertx.core.Handler-) 方法来获取一个数据库连接。

此方法从连接池中获取一个数据库连接，并返回给回调方法：

```java
client.getConnection(res -> {
  if (res.succeeded()) {

    SQLConnection connection = res.result();

    connection.query("SELECT * FROM some_table", res2 -> {
      if (res2.succeeded()) {

        ResultSet rs = res2.result();
        // 用结果集results进行其他操作
      }
    });
  } else {
    // 获取连接失败 - 处理失败的情况
  }
});
```

获取的这个连接，是接口 [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html) 的一个实现。但是 [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html) 是一个通用接口，不只是在 Vert.x JDBC Client 中有用到。

更多详细内容，可以参考文档 [Vert.x Common SQL Interface](http://vertx.io/docs/vertx-sql-common/java/) 。

## 配置参数

在 Vert.x JDBC Client 创建或者部署的时候，我们应该把对应的配置参数传给它。

常用的配置参数有下面这些：

- `provider_class`

  此参数确定管理数据库连接池的实现类，默认是`io.vertx.ext.jdbc.spi.impl.C3P0DataSourceProvider` 。如果我们想使用其他的类，可以自己设置这个参数，但参数对应的类，必须实现接口`DataSourceProvider`。

- `row_stream_fetch_size`

  为了提升性能，`SQLRowStream` 设置有内部缓存。默认缓存大小为 `128`。

若我们使用的是 Vert.x 默认的 C3P0（`DataSourceProvider`），可以使用下面的配置参数：

- `url`

  数据库 JDBC 连接地址

- `driver_class`

  JDBC 驱动

- `user`

  数据库用户名

- `password`

  数据库密码

- `max_pool_size`

  连接池最大连接数，默认`15`

- `initial_pool_size`

  连接池初始连接数，默认`3`

- `min_pool_size`

  连接池最小连接数

- `max_statements`

  预处理SQL语句最小缓存数，默认 `0`

- `max_statements_per_connection`

  每个数据库连接的最大预处理SQL缓存数，默认`0`

- `max_idle_time`

   空闲连接保留时间，默认`0` (代表一直保留)

其它连接池实现：

- BoneCP
- Hikari

类似于 C3P0，上面的连接池也可以通过传递一个 JsonObject 对象来配置参数。考虑这样一种情况，应用程序要运行在一个 Vert.x 环境中，但我们却不想通过 *fat jar* 的方式来部署，且在没有权限把 JDBC 驱动包加到 Vert.x lib 目录下的时候，建议使用 BoneCP，并在命令行上加上`-cp`标示。

如果想要配置 C3P0 更多的参数，我们可以在 classpath 下添加文件`c3p0.properties`。

例如：

```java
JsonObject config = new JsonObject()
  .put("url", "jdbc:hsqldb:mem:test?shutdown=true")
  .put("driver_class", "org.hsqldb.jdbcDriver")
  .put("max_pool_size", 30);

JDBCClient client = JDBCClient.createShared(vertx, config);
```

需要注意，Hikari 使用的配置参数不一样：

- `jdbcUrl` 数据库 JDBC 连接地址
- `driverClassName`  JDBC 驱动类名
- `maximumPoolSize` 连接池最大连接数
- `username` 数据库用户名 (`password` 数据库密码)

可以阅读 [Hikari documentation](https://github.com/brettwooldridge/HikariCP#configuration-knobs-baby) 和 [BoneCP documentation](http://www.jolbox.com/configuration.html) 来了解更多关于两个数据源的知识。

## JDBC 驱动

如果使用默认的`DataSourceProvider`实现类（C3P0实现），我们要把JDBC 驱动jar包放在编译路径下。

如果我们的 Vert.x 应用程序打包成 *fat jar*，要确保 JDBC 驱动包含在里面。如果我们的 Vert.x 应用程序通过 `vertx` 命令行启动，要确保 JDBC 驱动包在 `${VERTX_HOME}/lib` 路径里面。

使用不同的连接池时，可能会稍有不一样。

## 数据类型

由于 Vert.x 使用 JSON 作为标准的消息格式，这使得客户端在接受数据类型时受到很多限制。我们只能从 JsonObject 获得下面的数据类型：

- null
- boolean
- number
- string

时间类型 (TIME, DATE, TIMESTAMP) 可以自动转换。需要注意的是，我们可以选择性的使用 UUID 的转换。虽然大部分数据库都支持 UUID，可并不是所有都支持，比如说 MySQL 就不支持。这种情况下，建议使用 VARCHAR(36) 类型的字段。对于其他支持 UUID 的数据库来说，使用下面的参数后，可以对 UUID 进行自动类型转换。

```json
{ "castUUID": true }
```

这样，UUID 将会被作为原生类型（译者注：`java.util.UUID`）来处理。

## 作为 OSGI 应用程序

Vert.x JDBC Client 也可以作为 OSGI 应用程序。但是，必须首先部署它所有的依赖。但有些连接池要求必须从 classpath 加载 JDBC 驱动，这样的就不能作为 OSGI 应用程序。

---

[原文档](http://vertx.io/docs/vertx-jdbc-client/java/)更新于2017-03-15 15:54:14 CET
