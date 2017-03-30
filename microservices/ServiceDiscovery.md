# Vert.x Service Discovery

* 引用：[http://vertx.io/docs/vertx-service-discovery/java/](http://vertx.io/docs/vertx-service-discovery/java/)

<hr>

This component provides an infrastructure to publish and discover various resources, such as service proxies, HTTP endpoints, data sources…

Vert.x Service Discovery，提供了一个基础的软件框架，用来发布和发现各种类型的资源，比如服务代理、HTTP应用、datasources等等。

These resources are called services. A service is a discoverable functionality. It can be qualified by its type, metadata, and location. So a service can be a database, a service proxy, a HTTP endpoint and any other resource you can imagine as soon as you can describe it, discover it and interact with it. It does not have to be a vert.x entity, but can be anything. Each service is described by a Record.

这些资源都可以称为服务，服务就是一个可以被发现和访问的功能，可以通过它的类型、元数据和位置来进行描述。所以，服务可以是一个数据库、一个服务代理、一个HTTP应用，以及任何你能想到的可描述、可发现、可交互的资源。它不一定是vert.x实体，它可以是任何组件。在Vert.x Service Discovery中，服务通过[服务记录](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html)  来进行描述。

The service discovery implements the interactions defined in service-oriented computing. And to some extent, also provides the dynamic service-oriented computing interactions. So, applications can react to arrival and departure of services.

Service Discovery实现了面向服务计算中定义的服务交互，此外，在某种程度上，还提供了动态的面向服务计算交互，这样应用程序可以对各种服务的获取、释放作出反应。

A service provider can:

publish a service record

un-publish a published record

update the status of a published service (down, out of service…​)

一个服务提供者可以：
+  发布一个服务记录
+  取消发布的服务记录
+  更新已发布服务记录的状态（下线、服务暂停等等） 

A service consumer can:

lookup services

bind to a selected service (it gets a ServiceReference) and use it

release the service once the consumer is done with it

listen for arrival, departure and modification of services.

一个服务消费者可以：
+ 查找各种服务
+ 绑定到某个服务（它所获取到的[服务引用](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html) ）并且使用这个服务
+ 当使用完后，释放绑定的服务
+ 监听服务的上线、下线和状态变更的消息

Consumer would 1) lookup a service record matching their need, 2) retrieve the ServiceReference that give access to the service, 3) get a service object to access the service, 4) release the service object once done.

服务消费者将会1）查找满足它需求的服务记录，2）取得可访问的[服务引用](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/ServiceReference.html) ，3）使用一个服务对象来访问服务，4）一旦使用完后释放服务对象

The process can be simplified using service type where you can directly retrieve the service object if you know from which type it is (JDBC client, Http client…​).

如果知道服务的类型（JDBC客户端、HTTP客户端），整个过程就可以简化为通过服务类型直接获取服务对象。

As stated above, the central piece of information shared by the providers and consumers are records.

从上面可以看出，服务提供者和服务消费者，通过服务记录来共享关键的信息。

Providers and consumers must create their own ServiceDiscovery instance. These instances are collaborating in the background (distributed structure) to keep the set of services in sync.

服务提供者和消费者，必须创建他们自己的ServiceDiscovery实例，这些实例通过幕后（分布式结构）的协作来保持服务的同步。

The service discovery supports bridges to import and export services from / to other discovery technologies.

Service Discovery支持桥接的方式，来从其他服务发现技术中导入和导出服务。

Using the service discovery
To use the Vert.x service discovery, add the following dependency to the dependencies section of your build descriptor:

## 使用Service Discovery
要使用Vert.x的Service Discovery，请将下列依赖加入到依赖配置中文件。


+ Maven (pom.xml文件中):

        <dependency>
           <groupId>io.vertx</groupId>
           <artifactId>vertx-service-discovery</artifactId>
           <version>3.4.1</version>
        </dependency>

+ Gradle (在 build.gradle 文件中):

         compile 'io.vertx:vertx-service-discovery:3.4.1'


Overall concepts
The discovery mechanism is based on a few concepts explained in this section.

## 基本概念
本节将解释服务发现机制所涉及到的一些概念。

Service records
A service Record is an object that describes a service published by a service provider. It contains a name, some metadata, a location object (describing where is the service). This record is the only object shared by the provider (having published it) and the consumer (retrieve it when doing a lookup).

The metadata and even the location format depend on the service type (see below).

A record is published when the provider is ready to be used, and withdrawn when the service provider is stopping.

### 服务记录
服务提供者发布的服务，通过服务记录来描述，服务记录是一个对象，它包含了名称、一些元数据、一个位置对象（描述服务所在的位置）。服务记录是服务提供者（发布了某个服务）和服务消费者（通过检索获得服务）所共享的唯一对象。

服务记录的元数据、甚至位置的格式，都有赖于服务的类型（详见后续章节）。

当服务提供者准备好可以提供服务时，会发布一条服务记录，在服务停止的时候，会收回这条服务记录。

Service Provider and publisher
A service provider is an entity providing a service. The publisher is responsible for publishing a record describing the provider. It may be a single entity (a provider publishing itself) or a different entity.

### 服务提供者和发布者
服务提供者是提供服务的实体，而发布者的职责是发布服务记录，通过该服务记录来描述服务提供者的信息，服务提供者和发布者可以是同一个实体，也可以是不同的实体。


Service Consumer
Service consumers search for services in the service discovery. Each lookup retrieves 0..n Record. From these records, a consumer can retrieve a ServiceReference, representing the binding between the consumer and the provider. This reference allows the consumer to retrieve the service object (to use the service), and release the service.

It is important to release service references to cleanup the objects and update the service usages.

###服务消费者
服务消费者在Service Discovery中搜索服务，每次搜索得到的结果是0..n条服务记录（Record），通过这些服务记录，消费者可以获得服务引用（ServiceReference），服务引用的作用是绑定服务消费者和服务提供者，然后通过服务引用，消费者可以得到服务对象来使用服务，也可以通过服务引用释放服务对象。

在使用完服务后，必须释放服务引用，才能清理服务对象和更新服务使用状态。

Service object
The service object is the object that gives access to a service. It can come in various forms, such as a proxy, a client, and may even be non-existent for some service types. The nature of the service object depends on the service type.

Notice that because of the polyglot nature of Vert.x, the service object can differ if you retrieve it from Java, Groovy or another language.

###服务对象
服务对象为服务消费者提供了一条获取服务的通道，它有各种实现方式，比如一个代理对象、一个客户端对象、甚至某些类型的服务可能不存在这样一个服务对象。服务对象的表现有赖于服务的类型。

由于Vert.x的多语言特性，因此当你从Java、Groovy或其他语言中获取服务对象的时候，可能会有差异。

Service types
Services are just resources, and there are a lot of different kinds of services. They can be functional services, databases, REST APIs, and so on. The Vert.x service discovery has the concept of service types to handle this heterogeneity. Each type defines:

how the service is located (URI, event bus address, IP / DNS…​) - location

the nature of the service object (service proxy, HTTP client, message consumer…​) - client

Some service types are implemented and provided by the service discovery component, but you can add your own.

###服务类型
服务就是资源，有很多各种各样的服务，比如功能性的服务组件、数据库、REST API等等。Vert.x的服务发现组件，通过服务类型的概念，来处理这种差异。每种类型都需要定义：
+ 如何定位服务（URI、event bus地址、IP/DNS  ...） - location
+ 提供服务的对象的性质（服务代理、HTTP client、message消费 ...） - client
  服务发现组件提供了一些现成的服务类型，但你也可以添加自己的服务类型

Service events
Every time a service provider is published or withdrawn, an event is fired on the event bus. This event contains the record that has been modified.

In addition, in order to track who is using who, every time a reference is retrieved with getReference or released with release, events are emitted on the event bus to track the service usages.

More details on these events below.

###服务事件
每当服务提供者发布一个服务或者撤回一个服务时，都会在event bus中触发一个事件，这个事件包含了一条服务记录，记录着被修改的服务信息。

此外，为了能跟踪到谁在使用什么服务，每当通过getReference获取一个服务引用或者释放一个服务引用时，都会有一个事件发送到event bus中，用来跟踪服务的使用情况。

关于服务事件的更详细内容参考后续章节。

Backend
The service discovery uses a Vert.x distributed data structure to store the records. So, all members of the cluster have access to all the records. This is the default backend implementation. You can implement your own by implementing the ServiceDiscoveryBackend SPI. For instance, we provide an implementation based on Redis.

Notice that the discovery does not require Vert.x clustering. In single-node mode, the structure is local. It can be populated with `ServiceImporter`s.

###服务后台
Service Discovery组件使用Vert.x的分布式数据组件来存储服务记录，所以，集群中所有的成员都可以访问到所有的服务记录，这是服务后台的默认实现。你也可以实现自己的服务后台，只要实现`ServiceDiscoveryBackend`接口就可以了。比如，Vert.x还通过实现该接口提供了基于Redis的服务后台。

此外，服务发现并不仅限于Vert.x集群，带单节点模式下，数据是存储在本地的，可以通过`ServiceImporter`来导入。

Creating a service discovery instance
Publishers and consumers must create their own ServiceDiscovery instance to use the discovery infrastructure:

## 创建Service Discovery实例
服务发布和服务消费，都必须通过创建一个私有的`ServiceDiscovery`来使用服务发现框架。

```java
    ServiceDiscovery discovery = ServiceDiscovery.create(vertx);
      
    // Customize the configuration
    discovery = ServiceDiscovery.create(vertx,
        new ServiceDiscoveryOptions()
            .setAnnounceAddress("service-announce")
            .setName("my-name"));
    
    // Do something...
    
    discovery.close();
```

By default, the announce address (the event bus address on which service events are sent is: vertx.discovery .announce. You can also configure a name used for the service usage (see section about service usage).

When you don’t need the service discovery object anymore, don’t forget to close it. It closes the different discovery importers and exporters you have configured and releases the service references.

You should avoid sharing the service discovery instance, so service usage would represent the right "usages".

服务事件默认情况下发送到event bus中的地址是`vertx.discovery .announce`，你可以自己配置一个（查看服务使用章节）

当你不再需要`Service Discovery`对象时，不要忘记close掉它，它会把你配置的各种服务发现中用到的输入输出都关闭掉，并且释放服务引用。

Publishing services
Once you have a service discovery instance, you can publish services. The process is the following:

create a record for a specific service provider

publish this record

keep the published record that is used to un-publish a service or modify it.

To create records, you can either use the Record class, or use convenient methods from the service types.

##发布服务
有了Service Discovery实例，就可以发布服务了，发布的流程如下：
1. 为服务提供者创建一个服务记录
2. 发布这个服务记录
3. 保存这个发布记录的引用，后面可以用来取消发布或者修改发布

创建服务记录，可以通过`Record` 类，或者服务类型所提供的快捷方法。

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

It is important to keep a reference on the returned records, as this record has been extended by a registration id.

一定要保持一个指向服务记录对象的引用，因为这个返回的服务记录，会带有一个注册Id。


Withdrawing services
To withdraw (un-publish) a record, use:

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


Looking for services
This section explains the low-level process to retrieve services, each service type provide convenient method to aggregates the different steps.

On the consumer side, the first thing to do is to lookup for records. You can search for a single record or all the matching ones. In the first case, the first matching record is returned.

##查找服务
*本节讲述的是最基本的获取服务的方法，每种服务类型，都提供了快捷的方法，来简化获取服务的步骤。*

在服务消费端，第一步要做的事情就是查找服务记录，可以查找并获取一条服务记录，也也可以获取一批满足条件的记录，如果是获取一条记录，那么将返回第一条满足条件的服务记录。

Consumer can pass a filter to select the service. There are two ways to describe the filter:

A function taking a Record as parameter and returning a boolean (it’s a predicate)

This filter is a JSON object. Each entry of the given filter is checked against the record. All entries must exactly match the record. The entry can use the special * value to denote a requirement on the key, but not on the value.

服务消费者通过传递一个过滤器来选择服务，有两种形式的过滤器：
1. 一个接收`Record`对象的函数，这个函数返回一个布尔值（就是一个`predicate`）
2. 过滤器是一个JSON对象，对象中的每个条目，将会用来过滤服务记录，服务记录必须满足所有的条目要求。这些条目，可以使用 * 号来代表必须存在某个key值，而不管value值

Let’s see an example of a JSON filter:

{ "name" = "a" } => matches records with name set to "a"
{ "color" = "*" } => matches records with "color" set
{ "color" = "red" } => only matches records with "color" set to "red"
{ "color" = "red", "name" = "a"} => only matches records with name set to "a", and color set to "red"

让我们看一些JSON过滤器的例子

```java
{ "name" = "a" } => 匹配所有名称为"a"的记录
{ "color" = "*" } => 匹配所有设置了 "color" 的记录
{ "color" = "red" } => 匹配所有"color" 值为 "red"的记录
{ "color" = "red", "name" = "a"} => 匹配所有名称为 "a", 并且"color"值为"red"的记录
```

If the JSON filter is not set (null or empty), it accepts all records. When using functions, to accept all records, you must return true regardless the record.

Here are some examples:

如果JSON过滤器设置为空，那将获取到所有的服务记录。当使用函数形式时，要获取所有的服务记录，你只需要返回true，不管是`Record`参数是什么。

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

You can retrieve a single record or all matching records with getRecords. By default, record lookup does include only records with a status set to UP. This can be overridden:

when using JSON filter, just set status to the value you want (or * to accept all status)

when using function, set the includeOutOfService parameter to true in getRecords .

你可以获取一条服务记录，也可以通过`getRecords`方法获取所有匹配到的服务记录。默认情况下，服务查找只会包含状态为`UP`的服务，可以通过如下方式覆盖默认设置：
+ 当使用JSON过滤器，设置`status`属性为你想要的值（或者 * 来接收所有的状态）
+ 当使用函数过滤器，将`getRecords`方法的参数`includeOutOfService`设置为`true`

Retrieving a service reference
Once you have chosen the Record, you can retrieve a ServiceReference and then the service object:

## 获取服务引用
当你选择好了服务记录后，你就可以获得到一个`ServiceReference`，然后得到服务对象。

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
Don’t forget to release the reference once done.

The service reference represents a binding with the service provider.

When retrieving a service reference you can pass a JsonObject used to configure the service object. It can contain various data about the service object. Some service types do not need additional configuration, some require configuration (as data sources):

使用完后，不要忘记释放服务引用。

服务引用代表了一个绑定的服务提供者。

获取服务引用的时候，可以传递一个`JsonObject`对象来配置服务对象，可以包括用来配置服务对象的各种参数。某些服务类型不需要额外的配置，有些需要（比如数据库对象）。

```java

ServiceReference reference = discovery.getReferenceWithConfiguration(record, conf);

// Then, gets the service object, the returned type depends on the service type:
// For http endpoint:
JDBCClient client = reference.getAs(JDBCClient.class);

// Do something with the client...

// When done with the service
reference.release();

```

In the previous examples, the code uses getAs. The parameter is the type of object you expect to get. If you are using Java, you can use get. However in the other language you must pass the expected type.

在前面的示例中，代码中使用的是`getAs`方法，参数是你期望获得的对象类型，如果你使用Java语言，那么可以直接用`get`，而其他语言中，你必须传递对象类型。


Types of services
A said above, the service discovery has the service type concept to manage the heterogeneity of the different services.

These types are provided by default:

##服务类型
前面提到，服务发现使用了服务类型的概念，来封装各种服务的差异性。
目前提供了几种默认的服务类型：

HttpEndpoint - for REST API’s, the service object is a HttpClient configured on the host and port (the location is the url).

EventBusService - for service proxies, the service object is a proxy. Its type is the proxies interface (the location is the address).

MessageSource - for message sources (publisher), the service object is a MessageConsumer (the location is the address).

JDBCDataSource - for JDBC data sources, the service object is a JDBCClient (the configuration of the client is computed from the location, metadata and consumer configuration).

RedisDataSource - for Redis data sources, the service object is a RedisClient (the configuration of the client is computed from the location, metadata and consumer configuration).

MongoDataSource - for Mongo data sources, the service object is a MongoClient (the configuration of the client is computed from the location, metadata and consumer configuration).

+ `HttpEndpoint` - 为REST API服务提供的类型，服务对象的类型是一个配置好了host和port的`HttpClient`（其location表现为一个url）
+ `EventBusService` - 服务代理，服务对象是一个代理，它的类型是所代理的接口（其location表现为一个event bus的address地址）
+ `MessageSource` - 消息源服务，服务对象的类型是一个`MessageConsumer`（其location表现为一个event bus的address地址）
+ `JDBCDataSource` - JDBC服务，服务对象的类型是一个`JDBCClient`（该Client的配置参数，将从location、元数据和服务消费者传递的参数中获取）
+ `RedisDataSource` - Redis服务，服务对象的类型是一个`RedisClient`（该client的配置参数，将从location、元数据和服务消费者传递的参数中获取）
+ `MongoDataSource` - Mongo数据库服务，服务对象的类型一个`MongoClient`（该client的配置参数，将从location、元数据和服务消费者传递的参数中获取）


This section gives details about service types in general and describes how to use the default service types.

本节将详细介绍一下服务类型，以及如何使用服务发现框架已提供的几种服务类型。

Services with no type
Some records may have no type (ServiceType.UNKNOWN). It is not possible to retrieve a reference for these records, but you can build the connection details from the location and metadata of the Record.

Using these services does not fire service usage events.

###无类型的服务
某些服务记录也可以不带有类型（`ServiceType.UNKNOWN`），通过这种服务记录，是无法获取到服务引用的，但是你可以通过服务记录（`Record`）的`location`和`metadata`来创建连接的细节。

使用这种服务，将不会产生服务使用的事件。

Implementing your own service type
You can create your own service type by implementing the ServiceType SPI:

###自定义的服务类型
通过实现`ServiceType`接口，可以自定义服务类型：

(optional) Create a public interface extending ServiceType. This interface is only used to provide helper methods to ease the usage of your type such as createRecord methods, getX where X is the type of service object you retrieve and so on. Check HttpEndpoint or MessageSource for example

Create a class implementing ServiceType or the interface you created in step 1. The type has a name, and a method to create the ServiceReference for this type. The name must match the type field of the Record associated with your type.

Create a class extending io.vertx.ext.discovery.types.AbstractServiceReference. You can parameterize the class with the type of service object your are going to return. You must implement AbstractServiceReference#retrieve() that creates the service object. This method is only called once. If your service object needs cleanup, also override AbstractServiceReference#close().

Create a META-INF/services/io.vertx.servicediscovery.spi.ServiceType file that is packaged in your jar. In this file, just indicate the fully qualified name of the class created at step 2.

Creates a jar containing the service type interface (step 1), the implementation (step 2 and 3) and the service descriptor file (step 4). Put this jar in the classpath of your application. Here you go, your service type is now available.

1. （可选）创建一个继承了`ServiceType`的`public`接口，在这个接口中，仅需要提供一些工具类，来简化自定义类型的使用，比如提供`createRecord`方法，`getX`，这里的`X`指的是将返回的服务对象的类型，等等。可以查看`HttpEndpoint`、`MessageSource`等实例。
2. 创建一个实现了`ServiceType`接口或者第一步定义的接口的类，这个类必须有一个`name`方法和一个用来创建`ServiceReference`的方法，这个name方法返回的名称，要和关联到自定义类型的`Record`的`type`属性一致。
3. 创建一个继承`io.vertx.ext.discovery.types.AbstractServiceReference`的类。你可以对类进行参数化，添加上你要返回的服务对象的类型信息，你必须实现`AbstractServiceReference#retrieve()`这个方法，在这个方法中创建服务对象，这个方法只会被调用一次，如果你的服务对象需要释放资源，那另外还需要覆写 `AbstractServiceReference#close()`方法。
4. 创建`META-INF/services/io.vertx.servicediscovery.spi.ServiceType`文件，并把这个文件打包到自定义类型的jar包中，在这个文件中，需要标明第二步中所创建类的全限定名。
5. 将第一步的服务接口、第二步第三步的实现类以及第四步中的服务描述文件打包成一个jar，将这个jar放到你应用的`classpath`中，然后，这个自定义类型就可以使用了。


HTTP endpoints
A HTTP endpoint represents a REST API or a service accessible using HTTP requests. The HTTP endpoint service objects are HttpClient configured with the host, port and ssl.

Publishing a HTTP endpoint

To publish a HTTP endpoint, you need a Record. You can create the record using HttpEndpoint.createRecord.

The next snippet illustrates hot to create a Record from HttpEndpoint:

##HTTP Endpoint
一个HTTP Endpoint，就是一个REST API或可以通过HTTP请求访问的服务。HTTP终点服务对象，是一个配置了host、port和ssl的`HttpClient`对象。

###发布一个HTTP Endpoint
要发布一个HTTP Endpoint，你需要一个`Record`对象，你可以通过调用`HttpEndpoint.createRecord`创建这样一个服务记录对象。

下面的代码片段，展示了如何用`HttpEndpoint`创建一个`Record`。

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

When you run your service in a container or on the cloud, it may not know its public IP and public port, so the publication must be done by another entity having this info. Generally it’s a bridge.

当你在容器或云上部署你的服务时，可能你不能确定公开的IP地址和端口，所以，服务的发布，必须通过其他拥有这些信息的实体来进行，通常是一个桥接对象。

Consuming a HTTP endpoint

Once a HTTP endpoint is published, a consumer can retrieve it. The service object is a HttpClient with a port and host configured:

### 消费一个HTTP Endpoint服务
一旦一个HTTP Endpoint服务发布好了，服务消费者就可以获取到这个服务。服务对象是一个`HttpClient`实例，并且已经配置好了host和port参数。

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

You can also use the HttpEndpoint.getClient method to combine lookup and service retrieval in one call:

你也可以使用`HttpEndpoint.getClient`这个方法，一步就完成服务查找和服务获取。

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

In this second version, the service object is released using ServiceDiscovery.releaseServiceObject, so you don’t need to keep the service reference.

Since Vert.x 3.4.0, another client has been provided. This higher-level client, named WebClient tends to be easier to use. You can retrieve a WebClient instances using:

在第二种写法中，服务对象的释放，是通过`ServiceDiscovery.releaseServiceObject`这个方法，因此在这种情况下，你是不需要持有一个服务引用的。

从Vert.x 3.4.0开始，提供了另外一种客户端，一个更高层一些的叫做`WebClient`的客户端，更方便使用，你可以通过如下方式来获取一个`WebClient`客户端：


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

And, if you prefer the approach using the service type:

另外一种写法，使用服务类型的方式：

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

Event bus services
Event bus services are service proxies. They implement async-RPC services on top of the event bus. When retrieving a service object from an event bus service, you get a service proxy of the right type. You can access helper methods from EventBusService.

Notice that service proxies (service implementations and service interfaces) are developed in Java.

## Event bus服务
Event bus服务是一种服务代理，是基于event bus实现的一种异步RPC服务。当从一个Event bus服务中获取一个服务对象时，你实际上得到的一个某个类的服务代理，你也可以使用`EventBusService`的工具方法来获得服务代理。

注意：服务代理（服务实现和服务接口）都需要用Java语言开发。


Publishing an event bus service

To publish an event bus service, you need to create a Record:

### 发布一个event bus 服务
要发布一个event bus服务，你需要创建一个`Record`对象

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

You can also pass the service interface as class:

你也可以直接传递服务接口类

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

Consuming an event bus service

To consume an event bus service you can either retrieve the record and then get the reference, or use the EventBusService interface that combines the two operations in one call.

When using the reference, you would do something like:

### 消费event bus 服务
要消费event bus服务，你可以通过获取到服务记录，然后获取服务引用的方式，也可以直接使用`EventBusService`接口，将两步合并成一次方法调用。

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

With the EventBusService class, you can get the proxy as follows:

当使用`EventBusService`类时，你可以通过如下方式获得代理对象：

```java
EventBusService.getProxy(discovery, MyService.class, ar -> {
if (ar.succeeded()) {
MyService service = ar.result();

// Dont' forget to release the service
ServiceDiscovery.releaseServiceObject(discovery, service);
}
});

```

Message source
A message source is a component sending messages on the event bus on a specific address. Message source clients are MessageConsumer.

The location or a message source service is the event bus address on which messages are sent.

##消息源服务
消息源服务，就是通过event bus发送消息到某个地址的组件，消息源服务的client是`MessageConsumer`。

消息源服务的`location`是消息所发送的event bus 地址。

Publishing a message source

As for the other service types, publishing a message source is a 2-step process:

create a record, using MessageSource

publish the record

### 发布一个消息源服务
和其他服务类型一样，发布一个消息源服务包含两个步骤：
1. 使用`MessageSource`创建一条服务记录
2. 发布这条记录

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

In the second record, the type of payload is also indicated. This information is optional.

In java, you can use Class parameters:

在第二个record创建时，同时指明了消息体的类型，这不是必须的。

如果使用Java，你可以使用`Class`参数

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

Consuming a message source

On the consumer side, you can retrieve the record and the reference, or use the MessageSource class to retrieve the service is one call.

With the first approach, the code is the following:

###消费消息源服务

在服务消费端，你可以获取服务记录和服务引用，也可以使用`MessageSource`类提供的方法直接获取。

第一种方式，代码示例如下：

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

When, using MessageSource, it becomes:
如果使用`MessageSource`，代码如下：

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

JDBC Data source
Data sources represents databases or data stores. JDBC data sources are a specialization for databases accessible using a JDBC driver. The client of a JDBC data source service is a JDBCClient.

## JDBC服务
数据源指的是数据库或数据存储。JDBC数据源通过JDBC驱动访问数据库，JDBC数据源服务对象是一个`JDBCClient`

Publishing a JDBC service

As for the other service types, publishing a JDBC data source is a 2-step process:

create a record, using JDBCDataSource

publish the record

###发布JDBC服务
和其他服务类型一样，发布JDBC数据源共两个步骤
1. 用`JDBCDataSource`创建一个服务记录
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

As JDBC data sources can represent a high variety of databases, and their access is often different, the record is rather unstructured. The location is a simple JSON object that should provide the fields to access the data source (JDBC url, username…​). The set of fields may depend on the database but also on the connection pool used in front.

JDBC数据源，可以代理了各种类型的数据库，而这些数据库的访问方式经常是很不一样，服务记录很难有统一结构。location信息有一个简单的JSON对象组成，对象中带有访问数据源的各种属性（JDBC url、username...），这些属性依赖于数据库，同时也依赖于所使用的连接池。

Consuming a JDBC service

As stated in the previous section, how to access a data source depends on the data source itself. To build the JDBCClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

###消费一个JDBC服务
如前所述，如何访问数据源，有赖于数据源本身。要创建一个`JDBCClient`，你需要同时提供：服务记录location信息、元数据以及服务消费者提供的一个json对象。

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
    
You can also use the JDBCClient class to the lookup and retrieval in one call:

你也可以使用`JDBCClient`类的方法，来查询和获取服务对象

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
    
Redis Data source
Redis data sources are a specialization for Redis persistence databases. The client of a Redis data source service is a RedisClient.

## Redis数据源
Redis数据源服务，是专门为Redis提供的服务类型，Redis数据源服务的服务对象是一个`RedisClient`。

Publishing a Redis service

Publishing a Redis data source is a 2-step process:

create a record, using RedisDataSource

publish the record

### 发布一个Redis服务
发布一个Redis服务共两个步骤：
1. 使用`RedisDataSource`创建一条服务记录
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

The location is a simple JSON object that should provide the fields to access the Redis data source (url, port…​).

这里location是一个JSON对象，需要提供访问Redis数据源的属性（url、port...）

Consuming a Redis service

As stated in the previous section, how to access a data source depends on the data source itself. To build the RedisClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

###消费Redis服务
如前所述，如何访问数据源有赖于数据源本身，要创建一个`RedisClient`，你需要同时提供：服务记录的location信息、元数据以及由消费者提供的一个json对象。

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
  
You can also use the RedisDataSource class to the lookup and retrieval in one call:

你也可以使用`RedisDataSource`这个类，来查询和获取服务对象。

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
  });
  
```

Mongo Data source
Mongo data sources are a specialization for MongoDB databases. The client of a Mongo data source service is a MongoClient.

Publishing a Mongo service

Publishing a Mongo data source is a 2-step process:

create a record, using MongoDataSource

publish the record

##Mongo数据源服务
Mongo数据源是专门为MongoDb数据库提供的一种服务类型，Mongo数据源服务的服务对象是一个`MongoClient`。

###发布Mongo服务
发布一个Mongo数据源服务共两步：
1. 使用`MongoDataSource`创建一条服务记录
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

The location is a simple JSON object that should provide the fields to access the Redis data source (url, port…​).

location是一个JSON对象，包含了访问Mongo数据源的所有属性（url、port....）

Consuming a Mongo service

As stated in the previous section, how to access a data source depends on the data source itself. To build the MongoClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

###消费Mongo服务
如前所述，如何访问数据源，有赖于数据源本身，要创建一个`MongoClient`，你需要同时提供：服务记录location信息、元数据以及服务消费者提供的一个json对象。

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

You can also use the MongoDataSource class to the lookup and retrieval in one call:

也可以调用`MongoDataSource`类的方法，来完成服务对象的查找和获取

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
  
Listening for service arrivals and departures
Every time a provider is published or removed, an event is published on the vertx.discovery.announce address. This address is configurable from the ServiceDiscoveryOptions.

The received record has a status field indicating the new state of the record:

UP : the service is available, you can start using it

DOWN : the service is not available anymore, you should not use it anymore

OUT_OF_SERVICE : the service is not running, you should not use it anymore, but it may come back later.

Listening for service usage
Every time a service reference is retrieved (bind) or released (release), an event is published on the vertx .discovery.usage address. This address is configurable from the ServiceDiscoveryOptions.

It lets you listen for service usage and map the service bindings.

The received message is a JsonObject containing:

the record in the record field

the type of event in the type field. It’s either bind or release

the id of the service discovery (either its name or the node id) in the id field

This id is configurable from the ServiceDiscoveryOptions. By default it’s "localhost" on single node configuration and the id of the node in clustered mode.

You can disable the service usage support by setting the usage address to null with setUsageAddress.

Service discovery bridges
Bridges let you import and export services from / to other discovery mechanism such as Docker, Kubernates, Consul…​ Each bridge decides how the services are imported and exported. It does not have to be bi-directional.

You can provide your own bridge by implementing the ServiceImporter interface and register it using registerServiceImporter.

The second parameter can provide an optional configuration for the bridge.

When the bridge is registered the

{@link io.vertx.servicediscovery.spi.ServiceImporter#start)} method is called. It lets you configure the bridge. When the bridge is configured, ready and has imported / exported the initial services, it must complete the given Future. If the bridge starts method is blocking, it must use an executeBlocking construct, and complete the given future object.

When the service discovery is stopped, the bridge is stopped. The close method is called that provides the opportunity to cleanup resources, removed imported / exported services…​ This method must complete the given Future to notify the caller of the completion.

Notice than in a cluster, only one member needs to register the bridge as the records are accessible by all members.

Additional bridges
In addition of the bridges supported by this library, Vert.x Service Discovery provides additional bridges you can use in your application.

Consul bridge
This discovery bridge imports services from Consul into the Vert.x service discovery.

The bridge connects to a Consul agent (server) and periodically scan for services:

new services are imported

services in maintenance mode or that has been removed from consul are removed

This bridge uses the HTTP API for Consul. It does not export to Consul and does not support service modification.

The service type is deduced from tags. If a tag matches a known service type, this service type will be used. If not, the service is imported as unknown. Only http-endpoint is supported for now.

Using the bridge

To use this Vert.x discovery bridge, add the following dependency to the dependencies section of your build descriptor:

Maven (in your pom.xml):

<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-consul</artifactId>
  <version>3.4.1</version>
</dependency>
Gradle (in your build.gradle file):

compile 'io.vertx:vertx-service-discovery-bridge-consul:3.4.1'
Then, when creating the service discovery registers this bridge as follows:

ServiceDiscovery.create(vertx)
    .registerServiceImporter(new ConsulServiceImporter(),
        new JsonObject()
            .put("host", "localhost")
            .put("port", 8500)
            .put("scan-period", 2000));
You can configure the:

agent host using the host property, it defaults to localhost

agent port using the port property, it defaults to 8500

scan period using the scan-period property. The time is set in ms, and is 2000 ms by default

Kubernetes bridge
This discovery bridge imports services from Kubernetes (or Openshift v3) into the Vert.x service discovery.

Kubernetes services are mapped to Record. This bridge only supports the importation of services from kubernetes in vert.x (and not the opposite).

Record are created from Kubernetes Service. The service type is deduced from the service.type label. If not set, the service is imported as unknown. Only http-endpoint are supported for now.

Using the bridge

To use this Vert.x discovery bridge, add the following dependency to the dependencies section of your build descriptor:

Maven (in your pom.xml):

<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-kubernetes</artifactId>
  <version>3.4.1</version>
</dependency>
Gradle (in your build.gradle file):

compile 'io.vertx:vertx-service-discovery-bridge-kubernetes:3.4.1'
Configuring the bridge

The bridge is configured using:

the oauth token (using the content of /var/run/secrets/kubernetes.io/serviceaccount/token by default)

the namespace in which the service are searched (defaults to default).

Be aware that the application must have access to Kubernetes and must be able to read the chosen namespace.

The Service to Record mapping

The record is created as follows:

the service type is deduced from the service.type label. If this label is not set the service type is set to unknown

the record’s name is the service’s name

the labels of the service are mapped to metadata

in addition are added: kubernetes.uuid, kubernetes.namespace, kubernetes.name

the location is deduced from the first port of the service

For HTTP endpoints, the ssl (https) attribute is set to true if the service has the ssl label set to true.

Dynamics

The bridge imports all services on start and removes them on stop. In between it watches the Kubernetes services and add the new ones and removes the deleted ones.

Unresolved directive in index.adoc - include::zookeeper-bridge.adoc[]

Docker Links bridge
This discovery bridge imports services from Docker Links into the Vert.x service discovery.

When you link a Docker container to another Docker container, Docker injects a set of environment variables. This bridge analyzes these environment variables and imports service record for each link. The service type is deduced from the service.type label. If not set, the service is imported as unknown. Only http-endpoint are supported for now.

As the links are created when the container starts, the imported records are created when the bridge starts and do not change afterwards.

Using the bridge

To use this Vert.x discovery bridge, add the following dependency to the dependencies section of your build descriptor:

Maven (in your pom.xml):

<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-bridge-docker-links</artifactId>
  <version>3.4.1</version>
</dependency>
Gradle (in your build.gradle file):

compile 'io.vertx:vertx-service-discovery-bridge-docker-links:3.4.1'
Then, when creating the service discovery, registers this bridge as follows:

ServiceDiscovery.create(vertx)
    .registerServiceImporter(new DockerLinksServiceImporter(), new JsonObject());
The bridge does not need any further configuration.

Additional backends
In addition of the backend supported by this library, Vert.x Service Discovery provides additional backends you can use in your application.

Redis backend
The service discovery has a plug-able backend using the ServiceDiscoveryBackend SPI.

This is an implementation of the SPI based on Redis.

Using the Redis backend

To use the Redis backend, add the following dependency to the dependencies section of your build descriptor:

Maven (in your pom.xml):

<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-discovery-backend-redis</artifactId>
  <version>3.4.1</version>
</dependency>
Gradle (in your build.gradle file):

compile 'io.vertx:vertx-service-discovery-backend-redis:3.4.1'
Be aware that you can have only a single implementation of the SPI in your classpath. If none, the default backend is used.

Configuration

The backend is based on the vertx-redis-client. The configuration is the client configuration as well as key indicating in which key on Redis the records are stored.

Here is an example:

ServiceDiscovery.create(vertx, new ServiceDiscoveryOptions()
    .setBackendConfiguration(
        new JsonObject()
            .put("host", "127.0.0.1")
            .put("key", "records")
    ));
Last updated 2017-03-15 15:54:14 CET