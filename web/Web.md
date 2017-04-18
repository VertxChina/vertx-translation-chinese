# Vert.x Web - Draft

## 中英对照表

* Container：容器
* Micro-service：微服务
* Bridge：桥接
* Router：路由器
* Route：路由
* Sub-Route: 子路由
* Handler：处理器，某些特定的地方未翻译
* Blocking：阻塞式
* Context：多指代路由的上下文 routing context，需要区别于 Vert.x core 的 Context
* Application：应用
* Header：消息头
* Body：消息体
* MIME types：互联网媒体类型

## 正文

**Vert.x-Web 是一系列用于基于 Vert.x 构建 web 应用的构建模块。**

可以把它想象成一把构建现代的、可扩展的 web 应用的瑞士军刀。

Vert.x core 提供了一系列底层的功能用于操作 HTTP，对于一部分应用来是足够的。

Vert.x-Web 基于 Vert.x core，提供了一系列更丰富的功能以便更容易地开发实际的 Web 应用。

它继承了 Vert.x 2.x 里的 [Yoke](http://pmlopes.github.io/yoke/) 的特点，灵感来自于 Node.js 的框架 [Express](http://expressjs.com/) 和 Ruby 的框架 [Sinatra](http://www.sinatrarb.com/) 等等。

Vert.x-Web 的设计是强大的，非侵入式的，并且是完全可插拔的，你可以只使用你需要的部分。Vert.x-Web 不是一个容器。

你可以使用 Vetx.x-Web 来构建经典的服务端 web 应用，RESTful 应用，实时的（服务端推送）web 应用，或任何类型的你所能想到的 Web 应用。应用类型的选择取决于你，而不是 Vert.x-Web。

Vert.x-Web 非常适合编写 **RESTful Http 微服务**，但我们不强制你必须把应用实现成这样。

Vert.x-Web 的一部分关键特性有：

* 路由（基于方法(1)、路径等）
* 基于正则表达式的路径匹配
* 从路径中提取参数
* 内容协商(2)
* 处理消息体
* 消息体的长度限制
* 接收和解析 Cookie
* Multipart 表单
* Multipart 文件上传
* 子路由
* 支持本地会话和集群会话
* 支持 CORS（跨域资源共享）
* 错误页面处理器
* HTTP基本认证
* 基于重定向的认证
* 授权处理器
* 基于 JWT 的授权
* 用户/角色/权限授权
* 网页图标处理器
* 支持服务端模板渲染，包括以下开箱机用的模板引擎：
    * Handlebars
    * Jade
    * MVEL
    * Thymeleaf
    * Apache FreeMarker
    * Pebble
* 响应时间处理器
* 静态文件服务，包括缓存逻辑以及目录监听
* 支持请求超时
* 支持 SockJS
* Event-bus 桥接
* CSRF 跨域请求伪造
* 虚拟主机

Vert.x-Web 的大多数特性被实现为了处理器（Handler），因此你随时可以实现你自己的处理器。我们预计随着时间的推移会有更多的处理器被实现。

我们会在本手册里讨论所有上述的特性。

### 使用 Vert.x Web

在使用 vert.x web 之前，需要将以下的依赖项添加到你的构建工具的描述文件中：

* Maven（在 pom.xml 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-web</artifactId>
  <version>3.4.1</version>
</dependency>
```

* Gradle（在 build.gradle 文件中）：

```gradle
dependencies {
  compile 'io.vertx:vertx-web:3.4.1'
}
```

### 回顾 Vert.x core 的 HTTP 服务器

Vert.x-Web 使用了 Vert.x core 暴露的 API，所以熟悉使用 Vert.x core 编写 HTTP 服务器的基本概念是很有价值的。

Vert.x core 的 [HTTP 文档](../core/Core.md) 有很多关于这方面的细节。

下面是一个使用 Vert.x core 编写的 hello world web 服务器，暂不涉及 Vert.x-Web：

```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(request -> {

  // This handler gets called for each request that arrives on the server
  HttpServerResponse response = request.response();
  response.putHeader("content-type", "text/plain");

  // Write to the response and end it
  response.end("Hello World!");
});

server.listen(8080);
```

我们创建了一个 HTTP 服务器，并设置了一个请求处理器。所有的请求都会调用这个处理器处理。

当请求到达时，我们设置了相应的 Content Type 为 `text/plain` 并写入了 `Hello World!` 然后结束了处理。

之后，我们告诉服务器监听 `8080` 端口（默认的主机名是 `localhost`）

你可以执行这段代码，并打开浏览器访问 [http://localhost:8080](http://localhost:8080) 来验证它是否如预期的一样工作。


### Vert.x-Web 的基本概念

[Router](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html) 是 Vert.x-Web 的核心概念之一。它是一个维护了零或多个 [Route](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html) 的对象。

Router 接受 HTTP 请求，并查找首个匹配该请求的 [Route](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)，然后将请求传递给这个 [Route](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)。

Route 可以持有一个与之关联的处理器用于接受请求。你可以通过这个处理器对请求做一些事情，然后结束响应或者把请求传递给下一个匹配的处理器。

以下是一个简单的 router 示例：

```java
HttpServer server = vertx.createHttpServer();

Router router = Router.router(vertx);

router.route().handler(routingContext -> {

  // This handler will be called for every request
  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "text/plain");

  // Write to the response and end it
  response.end("Hello World from Vert.x-Web!");
});

server.requestHandler(router::accept).listen(8080);
```

它做了和上文使用 Vert.x Core 实现的 HTTP 服务器基本相同的事情，只是这一次换成了 Vert.x-Web。

和上文一样，我们创建了一个 HTTP 服务器，然后创建了一个 [Router](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html)。在这之后，我们创建了一个没有匹配条件的 [Route](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)，这个 route 会匹配所有到达这个服务器的请求。

之后，我们为这个 route 指定了一个处理器，所有的请求都会调用这个处理器处理。

调用处理器的参数是一个 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。它不仅包含了 Vert.x 中标准的 [HttpServerRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html) 和
[HttpServerResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)，还包含了各种用于简化 Vert.x-Web 使用的东西。

每一个被路由的请求对应一个唯一的 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html)，这个实例会被传递到所有处理这个请求的处理器上。

当我们创建了处理器之后，我们设置了 HTTP 服务器的请求处理器，使所有的请求都通过 [accept](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#accept-io.vertx.core.http.HttpServerRequest-)(3)处理。

这些是基本的，下面我们来看一下更多的细节：


### 处理请求并调用下一个处理器

当 Vert.x-Web 决定路由一个请求到匹配的 route 上，他会使用一个 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 调用对应处理器。

如果你不在处理器里结束这个响应，你需要调用 [next](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--) 方法让其他匹配的 Route 来处理请求（如果有）。

你不需要在处理器执行完毕之前调用 [next](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--) 方法。你可以在之后你需要的时间点调用它：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid -> routingContext.next());
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Call the next matching route after a 5 second delay
  routingContext.vertx().setTimer(5000, tid ->  routingContext.next());
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
```

在上述的例子中，`route1` 向响应里写入了数据，5秒之后 `route2` 向响应里写入了数据，再5秒之后 `route3` 向响应里写入了数据并结束了响应。

*注意，所有发生的这些没有线程阻塞。*

### 使用阻塞式处理器

某些时候你可能需要在处理器里执行一些需要阻塞 event loop 的操作，比如调用某个传统的阻塞式 API 或者执行密集计算。

你不能在普通的处理器里执行这些操作，所以我们提供了向 route 设置阻塞式处理器的能力。

阻塞式处理器和普通处理器的区别是 Vert.x 会使用 worker pool 中的线程来执行这个处理器而不是 event loop 线程。

你可以使用 [blockingHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#blockingHandler-io.vertx.core.Handler-) 方法来设置阻塞式处理器。下面是一个例子：

```java
router.route().blockingHandler(routingContext -> {

  // Do something that might take some time synchronously
  service.doSomethingThatBlocks();

  // Now call the next handler
  routingContext.next();

});
```

默认情况下在一个 context（例如同一个 verticle 实例） 上执行的所有阻塞式处理器的执行是顺序的，也就意味着只有一个处理器执行完了才会继续执行下一个。
如果你不关心执行的顺序，并且不介意阻塞式处理器以并行的方式执行，你可以在调用 [blockingHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#blockingHandler-io.vertx.core.Handler-boolean-) 方法时将 `ordered` 设置为 false。

*注意，如果你需要在一个阻塞处理器中处理一个 multipart 类型的表单数据，你需要首先使用一个非阻塞的处理器来调用 `setExpectMultipart(true)`。下面是一个例子：*

```java
router.post("/some/endpoint").handler(ctx -> {
  ctx.request().setExpectMultipart(true);
  ctx.next();
}).blockingHandler(ctx -> {
  // ... Do some blocking operation
});
```

### 基于精确路径的路由

可以将 route 设置为只匹配指定的 URI。在这种情况下它只会匹配路径和该路径一致的请求。

在下面这个例子中会被路径为 `/some/path/` 的请求调用。我们会忽略结尾的 `/`，所以它也会被路径为 `/some/path` 或者 `/some/path//` 的请求调用：

```java
Route route = router.route().path("/some/path/");

route.handler(routingContext -> {
  // This handler will be called for the following request paths:

  // `/some/path`
  // `/some/path/`
  // `/some/path//`
  //
  // but not:
  // `/some/path/subdir`
});
```

### 基于路径前缀的路由

你经常需要为所有以某些路径开始的请求设置 route。你可以使用正则表达式来实现，但更简单的方式是在声明 route 的路径时使用一个 `*` 作为结尾。

在下面的例子中处理器会匹配所有 URI 以 `/some/path` 开头的请求。

例如 `/some/path/foo.html` 和 `/some/path/otherdir/blah.css` 都会匹配。

```java
Route route = router.route().path("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called for any path that starts with
  // `/some/path/`, e.g.

  // `/some/path`
  // `/some/path/`
  // `/some/path/subdir`
  // `/some/path/subdir/blah.html`
  //
  // but not:
  // `/some/bath`
});
```

也可以在创建 route 的时候指定任意的路径：

```java
Route route = router.route("/some/path/*");

route.handler(routingContext -> {
  // This handler will be called same as previous example
});
```

### 捕捉路径参数

可以通过占位符声明路径参数并在处理请求时通过 [params](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--) 方法获取：

以下是一个例子：

```java
Route route = router.route(HttpMethod.POST, "/catalogue/products/:producttype/:productid/");

route.handler(routingContext -> {

  String productType = routingContext.request().getParam("producttype");
  String productID = routingContext.request().getParam("productid");

  // Do something with them...
});
```

占位符由 `:` 和参数名构成。参数名由字母、数字和下划线构成。

在上述的例子中，如果一个 POST 请求的路径为  `/catalogue/products/tools/drill123/`，那么会匹配这个 Route，并且会接受到参数 `productType` 的值为 `tools`，参数 `productID` 的值为 `drill123`。

### 基于正则表达式的路由

正则表达式同样也可用于在路由时匹配 URI 路径。

```java
Route route = router.route().pathRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called for:

  // /some/path/foo
  // /foo
  // /foo/bar/wibble/foo
  // /bar/foo

  // But not:
  // /bar/wibble
});
```

或者在创建 route 时指定正则表达式：

```java
Route route = router.routeWithRegex(".*foo");

route.handler(routingContext -> {

  // This handler will be called same as previous example

});
```

### 通过正则表达式捕捉路径参数

你也可以捕捉通过正则表达式声明的路径参数，下面是一个例子：

```java
Route route = router.routeWithRegex(".*foo");

// This regular expression matches paths that start with something like:
// "/foo/bar" - where the "foo" is captured into param0 and the "bar" is captured into
// param1
route.pathRegex("\\/([^\\/]+)\\/([^\\/]+)").handler(routingContext -> {

  String productType = routingContext.request().getParam("param0");
  String productID = routingContext.request().getParam("param1");

  // Do something with them...
});
```

在上面的例子中，如果一个请求的路径为 `/tools/drill123/`，那么会匹配这个 route，并且会接受到参数 `productType` 的值为 `tools`，参数 `productID` 的值为 `drill123`。

### 基于 HTTP method 的路由

默认的，route 会匹配所有 HTTP method。

如果你需要一个 route 只匹配指定的 HTTP method，你可以使用 [method](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#method-io.vertx.core.http.HttpMethod-) 方法。

```java
Route route = router.route().method(HttpMethod.POST);

route.handler(routingContext -> {

  // This handler will be called for any POST request

});
```

或者可以再创建这个 route 时和路径一起指定：

```java
Route route = router.route(HttpMethod.POST, "/some/path/");

route.handler(routingContext -> {

  // This handler will be called for any POST request to a URI path starting with /some/path/

});
```

如果你想路由指定的 HTTP method ，你也可以使用对应的 [get](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#get--)、[post](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#post--)、[put](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#put--) 等方法。下面是一个例子：

```java
router.get().handler(routingContext -> {

  // Will be called for any GET request

});

router.get("/some/path/").handler(routingContext -> {

  // Will be called for any GET request to a path
  // starting with /some/path

});

router.getWithRegex(".*foo").handler(routingContext -> {

  // Will be called for any GET request to a path
  // ending with `foo`

});
```

如果你想一个 route 匹配不止一个 HTTP method，你可以调用 [method](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#method-io.vertx.core.http.HttpMethod-) 方法多次：

```java
Route route = router.route().method(HttpMethod.POST).method(HttpMethod.PUT);

route.handler(routingContext -> {

  // This handler will be called for any POST or PUT request

});
```

### 路由顺序

默认的 route 的匹配顺序与添加到 router 的顺序一致。

当一个请求到达时，router 会一步一步检查每一个 route 是否匹配，如果匹配则对应的处理器会被调用。

如果处理器随后调用了 [next](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--)，则下一个匹配的 route 对应的处理器（如果有）会被调用。 以此类推。

下面的例子演示了这个：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});
```

在上面的例子里，响应中会包含：

```java
route1
route2
route3
```

对于任意以 `/some/path` 开头的请求，Route 会被依次调用。

如果你想为 route 覆盖默认的顺序，你可以通过 [order](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#order-int-) 方法指定一个 integer 值。

当 route 被创建时会被赋予一个其被添加到 router 中时相应的顺序，例如第一个 route 是 0，第二个是 1，以此类推。

你可以使用特定的顺序覆盖默认的顺序。如果你需要确保一个 route 在顺序 0 的 route 之前执行，可以将其指定为负值。

让我们改变 `route2` 的值使其能在 `route1` 之前执行：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route1\n");

  // Now call the next matching route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // enable chunked responses because we will be adding data as
  // we execute over other handlers. This is only required once and
  // only if several handlers do output.
  response.setChunked(true);

  response.write("route2\n");

  // Now call the next matching route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // Now end the response
  routingContext.response().end();
});

// Change the order of route2 so it runs before route1
route2.order(-1);
```

此时响应内容会是：

```java
route2
route1
route3
```

如果两个匹配的 route 有相同的顺序值，则会按照添加它们的顺序来调用。

你也可以通过 [last](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#last--) 方法来指定 route 最后执行。


### 基于请求媒体类型（MIME types）的路由

你可以使用 [consumes](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#consumes-java.lang.String-) 方法指定 route 匹配对应 MIME 类型的请求。

在这种情况下，请求会包含一个 `content-type` 头声明了消息体的 MIME 类型。它会与通过 [consumes](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#consumes-java.lang.String-) 方法声明的值进行比较。

一般来说，`consumes` 描述了处理器能够处理的 MIME 类型。

匹配 MIME type 的过程是精确的：

```java
router.route().consumes("text/html").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html`

});
```

也可以指定精确匹配多个值（MIME 类型）：

```java
router.route().consumes("text/html").consumes("text/plain").handler(routingContext -> {

  // This handler will be called for any request with
  // content-type header set to `text/html` or `text/plain`.

});
```

基于通配符的子类型匹配也是支持的：

```java
router.route().consumes("text/*").handler(routingContext -> {

  // This handler will be called for any request with top level type `text`
  // e.g. content-type header set to `text/html` or `text/plain` will both match

});
```

你也可以匹配顶级的类型（top level type）：

```java
router.route().consumes("*/json").handler(routingContext -> {

  // This handler will be called for any request with sub-type json
  // e.g. content-type header set to `text/json` or `application/json` will both match

});
```

如果你没有在 consumers 中指定 `/`，则意味着是一个子类型（sub-type）。

### 基于客户端可接受媒体类型（MIME types acceptable）的路由

HTTP 的 `accept` 头用于表示哪些 MIME 类型的响应是客户端可接受的。

一个 `accept` 头可以包含多个用 `,` 分隔的 MIME 类型。

如果在 `accept` 头中匹配了不止一个 MIME 类型，则可以为每一个 MIME 类型追加一个 `q` 值来表示权重。q 的取值范围由 0 到 1.0。缺省值为 1.0。

例如，下面的 `accept` 头表示客户端只接受 `text/plain` 的类型。

Accept: text/plain

以下的客户端会无偏好地接受 `text/plain` 或 `text/html`。

Accept: text/plain, text/html

以下的客户端会接受 `text/plain` 或 `text/html`，但会更倾向于 `text/html`，因为其具有更高的 `q` 值（默认值为 1.0）。

Accept: text/plain; q=0.9, text/html

在这种情况下，如果服务器可以同时提供 text/plain 和 text/html，它需要提供 text/html。

你可以使用 [produces](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#produces-java.lang.String-) 来定义 route 可以提供哪些 MIME 类型。例如以下处理器可以提供 MIME 类型为 `application/json` 的响应。

```java
router.route().produces("application/json").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "application/json");
  response.write(someJSON).end();

});
```

在这种情况下这个 route 会匹配任何 `accept` 头匹配 `application/json` 的请求。例如：

Accept: application/json
Accept: application/*
Accept: application/json, text/html
Accept: application/json;q=0.7, text/html;q=0.8, text/plain

你也可以标记你的 route 提供不止一种 MIME 类型。在这种情况下，你可以使用 [getAcceptableContentType](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getAcceptableContentType--) 来找出真正被接受的 MIME 类型。

```java
router.route().produces("application/json").produces("text/html").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();

  // Get the actual MIME type acceptable
  String acceptableContentType = routingContext.getAcceptableContentType();

  response.putHeader("content-type", acceptableContentType);
  response.write(whatever).end();
});
```

在上述例子中，如果你发送一个包含如下 `accept` 头的请求：

Accept: application/json; q=0.7, text/html

那么会匹配上面的 route，并且 `acceptableContentType` 的值会是 `text/html` 因为其具有更高的 `q` 值。

### 组合路由标准

你可以用不同的方式来组合上述的路由规则，例如：

```java
Route route = router.route(HttpMethod.PUT, "myapi/orders")
                    .consumes("application/json")
                    .produces("application/json");

route.handler(routingContext -> {

  // This would be match for any PUT method to paths starting with "myapi/orders" with a
  // content-type of "application/json"
  // and an accept header matching "application/json"

});
```

### 启用和停用 Route

你可以通过 [disable](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#disable--) 方法来停用一个 Route。停用的 Route 在匹配时会被忽略。

你可以用 [enable](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#enable--) 方法来重新启用它。

### 上下文数据

你可以通过路由上下文 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 来维护任希望在请求的生命周期中经过的处理器之间共享的数据。

以下是一个例子，一个处理器设置了一些数据，另一个处理器获取它：

你可以使用 [put](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#put-java.lang.String-java.lang.Object-) 方法向上下文设置任何对象，使用 [get](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#get-java.lang.String-) 方法从上下文中获取任何对象。

一个路径为 `/some/path/other` 的请求会同时匹配两个 Route:

```java
router.get("/some/path").handler(routingContext -> {

  routingContext.put("foo", "bar");
  routingContext.next();

});

router.get("/some/path/other").handler(routingContext -> {

  String bar = routingContext.get("foo");
  // Do something with bar
  routingContext.response().end();

});
```

另一种你可以访问上下文数据的方式是使用 [data](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#data--)。


### 转发（Reroute）

(4)到目前为止，所有的路由机制允许你顺序地处理你的请求，但某些情况下你可能需要回退。处理器的顺序是动态的，上下文并没有暴露出任何关于前一个或后一个处理器的信息。有一个方式是在当前的 Router 里重启整个路由过程。

```java
router.get("/some/path").handler(routingContext -> {

  routingContext.put("foo", "bar");
  routingContext.next();

});

router.get("/some/path/B").handler(routingContext -> {
  routingContext.response().end();
});

router.get("/some/path").handler(routingContext -> {
  routingContext.reroute("/some/path/B");
});
```

从代码中可以看到，如果一个到达的请求包含路径 `/some/path`，首先第一个处理器向上下文添加了值，然后路由到了下一个处理器。第二个处理器转发到了路径 `/some/path/B`，该处理器最后结束了响应。

你可以使用路径或者同时使用路径和方法来转发。*注意，基于方法的重定向可能会带来安全问题，例如将一个通常安全的 GET 请求可能会成为 DELETE。*

也可以在失败处理器中转发。由于转发的性质，在这种情况下，当前的状态码和失败原因也会被重置。因此在转发后的处理器应该根据需要生成正确的状态码，例如：

```java
router.get("/my-pretty-notfound-handler").handler(ctx -> {
  ctx.response()
          .setStatusCode(404)
          .end("NOT FOUND fancy html here!!!");
});

router.get().failureHandler(ctx -> {
  if (ctx.statusCode() == 404) {
    ctx.reroute("/my-pretty-notfound-handler");
  } else {
    ctx.next();
  }
});
```

### 子路由（Sub-routers）

当你有很多处理器的情况下，合理的方式是将它们分隔为多个 Routers。这也有利于你在多个不用的应用中通过设置不同的根路径来复用处理器。

你可以通过将一个 Router 挂载到另一个 Router 的挂载点上来实现。挂载的 Router 被称为子路由（Sub Router）。Sub Router 上也可以挂载其他的 Sub Router。因此，你可以包含若干级别的 Sub Router。

让我们看一个 Sub Router 挂载到另一个 Route 上的例子：

这个 Sub Router 维护了一系列处理器，对应了一个虚构的 REST API。我们会将它挂载到另一个 Router 上。
例子忽略了 REST API 的具体实现：

```java
Router restAPI = Router.router(vertx);

restAPI.get("/products/:productID").handler(rc -> {

  // TODO Handle the lookup of the product....
  rc.response().write(productJSON);

});

restAPI.put("/products/:productID").handler(rc -> {

  // TODO Add a new product...
  rc.response().end();

});

restAPI.delete("/products/:productID").handler(rc -> {

  // TODO delete the product...
  rc.response().end();

});
```

如果这个 Router 是一个顶级的 Router，那么例如 `/products/product1234` 这种 url 的 GET/PUT/DELETE 请求都会调用这个 API。

如果我们已经有了一个网站包含以下的 Router：

```java
Router mainRouter = Router.router(vertx);

// Handle static resources
mainRouter.route("/static/*").handler(myStaticHandler);

mainRouter.route(".*\\.templ").handler(myTemplateHandler);
```

我们可以将这个 Sub Router 通过一个挂载点挂载到主 Router 上，这个例子使用了 `/preoductAPI`：

```java
mainRouter.mountSubRouter("/productsAPI", restAPI);
```

这意味着这个 REST API 现在可以通过这种路径访问：`/productsAPI/products/product1234`。

### 本地化

Vert.x Web 解析 `Accept-Language` 头并提供了一些识别客户端偏好的语言，以及提供通过 `quality` 排序的语言偏好列表的方法。

```java
Route route = router.get("/localized").handler( rc -> {
  // although it might seem strange by running a loop with a switch we
  // make sure that the locale order of preference is preserved when
  // replying in the users language.
  for (LanguageHeader language : rc.acceptableLanguages()) {
    switch (language.tag()) {
      case "en":
        rc.response().end("Hello!");
        return;
      case "fr":
        rc.response().end("Bonjour!");
        return;
      case "pt":
        rc.response().end("Olá!");
        return;
      case "es":
        rc.response().end("Hola!");
        return;
    }
  }
  // we do not know the user language so lets just inform that back:
  rc.response().end("Sorry we don't speak: " + rc.preferredLocale());
});
```

方法 [acceptableLocales](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#acceptableLocales--) 会返回客户端能够理解的排序好的语言列表。
如果你只关心用户偏好的语言，那么使用 [preferredLocale](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#preferredLocale--) 会返回列表的第一个元素。
如果用户没有提供，则返回空。


### 默认的 404 处理器

如果没有为请求匹配到任何 Router，Vert.x-Web 会声明一个 404 错误。

这可以被你自己实现的错误处理器处理，或者有可能被我们未来提供的错误处理器处理。
如果没有提供错误处理器，Vert.x-Web 会发送一个基本的 404 (Not Found) 响应。

### 错误处理

和设置处理器处理请求一样，你可以设置处理器处理路由过程中的失败。

失败处理器和普通的处理器具有完全一样的路由匹配标准。

例如你可以提供一个只处理在某个路径上失败的失败处理器，或某个 HTTP 方法。

这允许你在应用的不同部分设置不同的失败处理器。

下面例子中的失败处理器只会在路由路径为 `/somepath/` 的 GET 请求失败时被调用：

```java
Route route = router.get("/somepath/*");

route.failureHandler(frc -> {

  // This will be called for failures that occur
  // when routing requests to paths starting with
  // '/somepath/'

});
```

当一个处理器抛出异常，或者一个处理器通过了 [fail](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#fail-int-) 方法指定了 HTTP 状态码时，
会执行失败的路由。

从一个处理器捕捉到异常时会标记一个状态码为 `500` 的错误。

在处理这个错误时，[RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 会被传递到失败处理器里，失败处理器可以通过获取到的错误或错误编码来构造失败的响应。

```java
Route route1 = router.get("/somepath/path1/");

route1.handler(routingContext -> {

  // Let's say this throws a RuntimeException
  throw new RuntimeException("something happened!");

});

Route route2 = router.get("/somepath/path2");

route2.handler(routingContext -> {

  // This one deliberately fails the request passing in the status code
  // E.g. 403 - Forbidden
  routingContext.fail(403);

});

// Define a failure handler
// This will get called for any failures in the above handlers
Route route3 = router.get("/somepath/*");

route3.failureHandler(failureRoutingContext -> {

  int statusCode = failureRoutingContext.statusCode();

  // Status code will be 500 for the RuntimeException or 403 for the other failure
  HttpServerResponse response = failureRoutingContext.response();
  response.setStatusCode(statusCode).end("Sorry! Not today");

});
```

某些情况下失败处理器会由于使用了不支持的字符集作为状态消息而导致错误。在这种情况下，会被根据状态码将状态消息替换为默认值。
这是为了保证 HTTP 协议的语义，而不至于崩溃并断开 socket 导致协议运行的不完整。


### 处理请求消息体

你可以使用消息体处理器 [BodyHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html) 来获取请求的消息体，限制消息体大小，或者处理文件上传。

你需要保证消息体处理器能够匹配到所有你需要这个功能的请求。

由于它需要在所有异步执行之前处理请求的消息体，因此这个处理器要尽可能早地安装到 Router 上。

```java
router.route().handler(BodyHandler.create());
```

#### 获取请求的消息体

如果你知道消息体的类型是 JSON，你可以使用 [getBodyAsJson](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBodyAsJson--)；如果你知道它的类型是字符串，你可以使用 [getBodyAsString](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBodyAsString--)；否则可以通过 [getBody](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBody--) 作为 buffer 来处理。

#### 限制消息体大小

如果要限制请求消息体的大小，可以在创建消息体处理器时使用 [setBodyLimit](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html#setBodyLimit-long-) 来指定消息体的最大字节数。这对于规避由于过大的消息体导致的内存溢出的问题很有用。

如果尝试发送一个大于最大值的消息体，则会得到一个 HTTP 状态码 `413 - Request Entity Too Large` 的响应。

默认的没有消息体大小限制。

#### 合并表单属性

消息体处理器默认地会合并表单属性到请求的参数里。
如果你不需要这个行为，可以通过 [setMergeFormAttributes](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html#setMergeFormAttributes-boolean-) 来禁用。

#### 处理文件上传

消息体处理器也可以用于处理 Multipart 的文件上传。

当消息体处理器匹配到请求时，任何上传的文件会被自动地写入到上传目录中，默认地该目录为 `file-uploads`。

每一个上传的文件会被自动生成一个文件名，并可以通过 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 的 [fileUploads](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#fileUploads--) 来获得。

以下是一个例子：

```java
router.route().handler(BodyHandler.create());

router.post("/some/path/uploads").handler(routingContext -> {

  Set<FileUpload> uploads = routingContext.fileUploads();

  // Do something with uploads....

});
```

每一个上传的文件通过一个 [FileUpload](http://vertx.io/docs/apidocs/io/vertx/ext/web/FileUpload.html) 对象来描述，通过这个对象可以获得名称、文件名、大小等属性。

### 处理 Cookie

Vert.x-Web 通过 cookie 处理器 [CookieHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/CookieHandler.html) 来支持 cookie。

你需要保证 cookie 处理器器能够匹配到所有你需要这个功能的请求。

```java
router.route().handler(CookieHandler.create());
```

#### 操作 Cookie

你可以使用 [getCookie](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getCookie-java.lang.String-) 来通过名称获取 cookie 值，或者使用 [cookies](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#cookies--) 获取整个集合。

使用 [removeCookie](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#removeCookie-java.lang.String-) 来删除 cookie。

使用 [addCookie](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#addCookie-io.vertx.ext.web.Cookie-) 来添加 cookie。

当向响应中写入响应头时，cookie 的集合会自动被回写到响应里，这样浏览器就可以存储下来。

cookie 是使用 [Cookie](http://vertx.io/docs/apidocs/io/vertx/ext/web/Cookie.html) 对象来表述的。你可以通过它来获取名称、值、域名、路径或 cookie 的其他属性。

以下是一个查询和添加 cookie 的例子：

```java
router.route().handler(CookieHandler.create());

router.route("some/path/").handler(routingContext -> {

  Cookie someCookie = routingContext.getCookie("mycookie");
  String cookieValue = someCookie.getValue();

  // Do something with cookie...

  // Add a cookie - this will get written back in the response automatically
  routingContext.addCookie(Cookie.cookie("othercookie", "somevalue"));
});
```

### 处理会话

Vert.x-Web 提供了开箱即用的会话支持。

会话维持了 HTTP 请求和浏览器会话之间的关系，并提供了可以设置会话范围的信息的能力，例如一个购物篮。

Vert.x-Web 使用会话 cookie(5) 来标示一个会话。会话 cookie 是临时的，当浏览器关闭时会被删除。

我们不会在会话 cookie 中设置实际的会话数据，这个 cookie 只是在服务器上查找实际的会话时使用的标示。这个标示是一个通过安全的随机过程生成的 UUID，因此它是无法推测的(6)。

Cookie 会在 HTTP 请求和响应之间传递。因此通过 HTTPS 来使用会话功能永远是明智的。如果你尝试直接通过 HTTP 使用会话，Vert.x-Web 会给于警告。

你需要在匹配的 Route 上注册会话处理器 [SessionHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/SessionHandler.html) 来启用会话功能，并确保它能够在应用逻辑之前执行。

会话处理器会处理创建的会话 cookie 并查找会话信息，你不需要自己来实现。

#### 会话存储

你需要提供一个会话存储对象来创建会话处理器。会话存储用于为你的应用持有实际的会话数据。

会话存储负责持有一个伪随机数生成器（PRNG）用于安全地生成会话标示。PRNG 是独立于存储的，这意味着对于给定的存储 A 的会话标示是不能够派发出存储 B 的会话标示的，因为他们具有不同的种子和状态。

PRNG 默认使用混合模式，阻塞式地重置种子，非阻塞式地生成随机数(7)。PRNG 会每隔 5 分钟使用一个新的 64 位的熵作为种子。这个策略可以通过系统属性来设置：

- io.vertx.ext.auth.prng.algorithm e.g.: SHA1PRNG
- io.vertx.ext.auth.prng.seed.interval e.g.: 1000 (every second)
- io.vertx.ext.auth.prng.seed.bits e.g.: 128

大多数用户不需要配置这些值，除非你发现你的应用的性能被 PRNG 的算法所影响。

Vert.x-Web 提供了两种开箱机用的会话存储实现，你也可以编写你自己的实现。

##### 本地会话存储

该存储将会话保存在内存中，并只在当前实例中有效。

这个存储适用于你只有一个 Vert.x 实例的情况，或者你正在使用粘性会话，也就是说你可以配置你的负载均衡器来确保所有请求（来自同一用户的）永远被派发到同一个 Vert.x 实例上。

如果你不能够保证所有请求（来自同一用户的）被派发到同一个服务器上，那么就不要使用这个存储。这会导致在请求到达的服务器上无法识别请求中包含的会话。

本地会话存储基于本地的共享Map来实现，并包含了一个用于清理过期会话的回收器。

回收的周期可以通过 [LocalSessionStore.create](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/LocalSessionStore.html#create-io.vertx.core.Vertx-java.lang.String-long-) 来配置。

以下是一些创建 [LocalSessionStore](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/LocalSessionStore.html#create-io.vertx.core.Vertx-java.lang.String-long-) 的例子：

```java
SessionStore store1 = LocalSessionStore.create(vertx);

// Create a local session store specifying the local shared map name to use
// This might be useful if you have more than one application in the same
// Vert.x instance and want to use different maps for different applications
SessionStore store2 = LocalSessionStore.create(vertx, "myapp3.sessionmap");

// Create a local session store specifying the local shared map name to use and
// setting the reaper interval for expired sessions to 10 seconds
SessionStore store3 = LocalSessionStore.create(vertx, "myapp3.sessionmap", 10000);
```

##### 集群会话存储

该存储将会话保存在可以在 Vert.x 集群中访问的分布式 Map 中。

这个存储适用于你没有使用粘性会话的情况。比如你的负载均衡器会将来自同一个浏览器的不同请求转发到不同的服务器上。

通过这个存储，你的会话可以被集群中的任何节点访问。

如果要使用集群会话存储，你需要确保你的 Vert.x 实例是集群式的。

以下是一些创建 [ClusteredSessionStore](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/ClusteredSessionStore.html) 的例子：

```java
Vertx.clusteredVertx(new VertxOptions().setClustered(true), res -> {

  Vertx vertx = res.result();

  // Create a clustered session store using defaults
  SessionStore store1 = ClusteredSessionStore.create(vertx);

  // Create a clustered session store specifying the distributed map name to use
  // This might be useful if you have more than one application in the cluster
  // and want to use different maps for different applications
  SessionStore store2 = ClusteredSessionStore.create(vertx, "myclusteredapp3.sessionmap");
});
```

#### 创建会话处理器

当你创建会话存储之后，你可以创建一个会话处理器，并添加到 route 上。你需要确保会话处理器在你的应用处理器之前被执行。

由于会话处理器需要使用 cookie 来查找会话，因此你还需要包含一个 cookie 处理器。这个 cookie 处理器需要在会话处理器之前被执行。

以下是例子：

```java
Router router = Router.router(vertx);

// We need a cookie handler first
router.route().handler(CookieHandler.create());

// Create a clustered session store using defaults
SessionStore store = ClusteredSessionStore.create(vertx);

SessionHandler sessionHandler = SessionHandler.create(store);

// Make sure all requests are routed through the session handler too
router.route().handler(sessionHandler);

// Now your application handlers
router.route("/somepath/blah/").handler(routingContext -> {

  Session session = routingContext.session();
  session.put("foo", "bar");
  // etc

});
```

会话处理器会自动从会话存储中查找会话（如果没有则创建），并在你的应用处理器执行之前设置在上下文中。

#### 使用会话

在你的处理器中，你可以通过 [session](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#session--) 来访问会话对象。

你可以通过 [put](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#put-java.lang.String-java.lang.Object-) 来设置数据，通过 [get](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#get-java.lang.String-) 来获取数据，通过 [remove](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#remove-java.lang.String-) 来删除数据。

会话中的键的类型必须是字符串。本地会话存储的值可以是任何类型；集群会话存储的值类型可以是基本类型，或者 [Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)、[JsonObject](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)、[JsonArray](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 或可序列化对象。因为这些值需要在集群中进行序列化。

以下是操作会话数据的例子：

```java
router.route().handler(CookieHandler.create());
router.route().handler(sessionHandler);

// Now your application handlers
router.route("/somepath/blah").handler(routingContext -> {

  Session session = routingContext.session();

  // Put some data from the session
  session.put("foo", "bar");

  // Retrieve some data from a session
  int age = session.get("age");

  // Remove some data from a session
  JsonObject obj = session.remove("myobj");

});
```

在响应完成后会话会自动回写到存储中。

你可以使用  [destroy](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#destroy--) 来销毁一个会话。这会将这个会话同时从上下文和存储中删除。*注意，在删除会话之后，下一次通过浏览器访问并经过会话处理器处理时，新的会话会自动被创建。*

#### 会话超时

如果会话在指定的周期内没有被访问，则会超时。

当请求到达，并且会话被访问并且在响应完成会话被回写到存储是，会话会被标记为被访问的。

你也可以通过 [setAccessed](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#setAccessed--) 来人工指定会话被访问。

可以在创建会话处理器时配置超时时间。默认的超时时间是 30 分钟。

### 认证 / 授权

Vert.x-Web 提供了若干开箱机用的处理器来处理认证和授权。

#### 创建认证处理器

你需要一个 [AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html) 实例来创建认证处理器。Auth provider 用于为用户提供认证和授权。Vert.x 在 `vertx-auth` 项目中提供了若干开箱即用的 auth provider。完整的 auth provider 的配置和用法请参考 [vertx-auth 的文档](../auth/Auth.md)。

以下是一个使用 auth provider 来创建认证处理器的例子：

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);
```

#### 在你的应用中处理认证

我们来假设你希望所有路径为 `/private` 的请求都需要认证控制。为了实现这个，你需要确保你的认证处理器匹配这个路径，并在你的应用处理器之前执行：

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

// All requests to paths starting with '/private/' will be protected
router.route("/private/*").handler(basicAuthHandler);

router.route("/someotherpath").handler(routingContext -> {

  // This will be public access - no login required

});

router.route("/private/somepath").handler(routingContext -> {

  // This will require a login

  // This will have the value true
  boolean isAuthenticated = routingContext.user() != null;

});
```

如果认证处理器完成了授权和认证，它会向 [RoutingContext](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 中注入一个 [User](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html)。你可以通过 [user](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#user--) 方法在你的处理器中获取到该对象。

如果你希望在回话中存储用户对象，以避免对所有的请求都执行认证过程，你需要使用会话处理器。确保它匹配了对应的路径，并且会在认证处理器之前执行。

一旦你获取到了 user 对象，你使用它的相关方法，通过编程的方式来为用户授权。

如果你希望用户登出，你可以调用 routing context 的 [clearUser](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#clearUser--) 方法。

#### HTTP 基础认证

HTTP基础认证是适用于简单应用的简单认证手段。

在这种认证方式下， 证书会以非加密的形式在 HTTP 请求中传输。因此，使用 HTTPS 而非 HTTP 来实现你的应用是非常必要的。

当用户请求一个需要授权的资源，基础认证处理器会返回一个包含 `WWW-Authenticate` 头的 `401` 响应。浏览器会显示一个登陆窗口并提示用户输入他们的用户名和密码。

在这之后，浏览器会重新发送这个请求，并将用户名和密码以 Base64 编码的形式包含在  `Authorization` 请求头里。

当基础认证处理器收到了这些信息，它会使用用户名和密码调用配置的 [AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html) 来认证用户。如果认证成功则该处理器会尝试用户授权，如果也成功了则这个请求被允许继续路由到后续的处理器里。否则，会返回一个 `403` 的响应来表示拒绝访问。

在设置认证处理器时可以指定一系列访问资源时需要被授予的权限。

#### 重定向认证处理器

重定向认证处理器用于当未登录的用户尝试访问受保护的资源时将他们重定向到登录页上。

当用户提交登录表单，服务器会处理用户认证。如果成功，则将用户重定向到原始的资源上。

则你可以配置一个 [RedirectAuthHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/RedirectAuthHandler.html) 对象来使用重定向处理器。

你还需要配置用于处理登录页面的处理器，以及实际处理登录的处理器。我们提供了一个内置的处理器 [FormLoginHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/FormLoginHandler.html) 来处理登录的问题。

这里是一个简单的例子，使用了一个重定向认证处理器并使用默认的重定向 url `/loginpage`。

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler redirectAuthHandler = RedirectAuthHandler.create(authProvider);

// All requests to paths starting with '/private/' will be protected
router.route("/private/*").handler(redirectAuthHandler);

// Handle the actual login
// One of your pages must POST form login data
router.post("/login").handler(FormLoginHandler.create(authProvider));

// Set a static server to serve static resources, e.g. the login page
router.route().handler(StaticHandler.create());

router.route("/someotherpath").handler(routingContext -> {
  // This will be public access - no login required
});

router.route("/private/somepath").handler(routingContext -> {

  // This will require a login

  // This will have the value true
  boolean isAuthenticated = routingContext.user() != null;

});
```

#### JWT 授权

JWT 授权通过权限来保护资源不被未为授权的用户访问。

使用这个处理器涉及 2 个步骤：

- 配置一个处理器用于颁发令牌（或依靠第三方）
- 配置授权处理器来过滤请求

*注意，这两个处理器应该只能通过 HTTPS 访问。否则可能会引起由流量嗅探引起的会话劫持。*

这里是一个派发令牌的例子：

```java
Router router = Router.router(vertx);

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

router.route("/login").handler(ctx -> {
  // this is an example, authentication should be done with another provider...
  if ("paulo".equals(ctx.request().getParam("username")) && "secret".equals(ctx.request().getParam("password"))) {
    ctx.response().end(authProvider.generateToken(new JsonObject().put("sub", "paulo"), new JWTOptions()));
  } else {
    ctx.fail(401);
  }
});
```

*注意，对于持有令牌的客户端，唯一需要做的是在所有后续的的 HTTP 请求中包含 `Authoriztion` 头并写入 `Bearer <token> `*，例如：

```java
Router router = Router.router(vertx);

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

router.route("/protected/*").handler(JWTAuthHandler.create(authProvider));

router.route("/protected/somepage").handler(ctx -> {
  // some handle code...
});
```

JWT 允许你向令牌中添加任何你需要的信息，只需要在创建令牌时向 JsonObject 参数中添加数据即可。这样做服务器上不存在任何的状态，你可以在不依赖集群会话数据的情况下扩展你的应用。

```java
JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

authProvider.generateToken(new JsonObject().put("sub", "paulo").put("someKey", "some value"), new JWTOptions());
```







## 引用


## 注释

1. 指 http 协议的 method
2. [内容协商](https://zh.wikipedia.org/zh-hans/%E5%86%85%E5%AE%B9%E5%8D%8F%E5%95%86) 指允许同一个 URI 可以根据请求中的 Accept 字段来提供资源的不同版本。
3. accept 指 Router 的 [accept](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#accept-io.vertx.core.http.HttpServerRequest-) 方法。
   示例代码使用了 lambda 的 [方法引用](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html) 语法。
4. Reroute 一词没有找到合适的方式来描述，译为了 `转发`。此处有别于 HTTP 的 Redirect 或 Proxy 等概念，只是进程内的逻辑跳转。
5. 会话 cookie 也即 session cookie，特指有效期为 `session` 的 cookie。可参考 [MSDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Session_cookie) 
6. 或可称之为不可枚举的。可防止碰撞攻击。
7. 指通过 vertx.executeBlocking 来定期刷新生成器的种子，在 event loop 线程中执行生成随机数的过程。

## 结语

1. `route` 一词同时具有名词和动词的含义。为了避免混淆，原文中所有使用名词的地方都统一按照专有名词 Route / route 处理。原文中的动词统一译为 `路由`。