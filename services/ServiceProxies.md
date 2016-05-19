# Vert.x Service Proxies

> 代理允许event bus 服务像本地服务一样被调用。

- [Source][1]
- [Examples][2]

## 使用手册
> 当编写一个vertx应用时，你可能想将某个功能应用在某处隔离开来，并对其他服务提供服务。

> 这就是服务代理的目的。它让你在event bus上暴露一个服务，并其他vertx组建使用，有点类似于event bus发布订阅中的address。

> 服务是通过java接口来描述的，java接口包含的方法遵循异步模式。在底层，你向event bus发送消息，调用服务并获取响应。为了使其更容易使用，他
可以产生一个代理，你可以直接调用这个代理（使用服务接口中的API）。


### 使用vertx服务代理
使用vertx服务代理需要添加如下依赖：
- Maven(在你的`pom.xml`文件中)
```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-proxy</artifactId>
  <version>3.2.1</version>
</dependency>
```
- Gradle(在你的`build.gradle`文件中)
```
compile 'io.vertx:vertx-service-proxy:3.2.1'
```
注意，服务代理机制依赖于代码生成，所以service接口修改后需重新编译源码。
在不同的编程语言中使用代理，你需要添加此语言的相关依赖，如groovy需添加`vertx-lang-groovy`

### 服务代理介绍
让我们先看看service proxies并了解为什么有用。假设有一个数据库服务暴露在event bus上，你可能会这样做：
```java
JsonObject message = new JsonObject();
message.put("collection", "mycollection")
    .put("document", new JsonObject().put("name", "tim"));
DeliveryOptions options = new DeliveryOptions().addHeader("action", "save");
vertx.eventBus().send("database-service-address", message, options, res2 -> {
  if (res2.succeeded()) {
    // done
  } else {
    // failure
  }
});
```
用有一定量的模版代码创建一个服务来监听event bus中的消息，然后将其路由到合适的方法并将结果返回到event bus。
如果使用服务代理，你可以避免写许多同样的模版代码而将精力集中在你的服务上。
当你写你的服务代码时，只需在java接口上用`@ProxyGen`对其进行注解，例如：
```java
@ProxyGen
public interface SomeDatabaseService {

  // A couple of factory methods to create an instance and a proxy
  static SomeDatabaseService create(Vertx vertx) {
    return new SomeDatabaseServiceImpl(vertx);
  }

  static SomeDatabaseService createProxy(Vertx vertx,
    String address) {
    return new SomeDatabaseServiceVertxEBProxy(vertx, address);
  }

 // Actual service operations here...
 void save(String collection, JsonObject document,
   Handler<AsyncResult<Void>> resultHandler);
}
```
有了这个接口，Vertx将产生所有的访问你的服务的模版代码，并且它将为你的服务产生**client side proxy**，所以你的客户端使用一个相当符合语言习惯的API,而不是手动的制作event bus消息发送。不管你的实际服务在哪个event bus上(甚至不同机器上)，client side proxy照样能工作。
也就是说，你可以通过以下方式与你的服务进行交互：
```java
SomeDatabaseService service = SomeDatabaseService.createProxy(vertx,
    "database-service-address");

// Save some data in the database - this time using the proxy
service.save("mycollection", new JsonObject().put("name", "tim"), res2 -> {
  if (res2.succeeded()) {
    // done
  }
});
```

你也可以在vertx支持的任何语言中联合`@ProxyGen`和语言API代码生成（`@VertxGen`）创建服务，这意味着你通过风格一致的其他语言API与用java编写的一个服务进行交互，不管这个服务在本地还是在event bus的别处。最后别忘了添加相关依赖。
```java
@ProxyGen // Generate service proxies
@VertxGen // Generate the clients
public interface SomeDatabaseService {
  // ...
}
```
### 异步接口
想要被service-proxy generation使用，服务接口必须遵循一些规则，首先要遵循异步模式。返回结果的方法需声明`Handler<AsyncResult<ResultType>>`。`ResultType`可以是另一个代理。
例如：
```java
@ProxyGen
public interface SomeDatabaseService {

 // A couple of factory methods to create an instance and a proxy

 static SomeDatabaseService create(Vertx vertx) {
   return new SomeDatabaseServiceImpl(vertx);
 }

 static SomeDatabaseService createProxy(Vertx vertx, String address) {
   return new SomeDatabaseServiceVertxEBProxy(vertx, address);
 }

 // A method notifying the completion without a result (void)
 void save(String collection, JsonObject document,
   Handler<AsyncResult<Void>> result);

 // A method providing a result (a json object)
 void findOne(String collection, JsonObject query,
   Handler<AsyncResult<JsonObject>> result);

 // Create a connection
 void createConnection(String shoeSize,
   Handler<AsyncResult<MyDatabaseConnection>> resultHandler);

}
```
with:
```java
@ProxyGen
@VertxGen
public interface MyDatabaseConnection {

 void insert(JsonObject someData);

 void commit(Handler<AsyncResult<Void>> resultHandler);

 @ProxyClose
 void close();
}
```
你可以声明一个特殊方法用注解`@ProxyClose`注销此代理.当这个方法被调用时，此代理实例被清除。
更多服务接口的限制在下面详解。

### Code generation
服务被`@ProxyGen`注解后，会触发服务帮助类的产生。

- 服务代理：一个编译时产生的代理，用`EventBus`通过消息与服务交互。
- 服务处理器： 一个编译时产生的EventBus处理器，用于响应由代理发送的事件。

产生的代理和处理器的命名是在类名的后面加相关的字段，例如，如果一个服务名为`MyService`，则处理器命名为：`MyServiceProxyHandler`,代理命名为：`MyServiceEBProxy`。

codegen 注解处理器是在编译时产生的这些类。这是java编译器一个功能，所以无需额外的步骤，只需正确的配置好编译器。
这是针对Maven的一个配置例子：
```
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <annotationProcessors>
      <annotationProcessor>io.vertx.codegen.CodeGenProcessor</annotationProcessor>
    </annotationProcessors>
    <compilerArgs>
      <arg>-AoutputDirectory=${project.basedir}/src/main</arg>
    </compilerArgs>
  </configuration>
</plugin>
```
这个功能被用在Gradle或甚至IDE已经提供了这样的注解处理器。

### 暴露你的服务
如果你有了服务接口，编译了源码并产生了相关代理等，你就需要写一些代码将你的服务注册到event bus上。

```java
SomeDatabaseService service = new SomeDatabaseServiceImpl();
// Register the handler
ProxyHelper.registerService(SomeDatabaseService.class, vertx, service,
    "database-service-address");
```
这可以在verticle中完成，或在你的任何代码中。
一旦注册，这个服务就可用了。如果你的应用运行在集群上，则集群中任务一台主机都可访问。
如果想注销这个服务，可使用[ProxyHelper.unregisterService][3]方法。
```java
SomeDatabaseService service = new SomeDatabaseServiceImpl();
// Register the handler
MessageConsumer<JsonObject> consumer = ProxyHelper.registerService(SomeDatabaseService.class, vertx, service,
    "database-service-address");

// ....

// Unregister your service.
ProxyHelper.unregisterService(consumer);
```
### 代理创建
当你的服务部署后，你可能想去访问它。这时，你需要创建一个代理，而代理的创建可以使用`ProxyHelper `类：
```java
SomeDatabaseService service = ProxyHelper.createProxy(SomeDatabaseService.class,
    vertx,
    "database-service-address");
// or with delivery options:
SomeDatabaseService service2 = ProxyHelper.createProxy(SomeDatabaseService.class,
    vertx,
    "database-service-address", options);
```
这第二个方法获取一个`DeliveryOptions`实例，在这里可以配置消息传递的相关参数（如timeout）
你也可以使用产生的代理类。这个代理类名是服务接口类加`VertxEBProxy`。例如，你的服务接口名为：`SomeDatabaseService`,则代理类名为：`SomeDatabaseServiceVertxEBProxy`。
一般情况下，服务接口包含了一个`createProxy`的静态方法用于创建代理。但这不是必须的：
```java
@ProxyGen
public interface SomeDatabaseService {

 // Method to create the proxy.
 static SomeDatabaseService createProxy(Vertx vertx, String address) {
   return new SomeDatabaseServiceVertxEBProxy(vertx, address);
 }

 // ...
}
```
### 服务接口的约束
在服务方法中用到的类型和返回值是有限制的，这样容易转化为event bus消息，也能被异步的使用。有：
**返回类型**
必须是一下其中一个：

- void
- `@Fluent`和返回这个服务的应用(`this`):

```java
@Fluent
SomeDatabaseService doSomething();
```
这是因为方法不能阻塞而且如果服务是远程的，不可能立即返回结果而不阻塞。

**参数类型**
`JSON ` = `JsonObject | JsonArray`
`PRIMITIVE ` = 任意原生类型或包装的原生类型。
参数可以是：

- `JSON`
- `PRIMITIVE`
- `List<JSON>`
- `List<PRIMITIVE>`
- `Set<JSON>`
- `Set<PRIMITIVE>`
- `Map<String, JSON>`
- `Map<String, PRIMITIVE>`
- 任何枚举类型
- 被`@DataObject`注解的类

如果一个异步结果要求提供最后一个参数类型`Handler<AsyncResult<R>>`。
`R`可以是：

- `JSON`
- `PRIMITIVE`
- `List<JSON>`
- `List<PRIMITIVE>`
- `Set<JSON>`
- `Set<PRIMITIVE>`
- 任何枚举类型
- 被`@DataObject`注解的类
- 另一个代理

**重载的方法**
服务方法中不能有被重载。

### 调用服务的约定（没有代理）
服务代理假定event bus中的消息遵循一定的格式，因此能被用于调用服务。
当然，如果你不愿意，你也可以不用客户代理来访问远程服务。与服务交互被广泛接受的方式是直接在event bus中发送消息。
为了使服务被交互的方式一致，必须遵循一定的消息格式。
格式非常简单：

- 必须有个header叫`action`，作为这个行为或操作的名称。
- 消息体应该是一个`JsonObject`,里面不惜包含此操作中所需要的所有操作。

举个例子：
```
Headers:
    "action": "save"
Body:
    {
        "collection", "mycollection",
        "document", {
            "name": "tim"
        }
    }
```
不论有没有用到服务代理，都应该用上面这种方式创建服务，这样允许服务被交互式保持一致性。
在上面的例子中服务代理使用‘action’值应映射到服务接口中的动作行为的方法名，在消息体中的每个`[key, value]`映射到行为方法中的`[arg_name, arg_value]`。
对于返回值，服务需使用`message.reply(…​)`方法去回发一个返回值。这个值可以是任务event bus支持的类型。发送失败信号可以使用`message.fail(…​)`。
如果使用服务代理产生代码会自动为你处理这些。

[1]: https://github.com/vert-x3/vertx-service-proxy
[2]: https://github.com/vert-x3/vertx-examples/tree/master/service-proxy-examples
[3]: http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ProxyHelper.html#unregisterService-io.vertx.core.eventbus.MessageConsumer-
