# Vert.x Service Discovery

* 引用：[http://vertx.io/docs/vertx-service-discovery/java/](http://vertx.io/docs/vertx-service-discovery/java/)

<hr>

This component provides an infrastructure to publish and discover various resources, such as service proxies, HTTP endpoints, data sources…​

Vert.x Service Discovery，提供了一个基础的软件框架，用来发布和发现各种类型的资源，比如服务代理、HTTP应用、datasources等等。

These resources are called services. A service is a discoverable functionality. It can be qualified by its type, metadata, and location. So a service can be a database, a service proxy, a HTTP endpoint and any other resource you can imagine as soon as you can describe it, discover it and interact with it. It does not have to be a vert.x entity, but can be anything. Each service is described by a Record.

这些资源都可以称为服务，服务就是一个可以被发现和访问的功能，可以通过它的类型、元数据和位置来进行描述。所以，服务可以是一个数据库、一个服务代理、一个HTTP应用，以及任何你能想到的可描述、可发现、可交互的资源。它不一定是vert.x实体，它可以是任何组件。在Vert.x Service Discovery中，每一个服务都被叫做一个[Record](http://vertx.io/docs/apidocs/io/vertx/servicediscovery/Record.html) 。

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

从上面可以看出，服务提供者和服务消费者，通过record共享关键的信息。

Providers and consumers must create their own ServiceDiscovery instance. These instances are collaborating in the background (distributed structure) to keep the set of services in sync.

服务提供者和消费者，必须创建他们自己的ServiceDiscovery实例，这些实例通过幕后（分布式结构）的协作来保持服务的同步。

The service discovery supports bridges to import and export services from / to other discovery technologies.

Service Discovery支持桥接的方式，来从其他服务发现技术中导入和导出服务。

Using the service discovery
To use the Vert.x service discovery, add the following dependency to the dependencies section of your build descriptor:

## 使用Service Discovery
要使用Vert.x的Service Discovery，请将下列依赖加入到依赖配置中文件。


Maven (in your pom.xml):

<dependency>
<groupId>io.vertx</groupId>
<artifactId>vertx-service-discovery</artifactId>
<version>3.4.1</version>
</dependency>
Gradle (in your build.gradle file):

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

Service Consumer
Service consumers search for services in the service discovery. Each lookup retrieves 0..n Record. From these records, a consumer can retrieve a ServiceReference, representing the binding between the consumer and the provider. This reference allows the consumer to retrieve the service object (to use the service), and release the service.

It is important to release service references to cleanup the objects and update the service usages.

Service object
The service object is the object that gives access to a service. It can come in various forms, such as a proxy, a client, and may even be non-existent for some service types. The nature of the service object depends on the service type.

Notice that because of the polyglot nature of Vert.x, the service object can differ if you retrieve it from Java, Groovy or another language.

Service types
Services are just resources, and there are a lot of different kinds of services. They can be functional services, databases, REST APIs, and so on. The Vert.x service discovery has the concept of service types to handle this heterogeneity. Each type defines:

how the service is located (URI, event bus address, IP / DNS…​) - location

the nature of the service object (service proxy, HTTP client, message consumer…​) - client

Some service types are implemented and provided by the service discovery component, but you can add your own.

Service events
Every time a service provider is published or withdrawn, an event is fired on the event bus. This event contains the record that has been modified.

In addition, in order to track who is using who, every time a reference is retrieved with getReference or released with release, events are emitted on the event bus to track the service usages.

More details on these events below.

Backend
The service discovery uses a Vert.x distributed data structure to store the records. So, all members of the cluster have access to all the records. This is the default backend implementation. You can implement your own by implementing the ServiceDiscoveryBackend SPI. For instance, we provide an implementation based on Redis.

Notice that the discovery does not require Vert.x clustering. In single-node mode, the structure is local. It can be populated with `ServiceImporter`s.

Creating a service discovery instance
Publishers and consumers must create their own ServiceDiscovery instance to use the discovery infrastructure:

ServiceDiscovery discovery = ServiceDiscovery.create(vertx);

// Customize the configuration
discovery = ServiceDiscovery.create(vertx,
    new ServiceDiscoveryOptions()
        .setAnnounceAddress("service-announce")
        .setName("my-name"));

// Do something...

discovery.close();
By default, the announce address (the event bus address on which service events are sent is: vertx.discovery .announce. You can also configure a name used for the service usage (see section about service usage).

When you don’t need the service discovery object anymore, don’t forget to close it. It closes the different discovery importers and exporters you have configured and releases the service references.

You should avoid sharing the service discovery instance, so service usage would represent the right "usages".

Publishing services
Once you have a service discovery instance, you can publish services. The process is the following:

create a record for a specific service provider

publish this record

keep the published record that is used to un-publish a service or modify it.

To create records, you can either use the Record class, or use convenient methods from the service types.

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
It is important to keep a reference on the returned records, as this record has been extended by a registration id.

Withdrawing services
To withdraw (un-publish) a record, use:

discovery.unpublish(record.getRegistration(), ar -> {
  if (ar.succeeded()) {
    // Ok
  } else {
    // cannot un-publish the service, may have already been removed, or the record is not published
  }
});
Looking for services
This section explains the low-level process to retrieve services, each service type provide convenient method to aggregates the different steps.

On the consumer side, the first thing to do is to lookup for records. You can search for a single record or all the matching ones. In the first case, the first matching record is returned.

Consumer can pass a filter to select the service. There are two ways to describe the filter:

A function taking a Record as parameter and returning a boolean (it’s a predicate)

This filter is a JSON object. Each entry of the given filter is checked against the record. All entries must exactly match the record. The entry can use the special * value to denote a requirement on the key, but not on the value.

Let’s see an example of a JSON filter:

{ "name" = "a" } => matches records with name set to "a"
{ "color" = "*" } => matches records with "color" set
{ "color" = "red" } => only matches records with "color" set to "red"
{ "color" = "red", "name" = "a"} => only matches records with name set to "a", and color set to "red"
If the JSON filter is not set (null or empty), it accepts all records. When using functions, to accept all records, you must return true regardless the record.

Here are some examples:

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
You can retrieve a single record or all matching records with getRecords. By default, record lookup does include only records with a status set to UP. This can be overridden:

when using JSON filter, just set status to the value you want (or * to accept all status)

when using function, set the includeOutOfService parameter to true in getRecords .

Retrieving a service reference
Once you have chosen the Record, you can retrieve a ServiceReference and then the service object:

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
Don’t forget to release the reference once done.

The service reference represents a binding with the service provider.

When retrieving a service reference you can pass a JsonObject used to configure the service object. It can contain various data about the service object. Some service types do not need additional configuration, some require configuration (as data sources):

ServiceReference reference = discovery.getReferenceWithConfiguration(record, conf);

// Then, gets the service object, the returned type depends on the service type:
// For http endpoint:
JDBCClient client = reference.getAs(JDBCClient.class);

// Do something with the client...

// When done with the service
reference.release();
In the previous examples, the code uses getAs. The parameter is the type of object you expect to get. If you are using Java, you can use get. However in the other language you must pass the expected type.

Types of services
A said above, the service discovery has the service type concept to manage the heterogeneity of the different services.

These types are provided by default:

HttpEndpoint - for REST API’s, the service object is a HttpClient configured on the host and port (the location is the url).

EventBusService - for service proxies, the service object is a proxy. Its type is the proxies interface (the location is the address).

MessageSource - for message sources (publisher), the service object is a MessageConsumer (the location is the address).

JDBCDataSource - for JDBC data sources, the service object is a JDBCClient (the configuration of the client is computed from the location, metadata and consumer configuration).

RedisDataSource - for Redis data sources, the service object is a RedisClient (the configuration of the client is computed from the location, metadata and consumer configuration).

MongoDataSource - for Mongo data sources, the service object is a MongoClient (the configuration of the client is computed from the location, metadata and consumer configuration).

This section gives details about service types in general and describes how to use the default service types.

Services with no type
Some records may have no type (ServiceType.UNKNOWN). It is not possible to retrieve a reference for these records, but you can build the connection details from the location and metadata of the Record.

Using these services does not fire service usage events.

Implementing your own service type
You can create your own service type by implementing the ServiceType SPI:

(optional) Create a public interface extending ServiceType. This interface is only used to provide helper methods to ease the usage of your type such as createRecord methods, getX where X is the type of service object you retrieve and so on. Check HttpEndpoint or MessageSource for example

Create a class implementing ServiceType or the interface you created in step 1. The type has a name, and a method to create the ServiceReference for this type. The name must match the type field of the Record associated with your type.

Create a class extending io.vertx.ext.discovery.types.AbstractServiceReference. You can parameterize the class with the type of service object your are going to return. You must implement AbstractServiceReference#retrieve() that creates the service object. This method is only called once. If your service object needs cleanup, also override AbstractServiceReference#close().

Create a META-INF/services/io.vertx.servicediscovery.spi.ServiceType file that is packaged in your jar. In this file, just indicate the fully qualified name of the class created at step 2.

Creates a jar containing the service type interface (step 1), the implementation (step 2 and 3) and the service descriptor file (step 4). Put this jar in the classpath of your application. Here you go, your service type is now available.

HTTP endpoints
A HTTP endpoint represents a REST API or a service accessible using HTTP requests. The HTTP endpoint service objects are HttpClient configured with the host, port and ssl.

Publishing a HTTP endpoint

To publish a HTTP endpoint, you need a Record. You can create the record using HttpEndpoint.createRecord.

The next snippet illustrates hot to create a Record from HttpEndpoint:

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
When you run your service in a container or on the cloud, it may not know its public IP and public port, so the publication must be done by another entity having this info. Generally it’s a bridge.

Consuming a HTTP endpoint

Once a HTTP endpoint is published, a consumer can retrieve it. The service object is a HttpClient with a port and host configured:

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
You can also use the HttpEndpoint.getClient method to combine lookup and service retrieval in one call:

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
In this second version, the service object is released using ServiceDiscovery.releaseServiceObject, so you don’t need to keep the service reference.

Since Vert.x 3.4.0, another client has been provided. This higher-level client, named WebClient tends to be easier to use. You can retrieve a WebClient instances using:

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
And, if you prefer the approach using the service type:

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
Event bus services
Event bus services are service proxies. They implement async-RPC services on top of the event bus. When retrieving a service object from an event bus service, you get a service proxy of the right type. You can access helper methods from EventBusService.

Notice that service proxies (service implementations and service interfaces) are developed in Java.

Publishing an event bus service

To publish an event bus service, you need to create a Record:

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
You can also pass the service interface as class:

Record record = EventBusService.createRecord(
"some-eventbus-service", // The service name
"address", // the service address,
MyService.class // the service interface
);

discovery.publish(record, ar -> {
// ...
});
Consuming an event bus service

To consume an event bus service you can either retrieve the record and then get the reference, or use the EventBusService interface that combines the two operations in one call.

When using the reference, you would do something like:

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
With the EventBusService class, you can get the proxy as follows:

EventBusService.getProxy(discovery, MyService.class, ar -> {
if (ar.succeeded()) {
MyService service = ar.result();

// Dont' forget to release the service
ServiceDiscovery.releaseServiceObject(discovery, service);
}
});
Message source
A message source is a component sending messages on the event bus on a specific address. Message source clients are MessageConsumer.

The location or a message source service is the event bus address on which messages are sent.

Publishing a message source

As for the other service types, publishing a message source is a 2-step process:

create a record, using MessageSource

publish the record

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
In the second record, the type of payload is also indicated. This information is optional.

In java, you can use Class parameters:

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
Consuming a message source

On the consumer side, you can retrieve the record and the reference, or use the MessageSource class to retrieve the service is one call.

With the first approach, the code is the following:

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
When, using MessageSource, it becomes:

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
JDBC Data source
Data sources represents databases or data stores. JDBC data sources are a specialization for databases accessible using a JDBC driver. The client of a JDBC data source service is a JDBCClient.

Publishing a JDBC service

As for the other service types, publishing a JDBC data source is a 2-step process:

create a record, using JDBCDataSource

publish the record

Record record = JDBCDataSource.createRecord(
    "some-data-source-service", // The service name
    new JsonObject().put("url", "some jdbc url"), // The location
    new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
As JDBC data sources can represent a high variety of databases, and their access is often different, the record is rather unstructured. The location is a simple JSON object that should provide the fields to access the data source (JDBC url, username…​). The set of fields may depend on the database but also on the connection pool used in front.

Consuming a JDBC service

As stated in the previous section, how to access a data source depends on the data source itself. To build the JDBCClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

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
You can also use the JDBCClient class to the lookup and retrieval in one call:

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
Redis Data source
Redis data sources are a specialization for Redis persistence databases. The client of a Redis data source service is a RedisClient.

Publishing a Redis service

Publishing a Redis data source is a 2-step process:

create a record, using RedisDataSource

publish the record

Record record = RedisDataSource.createRecord(
  "some-redis-data-source-service", // The service name
  new JsonObject().put("url", "localhost"), // The location
  new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
The location is a simple JSON object that should provide the fields to access the Redis data source (url, port…​).

Consuming a Redis service

As stated in the previous section, how to access a data source depends on the data source itself. To build the RedisClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

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
You can also use the RedisDataSource class to the lookup and retrieval in one call:

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
Mongo Data source
Mongo data sources are a specialization for MongoDB databases. The client of a Mongo data source service is a MongoClient.

Publishing a Mongo service

Publishing a Mongo data source is a 2-step process:

create a record, using MongoDataSource

publish the record

Record record = MongoDataSource.createRecord(
  "some-data-source-service", // The service name
  new JsonObject().put("connection_string", "some mongo connection"), // The location
  new JsonObject().put("some-metadata", "some-value") // Some metadata
);

discovery.publish(record, ar -> {
  // ...
});
The location is a simple JSON object that should provide the fields to access the Redis data source (url, port…​).

Consuming a Mongo service

As stated in the previous section, how to access a data source depends on the data source itself. To build the MongoClient, you can merge configuration: the record location, the metadata and a json object provided by the consumer:

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
You can also use the MongoDataSource class to the lookup and retrieval in one call:

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