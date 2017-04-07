# Vert.x Common SQL Interface

>原文档：[Vert.x Common SQL interface](http://vertx.io/docs/vertx-sql-common/java/)

Vert.x Common SQL Interface组件定义了 Vert.x 与各种 SQL 服务交互的方法。

您必须通过使用特定的 SQL 服务（例如 JDBC/MySQL/PostgreSQL）的接口来获取数据库连接。

要使用此组件，需要添加下列依赖：

- Maven (在 `pom.xml`文件中):

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-sql-common</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-sql-common:3.4.1'
```

## SQL 连接

我们用 [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html) 接口来表示数据库连接（译者注：此接口中包含各种基本的操作方法）。

### 自动提交

当您获取的数据库连接，其自动提交选项（auto commit）默认设置为 `true`。这意味着您的每个操作都将在单独的事务中有效执行。

如果您希望在同一个事务中执行多个操作，就应该使用 [` setAutoCommit`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#setAutoCommit-boolean-io.vertx.core.Handler-) 方法设置自动提交为`false`。

当操作完成时，回调方法将会被执行：

```java
connection.setAutoCommit(false, res -> {
  if (res.succeeded()) {
    // 成功！
  } else {
    // 失败!
  }
});
```

### 执行查询

您可以使用 [`query`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#query-java.lang.String-io.vertx.core.Handler-) 方法执行查询操作。

查询语句（原生SQL）传给数据库时，不会经过任何修改。

当查询结束时，将执行回调方法处理结果。查询结果包装在 [`ResultSet`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html) 中。

```java
connection.query("SELECT ID, FNAME, LNAME, SHOE_SIZE from PEOPLE", res -> {
  if (res.succeeded()) {
    // 获得 result set
    ResultSet resultSet = res.result();
  } else {
    // 失败!
  }
});
```

[`ResultSet`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html) 类代表查询结果。

您可以通过 [`getColumnNames`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html#getColumnNames--) 方法获得查询结果的列名 List 集合，实际的结果集可以通过 [`getResults`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html#getResults--) 方法获得。结果集被包装成了一组  [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 列表，其中的每个元素代表一行结果。

```java
List<String> columnNames = resultSet.getColumnNames();

List<JsonArray> results = resultSet.getResults();

for (JsonArray row : results) {

  String id = row.getString(0);
  String fName = row.getString(1);
  String lName = row.getString(2);
  int shoeSize = row.getInteger(3);

}
```

您还可以使用  [`getRows`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html#getRows--) 方法来获得被包装成了 JSON 对象列表（`List<JsonObject> `）的结果集，这样能让 API 的操作更简单些。但要注意的是，查询出的结果集中可能会出现重复的列名。若遇到这样的情况，您应该选择使用 [`getResults`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/ResultSet.html#getResults--) 方法。

下面是将结果集作为 `JsonObject` 进行迭代的例子：

```java
List<JsonObject> rows = resultSet.getRows();

for (JsonObject row : rows) {

  String id = row.getString("ID");
  String fName = row.getString("FNAME");
  String lName = row.getString("LNAME");
  int shoeSize = row.getInteger("SHOE_SIZE");

}
```

### 预编译查询

您可以使用 [`queryWithParams`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#queryWithParams-java.lang.String-io.vertx.core.json.JsonArray-io.vertx.core.Handler-) 方法执行预编译查询（prepared statement queries）。此方法接受含参数占位符的SQL查询语句以及 [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 对象（用于传递参数）或参数值。

```java
String query = "SELECT ID, FNAME, LNAME, SHOE_SIZE from PEOPLE WHERE LNAME=? AND SHOE_SIZE > ?";
JsonArray params = new JsonArray().add("Fox").add(9);

connection.queryWithParams(query, params, res -> {

  if (res.succeeded()) {
    // 获得 result set
    ResultSet resultSet = res.result();
  } else {
    // 失败!
  }
});
```

### 执行 INSERT/UPDATE/DELETE 语句

您可以使用 [`update`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#update-java.lang.String-io.vertx.core.Handler-) 方法来执行更新数据库的操作（包括增、删、改）。

更新语句（原生SQL）传给数据库时，不会经过任何处理。

当更新结束时，将执行回调方法处理结果。更新结果包装在  [`UpdateResult`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/UpdateResult.html) 对象中。

您可以通过 [`getUpdated`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/UpdateResult.html#getUpdated--) 方法获得更新的数据条数，并且如果更新操作有生成主键，可以通过 [`getKeys`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/UpdateResult.html#getKeys--)方法获得对应的主键。

```java
connection.update("INSERT INTO PEOPLE VALUES (null, 'john', 'smith', 9)", res -> {
  if (res.succeeded()) {

    UpdateResult result = res.result();
    System.out.println("Updated no. of rows: " + result.getUpdated());
    System.out.println("Generated keys: " + result.getKeys());

  } else {
    // Failed!
  }
});
```

### 预编译更新

您可以使用 [`updateWithParams`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#updateWithParams-java.lang.String-io.vertx.core.json.JsonArray-io.vertx.core.Handler-) 方法来执行预编译更新（prepared statement updates）。此方法接受含参数占位符的SQL更新语句以及 [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 对象（用于传递参数）或参数值。

```java
String update = "UPDATE PEOPLE SET SHOE_SIZE = 10 WHERE LNAME=?";
JsonArray params = new JsonArray().add("Fox");

connection.updateWithParams(update, params, res -> {

  if (res.succeeded()) {

    UpdateResult updateResult = res.result();

    System.out.println("No. of rows updated: " + updateResult.getUpdated());

  } else {

    // Failed!

  }
});
```

### 可调用语句

您可以使用 [`callWithParams `](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#callWithParams-java.lang.String-io.vertx.core.json.JsonArray-io.vertx.core.json.JsonArray-io.vertx.core.Handler-) 方法来执行可调用语句（callable statements），例如 SQL 函数或者存储过程。此方法接受以下参数：

- 可调用语句。可以使用标准 JDBC 格式 `{ call func_proc_name() }`，也可以选择使用占位符传参数的形式，例如：`{ call func_proc_name(?, ?) }`
- 输入参数集（`params`），[`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 类型
- 包含输出类型的输出结果集（`output`），[`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 类型，例如：`[null, 'VARCHAR']`
- 对应的回调函数（`resultHandler`）

请注意，输出结果集的 [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 的下标和输入参数的  [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 同样重要。如果第二个参数代表输出结果集，那么应该设置结果集的 [`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 的第一个元素为 null。

有些 SQL 函数只使用 `return` 关键字返回输出结果集，这时可以这样调用：

```java
String func = "{ call one_hour_ago() }";

connection.call(func, res -> {

  if (res.succeeded()) {
    ResultSet result = res.result();
  } else {
    // Failed!
  }
});
```

但是当您使用存储过程时，还是需要使用它的参数来返回结果集。如果一个存储过程没有返回值的话，可以像下面这样调用：

```java
String func = "{ call new_customer(?, ?) }";

connection.callWithParams(func, new JsonArray().add("John").add("Doe"), null, res -> {

  if (res.succeeded()) {
    // 成功!
  } else {
    // 失败!
  }
});
```

但是如果存储过程有返回值的话，需要像下面这样调用：

```java
String func = "{ call customer_lastname(?, ?) }";

connection.callWithParams(func, new JsonArray().add("John"), new JsonArray().addNull().add("VARCHAR"), res -> {

  if (res.succeeded()) {
    ResultSet result = res.result();
  } else {
    // 失败!
  }
});
```

请注意：输入输出参数的下标必须匹配 `?` 的下标，并且输出结果集元素的值必须是结果集类型的字符串表示。

为避免歧义，实现类需要遵循以下规则（译者注：可参考 Vert.x JDBC Client 的实现源码 [`JDBCStatementHelper.fillStatement(statement, in, out)`](https://github.com/vert-x3/vertx-jdbc-client/blob/master/src/main/java/io/vertx/ext/jdbc/impl/actions/JDBCStatementHelper.java#L97)）：

- 当 `IN` 参数的元素是 `NOT NULL` 时，此元素将被注册为输入参数
- 当 `IN` 参数的元素是 null 时，将进一步去检查 `OUT` 参数的元素值，再做判断
- 若当 `IN` 参数的元素是 null，且 `OUT` 参数的元素值不是 null 时，`OUT` 参数的元素值将被注册为输出参数
- 若当 `IN` 参数的元素是 null，且 `OUT` 参数的元素值也是 null 时， `IN` 参数的元素将被当作 `NULL` 值传入存储过程

注册为 `OUT` 的参数，设置成了 `ResultSet` 的 `output` 属性。

### 批量操作

Vert.x SQL 公共接口定义了3种批量操作的方法：

- 批量操作 [`batch`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#batch-java.util.List-io.vertx.core.Handler-)
- 批量预编译操作  [`batchWithParams`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#batchWithParams-java.lang.String-java.util.List-io.vertx.core.Handler-)
- 批量调用语句  [`batchCallableWithParams`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#batchCallableWithParams-java.lang.String-java.util.List-java.util.List-io.vertx.core.Handler-)

批量操作能执行一组 SQL 语句（`List` 类型），例如：

```java
List<String> batch = new ArrayList<>();
batch.add("INSERT INTO emp (NAME) VALUES ('JOE')");
batch.add("INSERT INTO emp (NAME) VALUES ('JANE')");

connection.batch(batch, res -> {
  if (res.succeeded()) {
    List<Integer> result = res.result();
  } else {
    // 失败!
  }
});
```

预编译或者调用语句将会根据参数列表，来重复使用 SQL 语句，例如：

```java
List<JsonArray> batch = new ArrayList<>();
batch.add(new JsonArray().add("joe"));
batch.add(new JsonArray().add("jane"));

connection.batchWithParams("INSERT INTO emp (name) VALUES (?)", batch, res -> {
  if (res.succeeded()) {
    List<Integer> result = res.result();
  } else {
    // Failed!
  }
});
```

### 执行其他操作

若需要执行其他数据库操作，例如您可以使用 [`execute`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#execute-java.lang.String-io.vertx.core.Handler-) 方法来执行 `CREATE TABLE` 语句。

SQL语句传给数据库时，不会经过任何处理。操作结束时将调用回调方法。

```java
String sql = "CREATE TABLE PEOPLE (ID int generated by default as identity (start with 1 increment by 1) not null," +
  "FNAME varchar(255), LNAME varchar(255), SHOE_SIZE int);";

connection.execute(sql, execute -> {
  if (execute.succeeded()) {
    System.out.println("Table created !");
  } else {
    // 失败!
  }
});
```

### 返回多个结果集

某些情况下，您的查询语句可能返回多个结果集 `ResultSet`，此时，返回的结果集会被转成纯 JSON，并且为了保持稳定性，下一个 `ResultSet` 被作为当前 `ResultSet` 的 `next` 属性链接着。一种简单的遍历所有结果集的方式如下：

```java
while (rs != null) {
  // do something with the result set...

  // next step
  rs = rs.getNext();
}
```

### Streaming

在处理大数据结果集时，不建议使用上面提到的API，而是使用数据流（stream data）的方式。因为它能够避免把所有的返回值加载到内存中，而且得到的 JSON 格式的数据也能够一行行的处理，例如：

```java
connection.queryStream("SELECT * FROM large_table", stream -> {
  if (stream.succeeded()) {
    stream.result().handler(row -> {
      // 处理 row...
    });
  }
});
```

您还可以控制 Stream 何时停止，何时恢复，何时结束。对于查询返回多个结果集的情况，您应该使用 ended event 来获得下一个结果集。如果有，Stream 将会得到新的结果集，若没有，将会调用结束方法。

```java
connection.queryStream("SELECT * FROM large_table; SELECT * FROM other_table", stream -> {
  if (stream.succeeded()) {
    SQLRowStream sqlRowStream = stream.result();

    sqlRowStream
      .resultSetClosedHandler(v -> {
        // 如果有新的结果集，将会重启 stream
        sqlRowStream.moreResults();
      })
      .handler(row -> {
        // 处理row
      })
      .endHandler(v -> {
        // 没有新的结果集需要处理
      });
  }
});
```

### 使用事务

要使用事务，首先要用  [`setAutoCommit`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#setAutoCommit-boolean-io.vertx.core.Handler-) 方法设置 auto-commit 为 `false`。

然后您就可以执行在同一个事务中的操作，在需要提交事务时，调用 [`commit`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#commit-io.vertx.core.Handler-) 方法；在需要回滚时，调用 [`rollback`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#rollback-io.vertx.core.Handler-) 方法。

一旦 `commit`/`rollback` 方法执行结束，将会调用回调方法。然后下一个事务也将自动开始。

```java
connection.commit(res -> {
  if (res.succeeded()) {
    // 事物提交成功!
  } else {
    // 事物提交失败!
  }
});
```

### 关闭连接

您在用完连接后，必须使用 [`close`](http://vertx.io/docs/apidocs/io/vertx/ext/sql/SQLConnection.html#close-io.vertx.core.Handler-) 方法把连接返回给连接池。

---

> [原文档](http://vertx.io/docs/vertx-sql-common/java/)更新于2017-03-15 15:54:14 CET
