# Vert.x MySQL/PostgreSQL Client

> 原文档：[Vert.x MySQL/PostgreSQL client](http://vertx.io/docs/vertx-mysql-postgresql-client/java/)

**此异步 MySQL/PostgreSQL 客户端给需要与 MySQL 或 PostgreSQL 数据库交互的 Vert.x 应用提供了相应的接口。**

Vert.x MySQL/PostgreSQL Client（以下简称客户端）底层实现基于 Mauricio Linhares 写的异步驱动 [postgresql-async](https://github.com/mauricio/postgresql-async)。此组件让 Vert.x 应用能够以异步、非阻塞的方式访问 MySQL 或者 PostgreSQL 数据库。

## 使用 Vert.x MySQL/PostgreSQL 客户端

这部分内容阐述了如何在您的应用中配置 Vert.x MySQL/PostgreSQL 客户端。

### 在常规应用中

要使用 Vert.x MySQL / PostgreSQL 客户端，您需要把下面的 jar 包加入 `CLASSPATH`:

- vertx-mysql-postgresql-client 3.4.1 (此客户端)
- scala-library 2.11.4
- postgress-async-2.11 和 mysql-async-2.11，来自 [https://github.com/mauricio/postgresql-async](https://github.com/mauricio/postgresql-async)
- joda time

所有这些都可以从 Maven 中心库下载。

### 在打包成 fat jar 的应用中

如果您使用 Maven 或者 Gradle 构建 *Fat-jar* 应用，只需要加入下面的依赖：

- Maven (在 `pom.xml` 文件中):

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-mysql-postgresql-client</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-mysql-postgresql-client:3.4.1'
```

### 在使用 Vert.x 环境的应用中

如果您使用 Vert.x 环境，需要把上面列出的 jar 包加入到  `$VERTX_HOME/lib`  路径下。

或者，您可以编辑位于 `$VERTX_HOME` 路径下的`vertx-stack.json` 文件，并且设置 `vertx-mysql-postgresql-client` 的依赖为 `"included": true`。完成后，执行命令：`vertx resolve --dir=lib --stack= ./vertx-stack.json`，就会下载此客户端及它的依赖。

## 创建客户端对象

有几种方式来创建客户端，我们一起来看下。

### 使用默认共享连接池

大部分情况下，您都将希望不同的客户端实例（`AsyncSQLClient`）共享一个连接池。

考虑这样一种情况：您在部署 Verticle 时，设置了 Verticle 拥有多个实例化的对象，但是您希望每个 Verticle 实例能够共享同一个数据源，而不是单独为每个 Verticle 实例设置不同的数据源。

要解决上面的问题，您可以这么做：

```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig);

// 创建一个 PostgreSQL 客户端

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig);
```

只有在第一次调用  [`MySQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/MySQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 或者 [`PostgreSQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/PostgreSQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法的时候，才会真正地根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的客户端实例，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

### 指定数据源名称

您还可以像下面这样，在创建一个客户端实例的时候指定数据源的名称：

```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig, "MySQLPool1");

// To create a PostgreSQL client:

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig, "PostgreSQLPool1");
```

如果不同的客户端对象使用了相同的 Vert.x 对象和相同的数据源名称，那么它们将共享数据源。

只有在第一次调用  [`MySQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/MySQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 或者 [`PostgreSQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/PostgreSQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法的时候，才会真正的根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的客户端对象，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

当我们希望不同含义的客户端对象拥有不同的数据源时，可以采用这种方式来创建它的对象，比如它们要与不同的数据库进行交互。

### 创建不共享数据源的客户端对象

在大部分情况下，我们会希望在不同的客户端实例之间共享数据源。但有时候，却恰恰相反。

这时，可以调用[`MySQLClient.createNonShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/MySQLClient.html#createNonShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 或者[`PostgreSQLClient.createNonShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/PostgreSQLClient.html#createNonShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法：

```java
JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
AsyncSQLClient mySQLClient = MySQLClient.createNonShared(vertx, mySQLClientConfig);

// 创建 PostgreSQL 客户端

JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
AsyncSQLClient postgreSQLClient = PostgreSQLClient.createNonShared(vertx, postgreSQLClientConfig);
```

每次调用此方法，就相当于在调用  [`MySQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/MySQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 或者 [`PostgreSQLClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/PostgreSQLClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法时加上了具有唯一名称的数据源参数。

## 关闭客户端

您可以较长时间的持有客户端对象（比如在 Verticle 的整个生命周期里），可一旦不再使用，就应该使用 [`close(handler)`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/AsyncSQLClient.html#close-io.vertx.core.Handler-) 或者 [`close()`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/AsyncSQLClient.html#close--) 方法来关闭它。

## 获取数据库连接

您可以使用 [`getConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/asyncsql/AsyncSQLClient.html#getConnection-io.vertx.core.Handler-) 方法来获得数据库连接。

此方法从连接池中获取一个数据库连接，并返回给回调方法：

```java
client.getConnection(res -> {
  if (res.succeeded()) {

    SQLConnection connection = res.result();

    // 获得一个连接

  } else {
    // 获取连接失败 - 处理异常
  }
});
```

**一旦您用完一个数据库连接后，请确保关闭它。**

获取的连接，是接口 [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html) 的一个实现。但是 [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html) 是一个通用接口，不只是在此客户端中有用到。

您可以在 [Vert.x Common SQL Interface 文档](http://vertx.io/docs/vertx-sql-common/java/) 中了解如何去使用它。

### 日期和时间戳

只要您从数据库从获取时间格式的数据，此客户端都将会隐式将它们转换成 ISO 8601（`yyyy-MM-ddTHH:mm:ss.SSS`）格式的字符串。MySQL 会舍弃毫秒项，所以您会看到 `.000`。

### 最后插入的数据id

在表中插入新数据时，您也许希望获得数据库的自增长id。JDBC API 通常都会让您从数据库连接中得到最后一个插入的 id。在 MySQL 中，可以按照 JDBC API 描述的方式获得最后插入的 id，而在 PostgreSQL 中，您可以使用 [RETURNING clause](http://www.postgresql.org/docs/current/static/sql-insert.html)。您可以选择使用其中的一个 `query` 方法来获取返回的列。

### 存储过程

 `call` 和 `callWithParams` 方法目前暂未实现。

## 配置参数

PostgreSql 和 MySql 客户端的配置参数一样：

```text
{
  "host" : <主机地址>,
  "port" : <端口>,
  "maxPoolSize" : <最大连接数>,
  "username" : <用户名>,
  "password" : <密码>,
  "database" : <数据库名称>,
  "charset" : <编码>,
  "queryTimeout" : <查询超时时间-毫秒>
}
```

- `host`

  数据库主机地址，默认为 `localhost`。

- `port`

  数据库端口，PostgreSQL 默认为 `5432`，MySQL 默认为 `3306` 。

- `maxPoolSize`

  最大连接数。默认为  `10`。

- `username`

  数据库用户名，PostgreSQL 默认为 `postgres` ，MySQL 默认为 `root` 。

- `password`

  数据库密码，默认不设置。

- `database`

  数据库名称，默认为 `testdb`。

- `charset`

  编码格式，默认为 `UTF-8`。

- `queryTimeout`

  查询超时时间（毫秒），默认为 `10000` (= 10秒)。

---

> [原文档](http://vertx.io/docs/vertx-mysql-postgresql-client/java/)更新于2017-03-15 15:54:14 CET
