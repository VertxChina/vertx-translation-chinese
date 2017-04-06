# Vert.x Service Discovery

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 组件介绍

Vert.x 提供了一个服务发现的基础组件，用来发布和发现各种类型的资源，比如服务代理、HTTP端点（endpoint）、数据源（data source）等等。

这些资源都可以称为**服务**。服务就是一个可以被发现和访问的功能，可以通过它的类型、元数据和位置来进行描述。所以，服务可以是一个数据库、一个服务代理、一个HTTP应用，以及任何你能想到的可描述、可发现、可交互的资源。它不一定是Vert.x实体，它可以是任何组件。在Vert.x 服务发现组件中，我们通过 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 来描述每个服务。

服务发现组件实现了面向服务计算中定义的服务交互。此外，在某种程度上，还提供了动态的面向服务计算交互，这样应用程序可以对各种服务的上线、下线作出反应。

一个服务提供者可以：

+  发布一个服务记录
+  将已经发布的服务记录注销
+  更新已发布服务记录的状态（下线、服务暂停等等）

一个服务消费者可以：

+ 查找各种服务
+ 绑定到某个服务（它所获取到的 [`ServiceReference`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html)）并且使用这个服务
+ 当使用完后，释放绑定的服务
+ 监听服务的上线、下线和状态变更的消息

服务消费者访问服务的步骤：

1. 查找满足它需求的服务记录
2. 取得可访问的 [`ServiceReference`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html)
3. 通过服务对象来访问服务
4. 一旦使用完后释放服务对象

如果知道服务的类型（JDBC客户端、HTTP客户端），整个过程就可以简化为通过服务类型直接获取服务对象。

从上面可以看出，服务提供者和服务消费者，通过服务记录来共享关键的信息。

服务提供者和消费者，必须创建他们自己的 `ServiceDiscovery` 实例。这些实例通过底层的分布式数据结构来协同保持服务集合的同步。

服务发现组件支持桥接的方式，来从其他服务发现技术中导入和导出服务。

## 使用Service Discovery

要使用Vert.x 服务发现组件，需要将下列依赖加入到依赖配置中文件：

+ Maven (`pom.xml`文件中)：

```xml
<dependency>
   <groupId>io.vertx</groupId>
   <artifactId>vertx-service-discovery</artifactId>
   <version>3.4.1</version>
</dependency>
```

+ Gradle (`build.gradle` 文件中)：

```groovy
compile 'io.vertx:vertx-service-discovery:3.4.1'
```

## 基本概念

本节将解释服务发现机制所涉及到的一些概念。

### 服务记录

我们用服务记录 （[`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 对象）来描述服务提供者提供的服务，它包含了服务名称、一些元数据和一个描述服务所在位置的位置对象。

服务记录的元数据、甚至位置的格式，都有赖于服务的类型（详见后续章节）。

当服务提供者准备好可以提供服务时，会发布一条服务记录，在服务停止的时候，会收回这条服务记录。

### 服务提供者和发布者

服务提供者是提供服务的实体，而发布者的职责是发布服务记录，通过该服务记录来描述服务提供者的信息。服务提供者和发布者可以是同一个实体，也可以是不同的实体。

### 服务消费者

服务消费者在Service Discovery中搜索服务，每次搜索得到的结果是0..n条服务记录（[`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)）。通过这些服务记录，消费者可以获得服务引用（[`ServiceReference`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html)）。服务引用的作用是绑定服务消费者和服务提供者。通过服务引用，消费者可以得到服务对象来使用服务，也可以通过服务引用释放服务对象。

在使用完服务后，必须释放服务引用，才能清理服务对象和更新服务使用状态。

### 服务对象

服务对象为服务消费者提供了一条获取服务的通道，它有各种实现方式，比如一个代理对象、一个客户端对象、甚至某些类型的服务可能不存在这样一个服务对象。服务对象的表现有赖于服务的类型。

由于Vert.x的多语言特性，当你从Java、Groovy或其他语言中获取服务对象的时候，可能会有差异。

### 服务类型

服务就是资源。有很多各种各样的服务，比如功能性的服务组件、数据库、REST API等等。Vert.x 服务发现组件通过服务类型的概念来处理这种差异。每种服务类型都需要定义：

+ 如何定位服务（URI、Event Bus地址、IP/DNS 等） - *location*
+ 提供服务的对象的性质（服务代理、HTTP Client、消息消费者 等） - *client*

服务发现组件提供了一些现成的服务类型，但你也可以添加自己的服务类型。

### 服务事件

每当发布或回收服务时，`Event Bus`中都会触发一个事件，这个事件包含着被修改的服务记录。

每当通过 `getReference` 方法获取一个服务引用或者通过 `release` 方法释放一个服务引用时，都会有事件发送到 Event Bus 中，用来跟踪服务的使用情况。

关于服务事件的更详细内容参考后续章节。

### 服务存储后端

服务发现组件使用Vert.x的分布式数据结构来存储服务记录。所以，集群中所有的成员都可以访问到所有的服务记录，这是服务后端的默认实现。你也可以实现自己的服务记录存储后端，只要实现 `ServiceDiscoveryBackend` 接口就可以了。比如，Vert.x还通过实现该接口提供了基于Redis的存储后端。

注意服务发现模块并不需要运行在Vert.x 集群模式下。在单机模式下，服务记录存储于本地，并且可以通过 `ServiceImporter` 来导入。

## 创建Service Discovery实例

服务发布者和服务消费者都必须通过单独创建自己的 [`ServiceDiscovery`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceDiscovery.html) 实例来使用服务发现模块：

```java
ServiceDiscovery discovery = ServiceDiscovery.create(vertx);

// 自定义配置
discovery = ServiceDiscovery.create(vertx,
      new ServiceDiscoveryOptions()
         .setAnnounceAddress("service-announce")
         .setName("my-name"));

// 做一些事情。。。

discovery.close();
```

在默认情况下，服务事件发送到Event Bus中的地址是 `vertx.discovery.announce`，你可以自己配置一个（查看服务使用章节）。

当你不再需要 `ServiceDiscovery` 对象时，不要忘记关掉它（通过 `close` 方法）。它会把你配置的不同的服务导入/导出模块都关掉，并且释放服务引用。

## 发布服务

有了 `ServiceDiscovery` 实例，就可以发布服务了。发布的流程如下：

1. 为服务提供者创建一个服务记录
2. 发布这个服务记录
3. 保存这个发布记录的引用，后面可以用来取消发布或者修改发布

你可以通过 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 类或者各种服务类型类提供的快捷方法来创建服务记录。

```java
Record record = new Record()
    .setType("eventbus-service-proxy")
    .setLocation(new JsonObject().put("endpoint", "the-service-address"))
    .setName("my-service")
    .setMetadata(new JsonObject().put("some-label", "some-value"));

discovery.publish(record, ar -> {
  if (ar.succeeded()) {
    // publication succeeded
    Record publishedRecord = ar.result();
  } else {
    // publication failed
  }
});

// Record creation from a type
record = HttpEndpoint.createRecord("some-rest-api", "localhost", 8080, "/api");
discovery.publish(record, ar -> {
  if (ar.succeeded()) {
    // publication succeeded
    Record publishedRecord = ar.result();
  } else {
    // publication failed
  }
});
```

一定要保持一个指向服务记录对象的引用，因为这个返回的服务记录会带有一个 **注册ID**。

## 取消发布的服务

要取消一个已发布的服务，可以用如下方式：

```java
discovery.unpublish(record.getRegistration(), ar -> {
  if (ar.succeeded()) {
    // Ok
  } else {
    // cannot un-publish the service, may have already been removed, or the record is not published
  }
});
```

## 查找服务

*本节讲述的是最基本的获取服务的方法。每种服务类型接口，都提供了快捷的方法，来简化获取服务的步骤。*

在服务消费端，第一步要做的事情就是查找服务记录。你可以查找并获取一条服务记录，也可以获取一批满足条件的记录。如果是获取一条记录，那么将返回第一条满足条件的服务记录。

服务消费者通过传递一个过滤器来选择服务，有两种形式的过滤器：

1. 一个接收 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 对象的函数，这个函数返回一个布尔值（就是一个 `predicate`，即判断函数）
2. 过滤器是一个JSON对象。对象中的每个条目，将会用来过滤服务记录。服务记录必须满足所有的条目要求。这些条目可以使用 `*` 号来代表必须存在某个key值，而不管value值

让我们看一些JSON过滤器的例子

```java
{ "name" = "a" } => 匹配所有名称为"a"的记录
{ "color" = "*" } => 匹配所有设置了 "color" 的记录
{ "color" = "red" } => 匹配所有"color" 值为 "red"的记录
{ "color" = "red", "name" = "a"} => 匹配所有名称为 "a", 并且"color"值为"red"的记录
```

如果JSON过滤器未设置（为空或`null`），获取时将获取到所有的服务记录。当使用函数形式时，要获取所有的服务记录，你只需要返回 `true` 而不需要管服务记录的内容。

下面是一些例子：

```java
discovery.getRecord(r -> true, ar -> {
  if (ar.succeeded()) {
    if (ar.result() != null) {
      // we have a record
    } else {
      // the lookup succeeded, but no matching service
    }
  } else {
    // lookup failed
  }
});

discovery.getRecord((JsonObject) null, ar -> {
  if (ar.succeeded()) {
    if (ar.result() != null) {
      // we have a record
    } else {
      // the lookup succeeded, but no matching service
    }
  } else {
    // lookup failed
  }
});


// Get a record by name
discovery.getRecord(r -> r.getName().equals("some-name"), ar -> {
  if (ar.succeeded()) {
    if (ar.result() != null) {
      // we have a record
    } else {
      // the lookup succeeded, but no matching service
    }
  } else {
    // lookup failed
  }
});

discovery.getRecord(new JsonObject().put("name", "some-service"), ar -> {
  if (ar.succeeded()) {
    if (ar.result() != null) {
      // we have a record
    } else {
      // the lookup succeeded, but no matching service
    }
  } else {
    // lookup failed
  }
});

// Get all records matching the filter
discovery.getRecords(r -> "some-value".equals(r.getMetadata().getString("some-label")), ar -> {
  if (ar.succeeded()) {
    List<Record> results = ar.result();
    // If the list is not empty, we have matching record
    // Else, the lookup succeeded, but no matching service
  } else {
    // lookup failed
  }
});


discovery.getRecords(new JsonObject().put("some-label", "some-value"), ar -> {
  if (ar.succeeded()) {
    List<Record> results = ar.result();
    // If the list is not empty, we have matching record
    // Else, the lookup succeeded, but no matching service
  } else {
    // lookup failed
  }
});
```


你可以获取一条服务记录，也可以通过 [`getRecords`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceDiscovery.html#getRecords-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法获取所有匹配到的服务记录。默认情况下，服务查找只会包含状态为`UP`的服务，可以通过如下方式覆盖默认设置：

+ 当使用JSON过滤器，设置`status`属性为你想要的值（或者 `*` 来接收所有的状态）
+ 当使用函数过滤器，将 [`getRecords`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceDiscovery.html#getRecords-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法的参数`includeOutOfService`设置为`true`


## 获取服务引用
当你选择好了服务记录后，你就可以获得到一个`ServiceReference`，然后得到服务对象：

```java
ServiceReference reference1 = discovery.getReference(record1);
ServiceReference reference2 = discovery.getReference(record2);

// Then, gets the service object, the returned type depends on the service type:
// For http endpoint:
HttpClient client = reference1.getAs(HttpClient.class);
// For message source
MessageConsumer consumer = reference2.getAs(MessageConsumer.class);

// When done with the service
reference1.release();
reference2.release();
```

**使用完后，不要忘记释放服务引用。**

服务引用代表了一个绑定的服务提供者。

获取服务引用的时候，可以传递一个`JsonObject`对象来配置服务对象，可以包括用来配置服务对象的各种参数。某些服务类型不需要额外的配置，有些需要（比如数据库对象）：

```java
ServiceReference reference = discovery.getReferenceWithConfiguration(record, conf);

// Then, gets the service object, the returned type depends on the service type:
// For http endpoint:
JDBCClient client = reference.getAs(JDBCClient.class);

// Do something with the client...

// When done with the service
reference.release();
```

在前面的示例中，代码中使用的是[`getAs`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html#getAs-java.lang.Class-)方法，参数是你期望获得的对象类型。如果你使用Java语言，那么可以直接用[`get`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html#get--)方法，而其他语言中，你必须传递对象类型。

## 服务类型
前面提到，服务发现使用了服务类型的概念，来封装各种服务的差异性。

目前服务发现组件提供了几种默认的服务类型：

+ [`HttpEndpoint`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/HttpEndpoint.html) - 为REST API服务提供的类型，服务对象的类型是一个配置好了host和port的`HttpClient`（其location表现为一个url）
+ [`EventBusService`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/EventBusService.html) - 服务代理，服务对象是一个代理，它的类型是所代理的接口（其location表现为一个Event Bus的address地址）
+ [`MessageSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MessageSource.html) - 消息源服务，服务对象的类型是一个`MessageConsumer`（其location表现为一个Event Bus的address地址）
+ [`JDBCDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/JDBCDataSource.html) - JDBC数据源服务，服务对象的类型是一个`JDBCClient`（该Client的配置参数，将从location、元数据和服务消费者传递的参数中获取）
+ [`RedisDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/RedisDataSource.html) - Redis数据源服务，服务对象的类型是一个`RedisClient`（该client的配置参数，将从location、元数据和服务消费者传递的参数中获取）
+ [`MongoDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MongoDataSource.html) - Mongo数据源服务，服务对象的类型一个`MongoClient`（该client的配置参数，将从location、元数据和服务消费者传递的参数中获取）

本节将详细介绍一下服务类型，以及如何使用服务发现框架已提供的几种服务类型。

### 无类型的服务

某些服务记录也可以不带有类型（`ServiceType.UNKNOWN`）。通过这种服务记录，是无法获取到服务引用的，但是你可以通过服务记录（[`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)）的`location`和`metadata`来创建连接的细节。

使用这种服务，将不会产生服务使用的事件。

### 自定义的服务类型

通过实现 [`ServiceType`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceType.html) SPI接口，可以自定义服务类型：

1. （可选）创建一个继承了 [`ServiceType`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceType.html) 的公共接口。在这个接口中，仅需要提供一些辅助方法，来简化自定义类型的使用，比如提供 `createRecord` 方法以及 `getX` 方法（这里的`X`指的是将返回的服务对象的类型）等等。可以查看 [`HttpEndpoint`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/HttpEndpoint.html) 和 [`MessageSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MessageSource.html) 等接口例子来了解这种设计。
2. 创建一个实现了 [`ServiceType`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceType.html) 接口或者第一步定义的接口的类，这个类必须有一个 `name` 方法和一个用来创建 [`ServiceReference`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html) 的方法，这个 `name` 方法返回的名称，要和关联到自定义类型的 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 的 `type` 属性一致。
3. 创建一个继承`io.vertx.ext.discovery.types.AbstractServiceReference`的类。你可以对类进行参数化，添加上你要返回的服务对象的类型信息，你必须实现`AbstractServiceReference#retrieve()`这个方法，在这个方法中创建服务对象，这个方法只会被调用一次，如果你的服务对象需要释放资源，那另外还需要覆写 `AbstractServiceReference#close()`方法。
4. 创建`META-INF/services/io.vertx.servicediscovery.spi.ServiceType`文件，并把这个文件打包到自定义类型的jar包中，在这个文件中，需要标明第二步中所创建类的全限定名。
5. 将第一步的服务接口、第二步第三步的实现类以及第四步中的服务描述文件打包成一个jar包，然后将这个jar包放到你应用的 `classpath` 中。然后，这个自定义类型就可以使用了。


### HTTP Endpoint

一个 HTTP 端点(endpoint)，就是一个REST API或可以通过HTTP请求访问的服务。HTTP Endpoint服务对象就是一个配置了host、port和ssl的 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 对象。

#### 发布HTTP Endpoint服务

要发布一个HTTP Endpoint服务，你需要一个 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 对象。你可以通过调用`HttpEndpoint.createRecord`创建这样一个服务记录对象。

下面的代码片段，展示了如何通过 `HttpEndpoint` 接口创建一个 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)：

```java
Record record1 = HttpEndpoint.createRecord(
  "some-http-service", // The service name
  "localhost", // The host
  8433, // the port
  "/api" // the root of the service
);

discovery.publish(record1, ar -> {
  // ...
});

Record record2 = HttpEndpoint.createRecord(
  "some-other-name", // the service name
  true, // whether or not the service requires HTTPs
  "localhost", // The host
  8433, // the port
  "/api", // the root of the service
  new JsonObject().put("some-metadata", "some value")
);
```

当你在容器或云上部署你的服务时，可能你不能确定公开的IP地址和端口。所以，服务的发布必须通过其他拥有这些信息的实体来进行，这通常是一个桥接对象（bridge）。

#### 调用HTTP Endpoint服务

一旦一个HTTP Endpoint服务发布好了，服务消费者就可以获取到这个服务。对应的服务对象是一个 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 实例，并且已经配置好了host和port参数。

```java
discovery.getRecord(new JsonObject().put("name", "some-http-service"), ar -> {
  if (ar.succeeded() && ar.result() != null) {
    // Retrieve the service reference
    ServiceReference reference = discovery.getReference(ar.result());
    // Retrieve the service object
    HttpClient client = reference.getAs(HttpClient.class);

    // You need to path the complete path
    client.getNow("/api/persons", response -> {

      // ...

      // Dont' forget to release the service
      reference.release();

    });
  }
});
```

你也可以使用 [`HttpEndpoint.getClient`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/HttpEndpoint.html#getClient-io.vertx.servicediscovery.ServiceDiscovery-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 这个方法，一步就完成服务查找和服务获取：

```java
HttpEndpoint.getClient(discovery, new JsonObject().put("name", "some-http-service"), ar -> {
  if (ar.succeeded()) {
    HttpClient client = ar.result();

    // You need to path the complete path
    client.getNow("/api/persons", response -> {

      // ...

      // Dont' forget to release the service
      ServiceDiscovery.releaseServiceObject(discovery, client);

    });
  }
});
```

在第二种写法中，服务对象的释放是通过 `ServiceDiscovery.releaseServiceObject` 这个方法完成的，因此在这种情况下你是不需要持有一个服务引用的。

从Vert.x 3.4.0开始，Vert.x提供了另一种更高层次封装、更方便使用的HTTP客户端 — [`WebClient`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClient.html)。你可以通过如下方式来获取一个 `WebClient` 实例：

```java
discovery.getRecord(new JsonObject().put("name", "some-http-service"), ar -> {
  if (ar.succeeded() && ar.result() != null) {
    // Retrieve the service reference
    ServiceReference reference = discovery.getReference(ar.result());
    // Retrieve the service object
    WebClient client = reference.getAs(WebClient.class);

    // You need to path the complete path
    client.get("/api/persons").send(
      response -> {

        // ...

        // Dont' forget to release the service
        reference.release();

      });
  }
});
```

另外一种写法，通过对应的服务类型接口获取的方式：

```java
HttpEndpoint.getWebClient(discovery, new JsonObject().put("name", "some-http-service"), ar -> {
  if (ar.succeeded()) {
    WebClient client = ar.result();

    // You need to path the complete path
    client.get("/api/persons")
      .send(response -> {

        // ...

        // Dont' forget to release the service
        ServiceDiscovery.releaseServiceObject(discovery, client);

      });
  }
});
```

### Event Bus 服务

Event Bus 服务是一种服务代理，是基于Event Bus实现的一种异步RPC服务。当从一个Event Bus服务中获取一个服务对象时，你实际上得到的某个服务类的服务代理。你也可以使用 [`EventBusService`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/EventBusService.html) 接口的辅助方法来获得服务代理。

注意服务代理（服务实现和服务接口）都需要用Java语言开发。

#### 发布Event Bus 服务

要发布一个Event Bus服务，你需要创建一个 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 对象：

```java
Record record = EventBusService.createRecord(
    "some-eventbus-service", // The service name
    "address", // the service address,
    "examples.MyService", // the service interface as string
    new JsonObject()
        .put("some-metadata", "some value")
);

discovery.publish(record, ar -> {
  // ...
});
```

你也可以直接传递服务接口类：

```java
Record record = EventBusService.createRecord(
"some-eventbus-service", // The service name
"address", // the service address,
MyService.class // the service interface
);

discovery.publish(record, ar -> {
// ...
});
```

#### 调用 Event Bus 服务

要调用（消费）Event Bus服务，你可以通过先获取到服务记录然后获取服务引用的方式，也可以直接通过 [`EventBusService`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/EventBusService.html) 接口，将两步合并成一次方法调用。

当使用服务引用的方式，你需要如下方式：

```java
discovery.getRecord(new JsonObject().put("name", "some-eventbus-service"), ar -> {
  if (ar.succeeded() && ar.result() != null) {
    // Retrieve the service reference
    ServiceReference reference = discovery.getReference(ar.result());
    // Retrieve the service object
    MyService service = reference.getAs(MyService.class);

    // Dont' forget to release the service
    reference.release();
  }
});
```

当使用 `EventBusService` 接口时，你可以通过如下方式获得代理对象：

```java
EventBusService.getProxy(discovery, MyService.class, ar -> {
  if (ar.succeeded()) {
    MyService service = ar.result();

    // 不要忘记释放服务对象！
    ServiceDiscovery.releaseServiceObject(discovery, service);
  }
});

```

### 消息源服务

消息源服务，就是通过Event Bus发送消息到某个地址的组件。消息源服务的Client是 [`MessageConsumer`](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html)。

消息源服务的`location`是消息所发送的Event Bus 地址。

#### 发布消息源服务

和其他服务类型一样，发布一个消息源服务包含两个步骤：

1. 通过 [`MessageSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MessageSource.html) 接口创建一条服务记录
2. 发布这条服务记录

```java
Record record = MessageSource.createRecord(
    "some-message-source-service", // The service name
    "some-address" // The event bus address
);

discovery.publish(record, ar -> {
  // ...
});

record = MessageSource.createRecord(
    "some-other-message-source-service", // The service name
    "some-address", // The event bus address
    "examples.MyData" // The payload type
);
```

在第二个 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 创建时，我们同时指明了消息体（payload）的类型，这不是必须的。

在Java中，你可以使用 `Class` 类型的参数：

```java
Record record1 = MessageSource.createRecord(
"some-message-source-service", // The service name
"some-address", // The event bus address
JsonObject.class // The message payload type
);

Record record2 = MessageSource.createRecord(
"some-other-message-source-service", // The service name
"some-address", // The event bus address
JsonObject.class, // The message payload type
new JsonObject().put("some-metadata", "some value")
);
```

#### 消费消息源服务

在服务消费端，你可以手动获取服务记录和服务引用，也可以使用 [`MessageSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MessageSource.html) 接口提供的辅助方法直接获取。

第一种方式对应的代码示例如下：

```java
discovery.getRecord(new JsonObject().put("name", "some-message-source-service"), ar -> {
  if (ar.succeeded() && ar.result() != null) {
    // Retrieve the service reference
    ServiceReference reference = discovery.getReference(ar.result());
    // Retrieve the service object
    MessageConsumer<JsonObject> consumer = reference.getAs(MessageConsumer.class);

    // Attach a message handler on it
    consumer.handler(message -> {
      // message handler
      JsonObject payload = message.body();
    });

    // ...
    // when done
    reference.release();
  }
});
```


如果使用 [`MessageSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MessageSource.html) 接口，代码如下：

```java
MessageSource.<JsonObject>getConsumer(discovery, new JsonObject().put("name", "some-message-source-service"), ar -> {
  if (ar.succeeded()) {
    MessageConsumer<JsonObject> consumer = ar.result();

    // Attach a message handler on it
    consumer.handler(message -> {
      // message handler
      JsonObject payload = message.body();
    });
    // ...

    // Dont' forget to release the service
    ServiceDiscovery.releaseServiceObject(discovery, consumer);

  }
});
```

### JDBC 数据源

数据源指的是数据库或数据存储。JDBC数据源通过JDBC驱动访问数据库，JDBC数据源服务对象是是 [`JDBCClient`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html) 实例。



#### 发布 JDBC 数据源服务

和其他服务类型一样，发布 JDBC 数据源服务共两个步骤：

1. 通过 [`JDBCDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/JDBCDataSource.html) 接口创建服务记录
2. 发布服务记录

```java
Record record = JDBCDataSource.createRecord(
    "some-data-source-service", // The service name
    new JsonObject().put("url", "some jdbc url"), // The location
    new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
```

JDBC 数据源可以代表各种类型的数据库，而这些数据库的访问方式一般是不同的，服务记录很难有统一结构。在服务记录中，`location` 由一个简单的JSON对象组成，里面包含访问数据源的各种属性（JDBC URL、用户名、密码等）。这些属性既依赖于数据库，同时也依赖于所使用的连接池。

#### 消费 JDBC 数据源服务

如前所述，访问数据源的方式依赖于数据源本身。要创建一个 [`JDBCClient`](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html)，你需要同时提供：服务记录位置信息、元数据以及服务消费者提供的JSON对象：

```java
discovery.getRecord(
    new JsonObject().put("name", "some-data-source-service"),
    ar -> {
      if (ar.succeeded() && ar.result() != null) {
        // Retrieve the service reference
        ServiceReference reference = discovery.getReferenceWithConfiguration(
            ar.result(), // The record
            new JsonObject().put("username", "clement").put("password", "*****")); // Some additional metadata

        // Retrieve the service object
        JDBCClient client = reference.getAs(JDBCClient.class);

        // ...

        // when done
        reference.release();
      }
    });
```

你也可以使用 `JDBCDataSource` 接口的辅助方法，来查询和获取服务对象：

```java
JDBCDataSource.<JsonObject>getJDBCClient(discovery,
    new JsonObject().put("name", "some-data-source-service"),
    new JsonObject().put("username", "clement").put("password", "*****"), // Some additional metadata
    ar -> {
      if (ar.succeeded()) {
        JDBCClient client = ar.result();

        // ...

        // Dont' forget to release the service
        ServiceDiscovery.releaseServiceObject(discovery, client);

      }
    });
```

### Redis 数据源

Redis 数据源服务是专门为Redis提供的服务类型，对应服务对象是 [`RedisClient`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html)。

#### 发布 Redis 数据源服务

发布一个 Redis 数据源服务共两个步骤：

1. 通过 [`RedisDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/RedisDataSource.html) 接口创建一条服务记录
2. 发布这个服务记录

```java
Record record = RedisDataSource.createRecord(
  "some-redis-data-source-service", // The service name
  new JsonObject().put("url", "localhost"), // The location
  new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
```

这里的 `location` 是一个JSON对象，包含访问Redis数据源的属性（URL、端口等）。



#### 消费 Redis 数据源服务

如前所述，访问数据源的方式依赖于数据源本身。要创建一个 [`RedisClient`](http://vertx.io/docs/apidocs/io/vertx/redis/RedisClient.html)，你需要同时提供：服务记录位置信息、元数据以及服务消费者提供的JSON对象：

```java
discovery.getRecord(
  new JsonObject().put("name", "some-redis-data-source-service"), ar -> {
    if (ar.succeeded() && ar.result() != null) {
      // Retrieve the service reference
      ServiceReference reference = discovery.getReference(ar.result());

      // Retrieve the service instance
      RedisClient client = reference.getAs(RedisClient.class);

      // ...

      // when done
      reference.release();
    }
  });
```

你也可以利用 [`RedisDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/RedisDataSource.html) 接口的辅助方法来查询和获取服务对象：

```java
RedisDataSource.getRedisClient(discovery,
  new JsonObject().put("name", "some-redis-data-source-service"),
  ar -> {
    if (ar.succeeded()) {
      RedisClient client = ar.result();

      // ...

      // Dont' forget to release the service
      ServiceDiscovery.releaseServiceObject(discovery, client);

    }
  })
```



### Mongo 数据源

Mongo 数据源服务是专门为 MongoDB 提供的一种服务类型，对应的服务对象是 [`MongoClient`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html)。

#### 发布 Mongo 数据源服务

发布一个 Mongo 数据源服务需要两步：

1. 通过 [`MongoDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MongoDataSource.html) 接口创建一条服务记录
2. 发布这条服务记录

```java
Record record = MongoDataSource.createRecord(
  "some-data-source-service", // The service name
  new JsonObject().put("connection_string", "some mongo connection"), // The location
  new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
```

其中`location`是一个JSON对象，包含了访问Mongo数据源的所有属性（URL、端口等）

#### 消费 Mongo 数据源服务

如前所述，访问数据源的方式依赖于数据源本身。要创建一个 [`MongoClient`](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html)，你需要同时提供：服务记录位置信息、元数据以及服务消费者提供的JSON对象：

```java
discovery.getRecord(
  new JsonObject().put("name", "some-data-source-service"),
  ar -> {
    if (ar.succeeded() && ar.result() != null) {
      // Retrieve the service reference
      ServiceReference reference = discovery.getReferenceWithConfiguration(
        ar.result(), // The record
        new JsonObject().put("username", "clement").put("password", "*****")); // Some additional metadata

      // Retrieve the service object
      MongoClient client = reference.get();

      // ...

      // when done
      reference.release();
    }
  });

```

你也可以利用 [`MongoDataSource`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/types/MongoDataSource.html) 接口中的辅助方法来完成服务对象的查找和获取：

```java
MongoDataSource.<JsonObject>getMongoClient(discovery,
  new JsonObject().put("name", "some-data-source-service"),
  new JsonObject().put("username", "clement").put("password", "*****"), // Some additional metadata
  ar -> {
    if (ar.succeeded()) {
      MongoClient client = ar.result();

      // ...

      // Dont' forget to release the service
      ServiceDiscovery.releaseServiceObject(discovery, client);

    }
  });
```


## 监听服务的上线与下线

每当服务发布或者取消发布，都会有相应的事件发送到 `vertx.discovery.announce` 这个地址。这个地址可以通过`ServiceDiscoveryOptions`配置。

收到的`Record`中有个`status`字段，用来表示服务的状态：

+ `UP`：服务已经可以使用了
+ `DOWN`：服务不再可用
+ `OUT_OF_SERVICE`：服务目前不可用，但是过段时间会继续提供服务。


## 监听服务的使用

每当有一个服务引用被绑定或者被释放，都会有相应的事件发送到 `vertx.discovery.usage` 这个地址。这个地址可以通过`ServiceDiscoveryOptions`配置。

通过这个事件，可以监听服务的使用和服务的映射。

收到的消息是一个包含如下内容的`JsonObject`对象：

+ 在 `record` 属性中，包含了服务记录信息
+ 在 `type` 属性中记录了事件的类型，类型分为`bind`和`release`
+ 在 `id` 属性中记录了服务发现实例的ID（服务发现实例的名称或节点ID）

其中 `id` 可以通过 `ServiceDiscoveryOptions` 进行配置。默认情况下，在单节点时它的值是`localhost`，在集群模式时是节点的ID。

你也可以通过`setUsageAddress`方法，将事件发送地址设置为`null`，这样就可以禁用服务使用情况的监听功能了。

## 服务发现桥接器

通过桥接器（bridge），你可以从其他服务发现组件中导入和导出服务，比如Docker，Kubernetes，Consul等。每种类型的桥接器，决定了服务如何导入和导出，并且不一定都是双向的。

要想自定义桥接器，你可以通过实现 [`ServiceImporter`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceImporter.html) 接口，然后再使用 [`registerServiceImporter`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceDiscovery.html#registerServiceImporter-io.vertx.servicediscovery.spi.ServiceImporter-io.vertx.core.json.JsonObject-) 方法注册一下。你可以通过第二个参数传递一些可选的配置信息给桥接器。

当桥接器注册后，[`start`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceImporter.html#start-io.vertx.core.Vertx-io.vertx.servicediscovery.spi.ServicePublisher-io.vertx.core.json.JsonObject-io.vertx.core.Future-) 方法将会被调用，这样你可以对桥接器进行一些配置。当桥接器配置好了，已经准备导入导出初始的服务时，必须 `complete` 所传递的 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html)。如果桥接器的启动方法是阻塞型的，那么就必须使用 `executeBlocking` 方法进行封装，并且`complete`所传递的 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html) 对象。

当服务发现实例被关闭的时候，对应的桥接器也一块被关闭了。执行关闭操作的时候，服务发现组件会调用 [`close`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/spi/ServiceImporter.html#close-io.vertx.core.Handler-) 方法以进行资源的释放以及移除导入/导出的服务。这个方法必须调用所传递的 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html) 的`complete`方法，来通知调用者关闭操作已经完成。

需要提醒的是，在一个集群中，只需要有一个节点注册了服务桥接器，集群中所有成员就都能使用了。

## 可用的服务发现桥接器

Vert.x 服务发现组件除了支持桥接器机制以外，还提供了一些现成的桥接器。

### Consul 桥接器

Consul 桥接器可以将 [Consul](http://consul.io/) 中的服务导入到Vert.x的服务发现组件中。

这个桥接器可以连接 Consul agent（服务器），并且会进行周期性的扫描，来更新服务情况：

+ 新的服务被导入
+ 维护模式下的服务或已经从 Consul 中移除的服务将会被移除

这个桥接器使用的是 Consul 的HTTP API接口。它不能将服务导出到Consul，并且也不支持服务的修改。

服务的类型是通过`tags`推断出来的，如果有一个`tag`和已知的服务类型一样，那么就使用这种服务类型，如果没有匹配的，那么服务导入后将标记为`unknown`类型。目前暂时只支持`http-endpoint`类型。


#### 桥接器的使用

要使用该服务发现桥接器，需要将如下的依赖包加入到依赖配置文件中：

+ Maven（在 `pox.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-consul</artifactId>
  <version>3.4.1</version>
</dependency>
```

+ Gradle（在 `build.gradle` 文件中）：
```groovy
compile 'io.vertx:vertx-service-discovery-bridge-consul:3.4.1'
```

然后，在创建服务发现对象的时候，像下面这样注册桥接器：

```java
ServiceDiscovery.create(vertx)
    .registerServiceImporter(new ConsulServiceImporter(),
        new JsonObject()
            .put("host", "localhost")
            .put("port", 8500)
            .put("scan-period", 2000));
```

你可以做一些配置：

+ `host` 属性，配置 agent 的地址，默认是`localhost`
+ `port` 属性，配置 agent 的端口，默认的端口是 8500
+ `scan-period` 属性，配置扫描的频率，扫描的单位是毫秒（ms），默认是 2000 ms

### Kubernetes 桥接器

Kubernetes 桥接器可以从Kubernetes（或者 Openshift v3）中导入服务到Vert.x的服务发现组件中。

Kubernetes的所有服务，都将映射为一条 [`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)。目前桥接器只支持将服务从Kubernetes中导入到Vert.x中（反过来不行）。

Kubernetes中的服务，在导入到Vert.x后都会创建对应的服务记录（[`Record`](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)）。服务类型是通过 `service.type.lable` 推断出来的。如果该属性没有设置，那么服务类型被设置为`unknown`。目前暂时只支持 `http-endpoint` 服务类型。

#### 桥接器的使用

要使用该服务发现桥接器，需要将如下的依赖包加入到依赖配置文件中：

+ Maven（在 `pox.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-kubernetes</artifactId>
  <version>3.4.1</version>
</dependency>
```

+ Gradle（在 `build.gradle` 文件中）：
```groovy
compile 'io.vertx:vertx-service-discovery-bridge-kubernetes:3.4.1'
```

#### 桥接器的配置

桥接器的配置项有：

+ OAuth token（默认是使用`/var/run/secrets/kubernetes.io/serviceaccount/token`中的内容）
+ 服务搜索的命名空间（默认是`default`）

请注意，应用程序必须能够访问 Kubernetes 并且能够读取所选择的命名空间。



#### 服务记录的映射

服务记录按照如下的步骤进行创建：

+ 从 `service.type` 中推断出服务类型；如果没有设置，那么服务类型被设置为`unknown`
+ 服务记录的名称就是服务的名称
+ 服务的标签（label）都被映射为服务记录的元数据
+ 此外还会加上：`kubernetes.uuid, kubernetes.namespace, kubernetes.name`
+ `location`信息将从服务的第一个端口推断出来

对于 HTTP 端点，如果服务带有`ssl`标签的话，那么服务记录的`ssl`(`https`)属性将被设置为`true`。

#### 动态性

Kubernetes 桥接器将会在启动(`start`)的时候导入所有的服务，在停止(`stop`)的时候移除所有的服务。在运行期间，它将监听 Kubernetes 的服务，并且动态地导入新加入的服务，移除被删除的服务。

### Docker Links 桥接器

Docker Links 桥接器可以从 Docker Links 中导入服务到 Vert.x 的服务发现组件中。

当你将一个Docker容器与另外一个Docker容器链接在一起(link)的时候，Docker将会注入一组环境变量。该桥接器将分析这些环境变量，并且针对每个链接(link)，生成一个服务记录。服务记录的类型从`service.type.lable`属性中推断；如果没有设置，那么服务类型将被设置为`unknown`。目前暂时只支持 `http-endpoint` 服务类型。

由于Docker容器只在启动的时候创建链接，所以这个桥接器只会在启动的时候导入服务记录，然后此后就都不改变了。

#### 桥接器的使用

要使用该服务发现桥接器，需要将如下的依赖包加入到依赖配置文件中：

+ Maven（在 `pox.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-docker-links</artifactId>
  <version>3.4.1</version>
</dependency>
```

+ Gradle（在 `build.gradle` 文件中）：
```groovy
compile 'io.vertx:vertx-service-discovery-bridge-docker-links:3.4.1'
```

然后，在创建服务发现对象的时候，像下面这样注册桥接器：

```java
ServiceDiscovery.create(vertx)
    .registerServiceImporter(new DockerLinksServiceImporter(), new JsonObject());
```

这种桥接器不需要任何进一步的配置。

## 其他存储后端

Vert.x服务发现框架还提供了一些现成的后端存储机制支持。

### Redis 存储后端

服务发现组件通过实现 `ServiceDiscoveryBackend` SPI提供了一种可插拔的存储后端扩展机制。

Vert.x Service Discovery Redis Backend组件是基于Redis的后端存储实现。


#### 使用 Redis 存储后端

要使用 Redis 存储后端，需要将如下的依赖包加入到依赖配置文件中：

+ Maven（在 `pom.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-backend-redis</artifactId>
  <version>3.4.1</version>
</dependency>
```

+ Gradle（在 `build.gradle` 文件中）：
```groovy
compile 'io.vertx:vertx-service-discovery-backend-redis:3.4.1'
```

需要注意的是，你只能在 `classpath` 中指定一个SPI的实现；如果没有指定，那么将使用默认的存储后端。

#### 配置

Redis存储后端是基于 [Vert.x Redis Client](http://vertx.io/docs/vertx-redis-client/java/) 实现的，所以配置内容和 `RedisClient` 的配置内容一致。

下面是一个示例：

```java
ServiceDiscovery.create(vertx, new ServiceDiscoveryOptions()
    .setBackendConfiguration(
        new JsonObject()
            .put("host", "127.0.0.1")
            .put("key", "records")
    ));
```

---

> [原文档](http://vertx.io/docs/vertx-service-discovery/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-service-discovery/java/
[2]: https://github.com/vert-x3/vertx-service-discovery
[3]: https://github.com/vert-x3/vertx-examples/tree/master/service-discovery-examples
