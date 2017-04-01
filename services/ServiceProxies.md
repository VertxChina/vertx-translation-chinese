# Vert.x Service Proxy

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

**当编写一个Vert.x应用时，你可能想将某个功能在某处隔离开来，并对应用的其它部分提供服务。**

这就是服务代理（service proxy）的目的。它允许你在Event Bus上暴露（expose）一个服务，所以，只要它们在服务发布时清楚服务的地址（address），其他任意的Vert.x组件都可以去调用它。

Vert.x用一个Java接口来描述一个 **服务**，这个接口包含的方法遵循异步模式。在底层，服务调用是通过调用端向Event Bus发送消息，被调用端收到消息调用服务并且返回结果来实现的。为了使其更容易使用，服务代理组件可以生成一个**代理类**，你可以直接调用这个代理（通过服务接口中的API）。

## 使用Vert.x 服务代理组件

要使用Vert.x Service Proxy组件，请先加入以下依赖：

- Maven(在 `pom.xml` 文件中)：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-proxy</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle(在 `build.gradle` 文件中)：

```groovy
compile 'io.vertx:vertx-service-proxy:3.4.1'
```

要**实现**服务代理（译者注：即生成服务代理类），还需要加入以下依赖：

- Maven(在 `pom.xml` 文件中)：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-codegen</artifactId>
  <version>3.4.1</version>
  <scope>provided</scope>
</dependency>
```

- Gradle(在 `build.gradle` 文件中)：

```groovy
compileOnly 'io.vertx:vertx-codegen:3.4.1'
```


注意服务代理机制依赖于代码生成，所以每次修改服务接口以后都需重新执行构建过程来重新生成代码。

如果需要生成不同语言的服务代理代码，你需要添加对应的语言支持依赖，比如 Groovy 对应 `vertx-lang-groovy`。

## 服务代理介绍

让我们先看看服务代理并了解一下为什么它们有用。假设有一个数据库服务暴露在Event Bus上，你需要做以下的事情来调用服务：

```java
JsonObject message = new JsonObject();
message.put("collection", "mycollection")
    .put("document", new JsonObject().put("name", "tim"));
DeliveryOptions options = new DeliveryOptions().addHeader("action", "save");
vertx.eventBus().send("database-service-address", message, options, res2 -> {
  if (res2.succeeded()) {
    // 调用成功
  } else {
    // 调用失败
  }
});
```
当我们用这种方式写服务模块的时候，需要写很多重复的模板代码来监听Event Bus中的消息、将其分派到合适的方法并将结果返回到Event Bus。而有了Vert.x 服务代理组件，你就不必再写这么多的模板代码了，只需要专注服务的实现即可。

你需要将服务接口抽象成一个Java接口，并且加上 `@ProxyGen` 注解，比如：

```java
@ProxyGen
public interface SomeDatabaseService {

  // 一些用于创建服务实例和服务代理实例的工厂方法
  static SomeDatabaseService create(Vertx vertx) {
    return new SomeDatabaseServiceImpl(vertx);
  }

  static SomeDatabaseService createProxy(Vertx vertx,
    String address) {
    return new SomeDatabaseServiceVertxEBProxy(vertx, address);
  }

 // 实际的服务方法
 void save(String collection, JsonObject document,
   Handler<AsyncResult<Void>> resultHandler);
}
```

有了这个接口，Vert.x会生成所有需要的用于在Event Bus上访问你的服务的模板代码，同时也会生成对应的 **调用端代理类**（client side proxy），这样你的服务调用端就可以使用一个相当符合习惯的API（译者注：即相同的服务接口）进行服务调用，而不是去手动地向Event Bus发送消息。不管你的服务实际在哪个Event Bus上（可能是在不同的机器上），调用端代理类都能正常工作。

也就是说，你可以通过以下方式进行服务调用：

```java
SomeDatabaseService service = SomeDatabaseService.createProxy(vertx,
    "database-service-address");

// 使用代理类进行服务调用 —— 向数据库中存储一些数据
service.save("mycollection", new JsonObject().put("name", "tim"), res2 -> {
  if (res2.succeeded()) {
    // 调用完毕
  }
});
```

> 译者注：Vert.x 服务代理组件提供的功能其实就是一种 **异步RPC** 的功能，其底层实现依赖于Event Bus。

你也可以将多语言API生成功能（`@VertxGen`注解）与 `@ProxyGen` 注解相结合，用于生成其它Vert.x支持的JVM语言对应的服务代理 —— 这意味着你可以只用Java编写你的服务一次，就可以在其他语言中以一种习惯的API风格进行服务调用，而不必管服务是在本地还是在Event Bus的别处。想要利用多语言代码生成功能，不要忘记添加对应支持语言的依赖。以下是使用示例：

```java
@ProxyGen // 生成服务代理
@VertxGen // 生成其他语言的代码
public interface SomeDatabaseService {
  // ...
}
```

## 异步接口

想要正确地生成服务代理类，服务接口的设计必须遵循一些规则。首先是需要遵循异步模式。如果需要返回结果，对应的方法需要包含一个 `Handler<AsyncResult<ResultType>>` 类型的参数，其中 `ResultType` 可以是另一种代理类型（所以一个代理类可以作为另一个代理类的工厂）。

例如：

```java
@ProxyGen
public interface SomeDatabaseService {

 // 一些用于创建服务实例和服务代理实例的工厂方法

 static SomeDatabaseService create(Vertx vertx) {
   return new SomeDatabaseServiceImpl(vertx);
 }

 static SomeDatabaseService createProxy(Vertx vertx, String address) {
   return new SomeDatabaseServiceVertxEBProxy(vertx, address);
 }

 // 异步方法，仅通知调用是否完成，不返回结果
 void save(String collection, JsonObject document,
   Handler<AsyncResult<Void>> result);

 // 异步方法，包含JsonObject类型的返回结果
 void findOne(String collection, JsonObject query,
   Handler<AsyncResult<JsonObject>> result);

 // 创建连接
 void createConnection(String shoeSize,
   Handler<AsyncResult<MyDatabaseConnection>> resultHandler);

}
```

以及：

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
你可以通过声明一个特殊方法，并给其加上 `@ProxyClose` 注解来注销代理。当此方法被调用时，代理实例被清除。

更多服务接口的限制会在下面详解。

## 代码生成

被 `@ProxyGen` 注解的服务接口会触发生成对应的服务辅助类：

- 服务代理类（service proxy）：一个编译时产生的代理类，用 `EventBus` 通过消息与服务交互。
- 服务处理器类（service handler）： 一个编译时产生的 `EventBus` 处理器类，用于响应由服务代理发送的事件。

产生的服务代理和处理器的命名是在类名的后面加相关的字段，例如，如果一个服务接口名为 `MyService`，则对应的处理器类命名为 `MyServiceProxyHandler` ，对应的服务代理类命名为 `MyServiceVertxEBProxy`。

同时Vert.x Codegen也提供数据对象转换器（data object converter）的生成，这使得在服务代理中处理数据实体更加容易。生成的转换器提供了一个接受 `JsonObject` 的构造函数（译者注：用于将 `JsonObject` 转换为数据实体类）以及一个 `toJson` 函数（译者注：用于将数据实体类转换为 `JsonObject`），这些函数对于在服务代理中处理数据实体来说都是必要的。

*Codegen* 注解处理器（annotation processor）会在编译期生成这些类。这是Java编译器的一个特性，所以不需要额外的步骤，只需要去配置一下对应的构建配置：

只需要在构建配置中加上 `io.vertx:vertx-service-proxy:processor` 依赖。

这是一个针对Maven的配置示例：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-proxy</artifactId>
  <version>3.4.1</version>
  <classifier>processor</classifier>
</dependency>
```

Gradle中也可以进行配置：

```groovy
compile "io.vertx:vertx-service-proxy:3.4.1:processor"
```

IDE通常会支持注解处理器。

`processor` classifier会自动通过 `META-INF/services` 插件机制向jar包中添加服务代理注解处理器的配置。

如果想要的话，你也可以通过正常的jar来使用注解处理器，但是你需要显式地声明注解处理器。比如在 Maven 中：

```xml
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <annotationProcessors>
      <annotationProcessor>io.vertx.serviceproxy.ServiceProxyProcessor</annotationProcessor>
    </annotationProcessors>
  </configuration>
</plugin>
```

## 暴露你的服务

当你写好服务接口以后，执行构建操作以生成代码。然后你需要将你的服务“注册”到Event Bus上：

```java
SomeDatabaseService service = new SomeDatabaseServiceImpl();
// 注册服务
ProxyHelper.registerService(SomeDatabaseService.class, vertx, service,
    "database-service-address");
```

这个过程既可以在 Verticle 中完成，也可以在你的代码的任何其它位置完成。

一旦注册了，这个服务就可用了。如果你的应用运行在集群上，则集群中节点都可访问。

如果想注销这个服务，可使用 [`ProxyHelper.unregisterService`](http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ProxyHelper.html#unregisterService-io.vertx.core.eventbus.MessageConsumer-) 方法：

```java
SomeDatabaseService service = new SomeDatabaseServiceImpl();
// 注册服务
MessageConsumer<JsonObject> consumer = ProxyHelper.registerService(SomeDatabaseService.class, vertx, service,
    "database-service-address");

// ....

// 注销服务
ProxyHelper.unregisterService(consumer);
```

## 代理的创建
当你的服务发布（expose）以后，你可能想要去调用它。这时，你需要创建一个服务代理，而代理的创建可以利用 `ProxyHelper` 类：

```java
SomeDatabaseService service = ProxyHelper.createProxy(SomeDatabaseService.class,
    vertx,
    "database-service-address");
// 也可以指定消息传递的配置
SomeDatabaseService service2 = ProxyHelper.createProxy(SomeDatabaseService.class,
    vertx,
    "database-service-address", options);
```
其中第二个方法会接受一个 [`DeliveryOptions`](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html) 实例，你可以在这里配置消息传递的相关参数（如 `timeout`）。

你也可以使用生成的代理类，代理类名是服务接口类名加上 `VertxEBProxy`。比如你的服务接口类名是 `SomeDatabaseService`，则代理名就是 `SomeDatabaseServiceVertxEBProxy`。

通常情况下，服务接口中会包含一个 `createProxy` 静态方法用来创建服务代理实例，但这不是必须的：

```java
@ProxyGen
public interface SomeDatabaseService {

 // 用于创建服务代理实例的方法
 static SomeDatabaseService createProxy(Vertx vertx, String address) {
   return new SomeDatabaseServiceVertxEBProxy(vertx, address);
 }

 // ...
}
```

## 错误处理

服务方法可能会通过向方法的处理器（`Handler`）传递一个失败状态的 `Future` （包含一个 [`ServiceException`](http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ServiceException.html) 实例）来返回错误。一个 [`ServiceException`](http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ServiceException.html) 包含一个整形（`int`）的错误状态码、一条消息和一个可选的 `JsonObject` 对象（用于包含额外的重要信息）。为了方便起见，我们可以使用 [`ServiceException.fail`](http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ServiceException.html#fail-int-java.lang.String-) 工厂方法来创建一个已经是失败状态并且包装着 [`ServiceException`](http://vertx.io/docs/apidocs/io/vertx/serviceproxy/ServiceException.html) 实例的 `Future`。比如：

```java
public class SomeDatabaseServiceImpl implements SomeDatabaseService {
private static final BAD_SHOE_SIZE = 42;
private static final CONNECTION_FAILED = 43;

  // 创建连接
  void createConnection(String shoeSize, Handler<AsyncResult<MyDatabaseConnection>> resultHandler) {
    if (!shoeSize.equals("9")) {
      resultHandler.handle(ServiceException.fail(BAD_SHOE_SIZE, "The shoe size must be 9!",
        new JsonObject().put("shoeSize", shoeSize));
     } else {
        doDbConnection(result -> {
          if (result.succeeded()) {
            resultHandler.handle(Future.succeededFuture(result.result()));
          } else {
            resultHandler.handle(ServiceException.fail(CONNECTION_FAILED, result.cause().getMessage()));
          }
        });
     }
  }
}
```

服务调用端（client side）可以检查它接收到的失败状态的`AsyncResult`包含的`Throwable`对象是否为`ServiceException`实例。如果是的话，继续检查内部的特定的错误状态码。调用端可以通过这些信息来将业务逻辑错误与系统错误（如服务没有被注册到Event Bus上）区分开，以便确定到底发生了哪一种业务逻辑错误。下面是一个例子：

```java
public void foo(String shoeSize, Handler<AsyncResult<JsonObject>> handler) {
  SomeDatabaseService service = SomeDatabaseService.createProxy(vertx, SERVICE_ADDRESS);
  service.createConnection("8", result -> {
    if (result.succeeded()) {
      // 正常调用
    } else {
      if (result.cause() instanceof ServiceException) {
        ServiceException exc = (ServiceException) result.cause();
        if (exc.failureCode() == SomeDatabaseServiceImpl.BAD_SHOE_SIZE) {
          handler.handle(Future.failedFuture(
            new InvalidInputError("You provided a bad shoe size: " +
              exc.getDebugInfo().getString("shoeSize"))
          ));
        } else if (exc.failureCode() == SomeDatabaseServiceImpl.CONNECTION) {
          handler.handle(Future.failedFuture(
            new ConnectionError("Failed to connect to the DB")));
        }
      } else {
        // 可能是一个系统错误(system error)，如服务代理没有对应的已注册的服务
        handler.handle(Future.failedFuture(
          new SystemError("An unexpected error occurred: + " result.cause().getMessage())
        ));
      }
    }
  }
}
```

如果需要的话，服务实现的时候也可以返回`ServiceException`的子类，只要向Event Bus注册了对应的默认`MessageCodec`就可以。比如给定下面的`ServiceException`子类：

```java
class ShoeSizeException extends ServiceException {
  public static final BAD_SHOE_SIZE_ERROR = 42;

  private final String shoeSize;

  public ShoeSizeException(String shoeSize) {
    super(BAD_SHOE_SIZE_ERROR, "In invalid shoe size was received: " + shoeSize);
    this.shoeSize = shoeSize;
  }

  public String getShoeSize() {
    return extra;
  }

  public static <T> AsyncResult<T> fail(int failureCode, String message, String shoeSize) {
    return Future.failedFuture(new MyServiceException(failureCode, message, shoeSize));
  }
}
```

只要向Event Bus注册了对应的`MessageCodec`，服务就可以直接向调用者返回自定义的异常类型：

```java
public class SomeDatabaseServiceImpl implements SomeDatabaseService {
  public SomeDataBaseServiceImpl(Vertx vertx) {
    // 在服务端（被调用端）注册MessageCodec。如果运行在单机模式下这就足够了
    // 因为服务代理会共享同一个Vertx实例
    vertx.eventBus().registerDefaultCodec(ShoeSizeException.class,
      new ShoeSizeExceptionMessageCodec());
  }

  // 创建连接
  void createConnection(String shoeSize, Handler<AsyncResult<MyDatabaseConnection>> resultHandler) {
    if (!shoeSize.equals("9")) {
      resultHandler.handle(ShoeSizeException.fail(shoeSize));
    } else {
      resultHandler.Handle(Future.succeededFuture(myDbConnection));
    }
  }
}
```

最后调用端可以检查自定义的异常类型了：

```java
public void foo(String shoeSize, Handler<AsyncResult<JsonObject>> handler) {
  // 如果运行在集群模式下，那么需要将ShoeSizeExceptionMessageCodec注册到当前节点的Event Bus下
  SomeDatabaseService service = SomeDatabaseService.createProxy(vertx, SERVICE_ADDRESS);
  service.createConnection("8", result -> {
    if (result.succeeded()) {
      // 进行方法调用
    } else {
      if (result.cause() instanceof ShoeSizeException) {
        ShoeSizeException exc = (ShoeSizeException) result.cause();
        handler.handle(Future.failedFuture(
          new InvalidInputError("You provided a bad shoe size: " + exc.getShoeSize())));
      } else {
        // 可能是一个系统错误(system error)，如服务代理没有对应的已注册的服务
        handler.handle(Future.failedFuture(
          new SystemError("An unexpected error occurred: + " result.cause().getMessage())
        ));
      }
    }
  }
}
```

注意在Vert.x 集群模式下，你需要向集群中每个节点的Event Bus注册对应的自定义异常类型的`MessageCodec`实例。

## 服务接口的约束

在服务方法中可用的参数类型和返回值类型是有限制的，这样使得转化为Event Bus消息更加容易。下面我们就来看一下：

### 方法返回类型

返回类型必须是以下其中之一：

- `void`
- 返回此服务实例的引用（`this`）并标注 `@Fluent` 注解：

```java
@Fluent
SomeDatabaseService doSomething();
```

这是因为方法不能阻塞，并且如果服务是远程的，不可能立即返回结果而不阻塞。

### 参数类型和异步返回类型

令 `JSON ` = `JsonObject | JsonArray`，`PRIMITIVE` = 任意原生类型或包装的原生类型。

参数类型可以是以下类型中任意一个：

- `JSON`
- `PRIMITIVE`
- `List<JSON>`
- `List<PRIMITIVE>`
- `Set<JSON>`
- `Set<PRIMITIVE>`
- `Map<String, JSON>`
- `Map<String, PRIMITIVE>`
- 任何枚举类型
- 任何被 `@DataObject` 注解的类

如果需要返回异步结果，可以提供一个 `Handler<AsyncResult<R>>` 类型的参数放到最后。其中类型`R`可以是：

- `JSON`
- `PRIMITIVE`
- `List<JSON>`
- `List<PRIMITIVE>`
- `Set<JSON>`
- `Set<PRIMITIVE>`
- 任何枚举类型
- 任何被 `@DataObject` 注解的类
- 另一个代理类

### 重载的方法

服务接口中不允许有重载的服务方法（即方法名相同，参数列表不同）。

## 通过Event Bus调用服务的约定（不使用服务代理的情况下）

服务代理假定Event Bus中的消息遵循一定的格式，因此能被用于服务的调用。

当然，如果不愿意的话，你也可以不用服务代理类来访问远程服务。被广泛接受的与服务交互的方式就是直接在Event Bus发送消息。

为了使服务访问的方式一致，所有的服务都必须遵循以下的消息格式。格式非常简单：

- 需要有一个名为 `action` 的 消息头(header)，作为要执行操作的名称
- 消息体（message body）应该是一个 `JsonObject` 对象，里面需要包含操作需要的所有参数。

举个例子，假如我们要去执行一个名为`save`的操作，此操作接受一个字符串类型的`collection`参数和一个`JsonObject`类型的`document`参数：

```json
Headers:
    "action": "save"
Body:
    {
        "collection": "mycollection",
        "document": {
            "name": "tim"
        }
    }
```

无论有没有用到服务代理，都应该用上面这种方式编写服务，因为这样允许服务交互时保持一致性。

在上面的例子中，`action`对应的值应该与服务接口的某个方法名称相对应，而消息体中每个 `[key, value]` 都要与服务方法中的某个 `[arg_name, arg_value]`相对应（译者注：`key`对应参数名，`value`对应参数值）。

对于返回值，服务需使用 `message.reply(…​)` 方法去向调用端发送回一个返回值 —— 这个值可以是Event Bus支持的任何类型。如果需要表示调用失败，可以使用 `message.fail(…​)` 方法。

如果你使用Vert.x 服务代理组件的话，生成的代码会自动帮你处理这些问题。

---

> [原文档](http://vertx.io/docs/vertx-service-proxy/java/#_error_handling)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-service-proxy/java/
[2]: https://github.com/vert-x3/vertx-service-proxy
[3]: https://github.com/vert-x3/vertx-examples/tree/master/service-proxy-examples
