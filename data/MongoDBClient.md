# Vert.x MongoDB Client

> 原文档：[Vert.x MongoDB Client](http://vertx.io/docs/vertx-mongo-client/java/)

## 组件介绍

您的 Vert.x 应用可以使用 Vert.x MongoDB Client（以下简称客户端）来与 MongoDB 进行交互，包括保存，获取，搜索和删除文档。

MongoDB 是在 Vert.x 应用进行数据持久化时的最佳选择，因为 MongoDB 天生就是处理 JSON（BSON）格式的文档数据库。

**特点**

- 完全非阻塞
- 支持自定义编解码器，从而实现 Vert.x JSON 快速序列化和反序列化
- 支持 MongoDB Java 驱动大部分配置项

本客户端基于 [MongoDB Async Driver](http://mongodb.github.io/mongo-java-driver/3.2/driver-async/getting-started)。

## 使用 Vert.x MongoDB Client

使用此客户端，需要添加下列依赖：

- Maven (在 `pom.xml` 文件中):

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-mongo-client</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-mongo-client:3.4.1'
```

## 创建客户端

您有以下几种方式来创建客户端：

### 默认使用共享的连接池

大部分情况下，您希望不同的客户端之间共享一个连接池。

例如：我们在部署 Verticle 时，设置了 Verticle 拥有多个实例化的对象，但是我们希望每个 Verticle 实例能够共享同一个数据源，而不是单独为每个 Verticle 实例设置不同的数据源。

要解决上面的问题，我们可以这么做：

```java
MongoClient client = MongoClient.createShared(vertx, config);
```

只有在第一次调用 [`MongoClient.createShared`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#createShared-io.vertx.core.Vertx-io.vertx.core.json.JsonObject-) 方法的时候，才会真正的根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的客户端对象，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

### 指定数据源名称

您还可以像下面这样，在创建一个客户端的时候指定数据源的名称：

```java
MongoClient client = MongoClient.createShared(vertx, config, "MyPoolName");
```

如果不同的客户端对象使用了相同的 Vert.x 对象和相同的数据源名称，那么它们将共享数据源。

同样的（与默认使用共享的数据源），只有在第一次调用`MongoClient.createShared`方法的时候，才会真正的根据 `config` 参数创建一个数据源。

之后再调用此方法，只会返回一个新的客户端对象，但使用的是相同的数据源。这时 `config` 参数也就不再有作用。

当我们希望不同含义的客户端对象拥有不同的数据源时，可以采用这种方式来创建它的对象。比如它们要与不同的数据源进行交互。

### 创建不共享数据源的客户端对象

在大部分情况下，我们会希望在不同的客户端对象之间共享数据源。但有时候，却恰恰相反。

这时，可以调用 `MongoClient.createNonShared` 方法：

```java
MongoClient client = MongoClient.createNonShared(vertx, config);
```

每次调用此方法，就相当于在调用 `MongoClient.createShared` 方法时加上了具有唯一名称的数据源参数。

## 使用客户端 API

[`MongoClient`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html) 接口定义了操作客户端的API 方法。您可以使用 [`MongoClient`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html) 来使用调用 API 方法。

### 保存文档

您可以使用 [`save`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#save-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来保存文档。

如果文档中没有 `_id` 字段，文档会被保存。若有，将执行 _upserted_。Upserted 意思是，如果此文档不存在，就保存此文档，此文档存在就更新。

如果文档没有 `_id` 字段，回调方法中可以获得保存后生成的 id 。

例如：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit");

mongoClient.save("books", document, res -> {

  if (res.succeeded()) {

    String id = res.result();
    System.out.println("Saved book with id " + id);

  } else {
    res.cause().printStackTrace();
  }

});
```

下面的例子，文档已有 `_id` ：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit").put("_id", "123244");

mongoClient.save("books", document, res -> {

  if (res.succeeded()) {

    // ...

  } else {
    res.cause().printStackTrace();
  }

});
```

### 插入文档

您可以使用 [`insert`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#insert-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来插入文档。

如果被插入的文档没有包含 id，回调方法中可以获得保存后生成的 id 。

```java
JsonObject document = new JsonObject().put("title", "The Hobbit");

mongoClient.insert("books", document, res -> {

  if (res.succeeded()) {

    String id = res.result();
    System.out.println("Inserted book with id " + id);

  } else {
    res.cause().printStackTrace();
  }

});
```

如果被插入的文档包含 id，但是此 id 代表的文档已经存在，插入就会失败：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit").put("_id", "123244");

mongoClient.insert("books", document, res -> {

  if (res.succeeded()) {

    //...

  } else {

    // Will fail if the book with that id already exists.
  }

});
```

### 更新文档

您可以使用 `update` 方法来更新文档（译者注：建议使用 `updateCollection` 方法或者 `updateCollectionWithOptions` 方法）。

此方法可以更新集合（译者注：MongoDB 中的集合概念对应 SQL 中的数据库表）中的一个或多个文档。其中 `update` 参数的 JSON 对象，必须包含 [Update Operators](http://docs.mongodb.org/manual/reference/operator/update-field/) ，因为由它决定更新的方式。

其中 `query` 参数决定更新集合中的哪个文档。

例如更新 books 集合中的一个文档：

```java
JsonObject query = new JsonObject().put("title", "The Hobbit");

// Set the author field
JsonObject update = new JsonObject().put("$set", new JsonObject().put("author", "J. R. R. Tolkien"));

mongoClient.update("books", query, update, res -> {

  if (res.succeeded()) {

    System.out.println("Book updated !");

  } else {

    res.cause().printStackTrace();
  }

});
```

您可以使用 [`updateWithOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#updateWithOptions-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.json.JsonObject-io.vertx.ext.mongo.UpdateOptions-io.vertx.core.Handler-) 方法。

如果要指定 update 操作到底是 upsert（upsert 意思是，如果此文档不存在，就保存此文档；此文档存在就更新）或者仅仅只更新，请使用 `updateWithOptions` 方法并传递参数 [`UpdateOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/UpdateOptions.html).

参数 `UpdateOptions` 有以下选项：

- `multi`

  若设置为 true，则可以更新多个文档

- `upsert`

  若设置为 true，则可以在没有查询到要更新的文档时，新增该文档

- `writeConcern`

  写操作的可靠性（译者注：源码中是用 `writeOption` 枚举类型来代表的）

```java
//译者注：MongoDB 默认写操作级别是 WriteOption.ACKNOWLEDGED

JsonObject query = new JsonObject().put("title", "The Hobbit");

// Set the author field
JsonObject update = new JsonObject().put("$set", new JsonObject().put("author", "J. R. R. Tolkien"));

UpdateOptions options = new UpdateOptions().setMulti(true);

mongoClient.updateWithOptions("books", query, update, options, res -> {

  if (res.succeeded()) {

    System.out.println("Book updated !");

  } else {

    res.cause().printStackTrace();
  }

});
```

### 替换文档

您可以使用 [`replace`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#replace-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来替换文档

`replace` 方法和 `update` 方法类似，但是并不需要 `update` 中的 `UpdateOptions` 参数。因为 `replace` 方法替换的是 `query` 参数找到的整个文档。

例如替换 books 集合中的一个文档：

```java
JsonObject query = new JsonObject().put("title", "The Hobbit");

JsonObject replace = new JsonObject().put("title", "The Lord of the Rings").put("author", "J. R. R. Tolkien");

mongoClient.replace("books", query, replace, res -> {

  if (res.succeeded()) {

    System.out.println("Book replaced !");

  } else {

    res.cause().printStackTrace();

  }

});
```

### 查找文档

您可以使用 [`find`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#find-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法查找文档。

其中 `query` 参数用来匹配集合中的文档。

例如匹配所有文档：

```java
JsonObject query = new JsonObject();

mongoClient.find("books", query, res -> {

  if (res.succeeded()) {

    for (JsonObject json : res.result()) {

      System.out.println(json.encodePrettily());

    }

  } else {

    res.cause().printStackTrace();

  }

});
```

又例如匹配 books 集合中某一个作者的所有文档：

```java
JsonObject query = new JsonObject().put("author", "J. R. R. Tolkien");

mongoClient.find("books", query, res -> {

  if (res.succeeded()) {

    for (JsonObject json : res.result()) {

      System.out.println(json.encodePrettily());

    }

  } else {

    res.cause().printStackTrace();

  }

});
```

查询的结果包装成了 JSON 对象的 List 集合。

如果您需要指定返回哪些域，又或者需要指定返回的数据条数，可以使用 `findWithOptions` 方法，在参数 `FindOptions` 中指定这些查询要求。

`FindOptions` 中可以设置以下参数：

- `fields`

  指定返回哪些域。默认值为 `null` ，意味着返回所有域。

- `sort`

  指定排序字段。默认为 `null`。

- `limit`

  指定返回的数据条数。默认值为 `-1`，意味着返回所有数据。

- `skip`

  表示返回的结果，跳过的数据行数。默认值为 `0` 。

```java
JsonObject query = new JsonObject().put("author", "J. R. R. Tolkien");

mongoClient.findBatch("book", query, res -> {

  if (res.succeeded()) {

    if (res.result() == null) {

      System.out.println("End of research");

    } else {

      System.out.println("Found doc: " + res.result().encodePrettily());

    }

  } else {

    res.cause().printStackTrace();

  }
});
```

所有返回的结果，都可以在回调方法中得到。

### 查询单个文档

要查询单个文档，您可以使用 `findOne` 方法。

这有点类似 `find` 方法，但是仅仅返回 `find` 方法查询到的第一条数据。

### 删除文档

您可以使用 [`removeDocuments`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#removeDocuments-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来删除文档。

其中 `query` 参数决定了要删除集合中的哪些文档。

例如删除作者为 Tolkien 的所有文档：

```java
JsonObject query = new JsonObject().put("author", "J. R. R. Tolkien");

mongoClient.remove("books", query, res -> {

  if (res.succeeded()) {

    System.out.println("Never much liked Tolkien stuff!");

  } else {

    res.cause().printStackTrace();

  }
});
```

### 删除单个文档

您可以使用 [`removeDocument`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#removeDocument-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来删除单个文档。

这有点类似 [`removeDocuments`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#removeDocuments-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法，但是 `removeDocument` 方法只返回匹配到的第一个文档。

### 文档计数

您可以使用 `count` 方法来计算文档数量。

例如计算作者为 Tolkien 的书的数量，结果包装在回调方法中。

```java
JsonObject query = new JsonObject().put("author", "J. R. R. Tolkien");

mongoClient.count("books", query, res -> {

  if (res.succeeded()) {

    long num = res.result();

  } else {

    res.cause().printStackTrace();

  }
});
```

### 管理 MongoDB 集合

MongoDB 的所有文档数据都存储在集合中。

您可以使用 [`getCollections`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#getCollections-io.vertx.core.Handler-) 方法来获得所有的集合：

```java
mongoClient.getCollections(res -> {

  if (res.succeeded()) {

    List<String> collections = res.result();

  } else {

    res.cause().printStackTrace();

  }
});
```

您也可以使用 [`createCollection`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#createCollection-java.lang.String-io.vertx.core.Handler-) 方法来创建一个新的集合：

```java
mongoClient.createCollection("mynewcollectionr", res -> {

  if (res.succeeded()) {

    // Created ok!

  } else {

    res.cause().printStackTrace();

  }
});
```

您也可以使用 [`dropCollection`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#dropCollection-java.lang.String-io.vertx.core.Handler-) 方法来删除文档

> 请注意：*删除一个集合将会删除集合中所有的文档！*

```java
mongoClient.dropCollection("mynewcollectionr", res -> {

  if (res.succeeded()) {

    // Dropped ok!

  } else {

    res.cause().printStackTrace();

  }
});
```

### 执行 MongoDB 其他命令

您可以使用 [`runCommand`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html#runCommand-java.lang.String-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法来执行任何 MongoDB 命令。

使用这种方式，可以发挥出 MongoDB 更多优点，比如使用 MapReduce。更多详情，请参考说明文档  [Commands](http://docs.mongodb.org/manual/reference/command)。

例如执行 aggregate（译者注：聚合）命令。请注意，命令的名称要做为 `runCommand` 方法的一个参数，并且同时也必须包含在包装命令的 JSON 参数中。这么做是因为 JSON 不是有序的，但 BSON 却是，而且 MongoDB 期望 BSON 参数的第一个键值对是命令的名称。所以，为了明确 JSON 中的哪个键值对是命令名称，我们也就必须把命令名称单独设置为一个参数：

```java
JsonObject command = new JsonObject()
  .put("aggregate", "collection_name")
  .put("pipeline", new JsonArray());

mongoClient.runCommand("aggregate", command, res -> {
  if (res.succeeded()) {
    JsonArray resArr = res.result().getJsonArray("result");
    // etc
  } else {
    res.cause().printStackTrace();
  }
});
```

### MongoDB 扩展的 JSON 支持

目前，MongoDB 只支持 date，oid 和 binary 类型（请参考：[http://docs.mongodb.org/manual/reference/mongodb-extended-json](http://docs.mongodb.org/manual/reference/mongodb-extended-json) ）

例如插入含有 date 类型字段的文档：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit")
  //ISO-8601 date
  .put("publicationDate", new JsonObject().put("$date", "1937-09-21T00:00:00+00:00"));

mongoService.save("publishedBooks", document, res -> {

  if (res.succeeded()) {

    String id = res.result();

    mongoService.findOne("publishedBooks", new JsonObject().put("_id", id), null, res2 -> {
      if (res2.succeeded()) {

        System.out.println("To retrieve ISO-8601 date : "
                + res2.result().getJsonObject("publicationDate").getString("$date"));

      } else {
        res2.cause().printStackTrace();
      }
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

例如插入含有 binary 类型字段的文档以及读取这个字段

```java
byte[] binaryObject = new byte[40];

JsonObject document = new JsonObject()
        .put("name", "Alan Turing")
        .put("binaryStuff", new JsonObject().put("$binary", binaryObject));

mongoService.save("smartPeople", document, res -> {

  if (res.succeeded()) {

    String id = res.result();

    mongoService.findOne("smartPeople", new JsonObject().put("_id", id), null, res2 -> {
      if(res2.succeeded()) {

        byte[] reconstitutedBinaryObject = res2.result().getJsonObject("binaryStuff").getBinary("$binary");
        //This could now be de-serialized into an object in real life
      } else {
        res2.cause().printStackTrace();
      }
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

例如保存一个 base 64 编码的字符串，将这个字符串作为 binary 字段插入。并且读取这个字段：

```java
String base64EncodedString = "a2FpbHVhIGlzIHRoZSAjMSBiZWFjaCBpbiB0aGUgd29ybGQ=";

JsonObject document = new JsonObject()
        .put("name", "Alan Turing")
        .put("binaryStuff", new JsonObject().put("$binary", base64EncodedString));

mongoService.save("smartPeople", document, res -> {

  if (res.succeeded()) {

    String id = res.result();

    mongoService.findOne("smartPeople", new JsonObject().put("_id", id), null, res2 -> {
      if(res2.succeeded()) {

        String reconstitutedBase64EncodedString = res2.result().getJsonObject("binaryStuff").getString("$binary");
        //This could now converted back to bytes from the base 64 string
      } else {
        res2.cause().printStackTrace();
      }
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

例如插入一个 object ID 并且读取它：

```java
String individualId = new ObjectId().toHexString();

JsonObject document = new JsonObject()
        .put("name", "Stephen Hawking")
        .put("individualId", new JsonObject().put("$oid", individualId));

mongoService.save("smartPeople", document, res -> {

  if (res.succeeded()) {

    String id = res.result();

    mongoService.findOne("smartPeople", new JsonObject().put("_id", id), null, res2 -> {
      if(res2.succeeded()) {
        String reconstitutedIndividualId = res2.result().getJsonObject("individualId").getString("$oid");
      } else {
        res2.cause().printStackTrace();
      }
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

例如获取 distinct 后的值：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit");

mongoClient.save("books", document, res -> {

  if (res.succeeded()) {

    mongoClient.distinct("books", "title", String.class.getName(), res2 -> {
      System.out.println("Title is : " + res2.result().getJsonArray(0));
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

例如获取批量模式下 distinct 的值：

```java
JsonObject document = new JsonObject().put("title", "The Hobbit");

mongoClient.save("books", document, res -> {

  if (res.succeeded()) {

    mongoClient.distinctBatch("books", "title", String.class.getName(), res2 -> {
      System.out.println("Title is : " + res2.result().getString("title"));
    });

  } else {
    res.cause().printStackTrace();
  }

});
```

## 客户端参数配置

此客户端把配置参数放在 JSON 对象中。

此客户端支持以下这些参数：

- `db_name`

  使用的 mongoDB 实例的数据库名称。默认是 `default_db`

- `useObjectId`

  此参数用来支持 ObjectId 的持久化和检索。如果设置为 `true` ，将会在集合的文档中，以 16 进制的字符串来保存 MongoDB 的 ObjectId 类型的字段。而且在设置为 `true` 后，可以让文档基于创建时间排序（译者注：前4个字节用来存储创建的时的时间戳，精确到秒）。您也可以通过使用 `ObjectId::getDate()` 方法，从这个 16进制的字符串中导出创建时间。若您选择其他类型作为 `_id` ，则设置此参数为 `false` 。如果您保存的文档中，没有设置 `_id` 字段的值，将会默认的生成 16进制的字符串作为 `_id` 。此参数默认为 `false `。

此客户端尝试着支持驱动所支持的大多数参数配置。有两种配置方式，一种是连接字符串，另一种是驱动配置选项。

> 请注意：如果使用了字符串连接的方式，此客户端将会忽略所有配置选项。

- `connection_string`

  连接字符串，指的是创建客户端的字符串，例如：`mongodb://localhost:27017` 。有关连接字符串格式的更多信息，请参考驱动程序文档。

**驱动配置的具体选项**

```text
{
  // Single Cluster Settings
  "host" : "127.0.0.1", // string
  "port" : 27017,      // int

  // Multiple Cluster Settings
  "hosts" : [
    {
      "host" : "cluster1", // string
      "port" : 27000       // int
    },
    {
      "host" : "cluster2", // string
      "port" : 28000       // int
    },
    ...
  ],
  "replicaSet" :  "foo",    // string
  "serverSelectionTimeoutMS" : 30000, // long

  // Connection Pool Settings
  "maxPoolSize" : 50,                // int
  "minPoolSize" : 25,                // int
  "maxIdleTimeMS" : 300000,          // long
  "maxLifeTimeMS" : 3600000,         // long
  "waitQueueMultiple"  : 10,         // int
  "waitQueueTimeoutMS" : 10000,      // long
  "maintenanceFrequencyMS" : 2000,   // long
  "maintenanceInitialDelayMS" : 500, // long

  // Credentials / Auth
  "username"   : "john",     // string
  "password"   : "passw0rd", // string
  "authSource" : "some.db"   // string
  // Auth mechanism
  "authMechanism"     : "GSSAPI",        // string
  "gssapiServiceName" : "myservicename", // string

  // Socket Settings
  "connectTimeoutMS" : 300000, // int
  "socketTimeoutMS"  : 100000, // int
  "sendBufferSize"    : 8192,  // int
  "receiveBufferSize" : 8192,  // int
  "keepAlive" : true           // boolean

  // Heartbeat socket settings
  "heartbeat.socket" : {
  "connectTimeoutMS" : 300000, // int
  "socketTimeoutMS"  : 100000, // int
  "sendBufferSize"    : 8192,  // int
  "receiveBufferSize" : 8192,  // int
  "keepAlive" : true           // boolean
  }

  // Server Settings
  "heartbeatFrequencyMS" :    1000 // long
  "minHeartbeatFrequencyMS" : 500 // long
}
```

**驱动参数说明**

- `host`

  mongoDB 实例运行的地址。默认是 `127.0.0.1`。 如果设置了 `hosts` 参数，就会忽略`host` 参数

- `port`

  mongoDB 实例监听的端口。默认是 `27017`。如果设置了 `hosts` 参数，就会忽略 `port` 参数

- `hosts`

  表示支持 MongoDB 集群（分片／复制）的一组地址和端口

- `host`

  集群中某个运行实例的地址

- `port`

  集群中某个运行实例监听的端口

- `replicaSet`

  某个 mongoDB 实例作为副本集的名称

- `serverSelectionTimeoutMS`

  驱动选择服务器的最大时间，单位毫秒

- `maxPoolSize`

  连接池最大连接数。默认为 `100`

- `minPoolSize`

  连接池最小连接数。默认为 `100`

- `maxIdleTimeMS`

  连接池的连接最大空闲时间。默认为 `0`，表示一直存在

- `maxLifeTimeMS`

  连接池的连接最大存活时间。默认为 `0`，表示永远存活。

- `waitQueueMultiple`

  连接池中最大等待连接数。默认为 `500`

- `waitQueueTimeoutMS`

  线程等待作为连接的最长等待时间。默认为 `120000`（2分钟）

- `maintenanceFrequencyMS`

  维护任务进行循环检查连接的时间间隔（译者注：维护任务会定时检查连接的状态，直到连接池剩下最小连接数）。默认为 `0`

- `maintenanceInitialDelayMS`

  连接池启动后，维护任务第一次启动的时间。默认为 `0`。

- `username`

  授权的用户名。默认为 `null`（意味着不需要授权）

- `password`

  授权的密码

- `authSource`

  与授权用户关联的数据库名称。默认值为 `db_name`

- `authMechanism`

  所使用的授权认证机制。请参考 [Authentication](http://docs.mongodb.org/manual/core/authentication/)  来获取更多信息。

- `gssapiServiceName`

  当使用 `GSSAPI` 的授权机制时，所使用的 Kerberos 服务名。

- `connectTimeoutMS`

  打开连接超时的时间，单位毫秒。默认为`10000`（10 秒）

- `socketTimeoutMS`

  在 socket 上接收或者发送超时的时间。默认为 `0` ，意味着永远不超时（译者注：这是客户端的超时时间。如果一个 *insert* 达到了 socketTimeoutMS， 将无法得知服务器是否已写入）。

- `sendBufferSize`

  设置 socket 发送缓冲区大小（SO_SNDBUF）。默认为 `0`，这将使用操作系统默认大小。

- `receiveBufferSize`

  设置 socket 接收缓冲区大小（SO_RCVBUF）。默认为 `0`，这将使用操作系统默认大小。

- `keepAlive`

  设置是否复用 socket（SO_KEEPALIVE）连接。默认为 `false`

- `heartbeat.socket`

  配置集群监视器监控 MongoDB 集群的 socket 连接情况的参数

- `heartbeatFrequencyMS`

  集群监视器访问每个集群服务器的频率。默认为 `5000` （5s）

- `minHeartbeatFrequencyMS`

  最小心跳频率。默认为 `1000` （1s）

> 请注意：上面提到的各类参数的默认值，都是 MongoDB Java 驱动的默认值。请参考驱动文档来获取最新信息。

---

> [原文档](http://vertx.io/docs/vertx-mongo-client/java/)更新于 2017-03-15 15:54:14 CET
