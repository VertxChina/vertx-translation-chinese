# Vert.x Web

## 中英对照表

* Container：容器
* Micro-service：微服务
* Bridge：桥接
* Router：路由器
* Route：路由
* Sub-Route: 子路由
* Handler：处理器，某些特定的地方未翻译
* Blocking：阻塞式
* Context：上下文。非特别说明指代路由的上下文 routing context，不同于 Vert.x core 的 Context
* Application：应用
* Header：消息头
* Body：消息体
* MIME types：互联网媒体类型
* Load-Balancer：负载均衡器
* Socket：套接字
* Mount：挂载

## 组件介绍

**Vert.x Web 是一系列用于基于 Vert.x 构建 Web 应用的构建模块。**

可以把它想象成一把构建现代的、可伸缩的 Web 应用的瑞士军刀。

Vert.x Core 提供了一系列底层的功能用于操作 HTTP，对于一部分应用来是足够的。

Vert.x Web 基于 Vert.x Core，提供了一系列更丰富的功能以便更容易地开发实际的 Web 应用。

它继承了 Vert.x 2.x 里的 [Yoke](http://pmlopes.github.io/yoke/) 的特点，灵感来自于 Node.js 的框架 [Express](http://expressjs.com/) 和 Ruby 的框架 [Sinatra](http://www.sinatrarb.com/) 等等。

Vert.x Web 的设计是强大的，非侵入式的，并且是完全可插拔的。Vert.x Web 不是一个容器，您可以只使用您需要的部分。

您可以使用 Vert.x Web 来构建经典的服务端 Web 应用、RESTful 应用、实时的（服务端推送）Web 应用，或任何类型的您所能想到的 Web 应用。应用类型的选择取决于您，而不是 Vert.x Web。

Vert.x Web 非常适合编写 **RESTful HTTP 微服务**，但我们不强制您必须把应用实现成这样。

Vert.x Web 的一部分关键特性有：

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
* 支持服务端模板渲染，包括以下开箱即用的模板引擎：
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
* 桥接 Event-bus
* CSRF 跨域请求伪造
* 虚拟主机

Vert.x Web 的大多数特性被实现为了处理器（Handler），因此您随时可以实现您自己的处理器。我们预计随着时间的推移会有更多的处理器被实现。

我们会在本手册里讨论所有上述的特性。

## 使用 Vert.x Web

在使用 Vert.x Web 之前，需要为您的构建工具在描述文件中添加依赖项：

* Maven（在 `pom.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-web</artifactId>
  <version>3.4.1</version>
</dependency>
```

* Gradle（在 `build.gradle` 文件中）：

```groovy
dependencies {
  compile 'io.vertx:vertx-web:3.4.1'
}
```

## 回顾 Vert.x Core 的 HTTP 服务端

Vert.x Web 使用了 Vert.x Core 暴露的 API，所以熟悉基于 Vert.x Core 编写 HTTP 服务端的基本概念是很有价值的。

Vert.x Core 的 [HTTP 文档](../core/Core.md) 有很多关于这方面的细节。

下面是一个使用 Vert.x Core 编写的 Hello World Web 服务器，暂不涉及 Vert.x Web：

```java
HttpServer server = vertx.createHttpServer();

server.requestHandler(request -> {

  // 所有的请求都会调用这个处理器处理
  HttpServerResponse response = request.response();
  response.putHeader("content-type", "text/plain");

  // 写入响应并结束处理
  response.end("Hello World!");
});

server.listen(8080);
```

我们创建了一个 HTTP 服务端，并设置了一个请求处理器。所有的请求都会调用这个处理器处理。

当请求到达时，我们设置了响应的 Content Type 为 `text/plain` 并写入了 `Hello World!` 然后结束了处理。

之后，我们告诉服务器监听 `8080` 端口（默认的主机名是 `localhost`）。

您可以执行这段代码，并打开浏览器访问 [http://localhost:8080](http://localhost:8080) 来验证它是否如预期的一样工作。


## Vert.x Web 的基本概念

[`Router`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html) 是 Vert.x Web 的核心概念之一。它是一个维护了零或多个 [`Route`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html) 的对象。

Router 接收 HTTP 请求，并查找首个匹配该请求的 [`Route`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)，然后将请求传递给这个 [`Route`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)。

`Route` 可以持有一个与之关联的处理器用于接收请求。您可以通过这个处理器对请求做一些事情，然后结束响应或者把请求传递给下一个匹配的处理器。

以下是一个简单的路由示例：

```java
HttpServer server = vertx.createHttpServer();

Router router = Router.router(vertx);

router.route().handler(routingContext -> {

  // 所有的请求都会调用这个处理器处理
  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "text/plain");

  // 写入响应并结束处理
  response.end("Hello World from Vert.x-Web!");
});

server.requestHandler(router::accept).listen(8080);
```

它做了和上文使用 Vert.x Core 实现的 HTTP 服务器基本相同的事情，只是这一次换成了 Vert.x Web。

和上文一样，我们创建了一个 HTTP 服务器，然后创建了一个 [`Router`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html)。在这之后，我们创建了一个没有匹配条件的 [`Route`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)，这个 route 会匹配所有到达这个服务器的请求。

之后，我们为这个 `route` 指定了一个处理器，所有的请求都会调用这个处理器处理。

调用处理器的参数是一个 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。它不仅包含了 Vert.x 中标准的 [`HttpServerRequest`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html) 和
[`HttpServerResponse`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)，还包含了各种用于简化 Vert.x Web 使用的东西。

每一个被路由的请求对应一个唯一的 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html)，这个实例会被传递到所有处理这个请求的处理器上。

当我们创建了处理器之后，我们设置了 HTTP 服务器的请求处理器，使所有的请求都通过 [`accept`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#accept-io.vertx.core.http.HttpServerRequest-)(3)处理。

这些是最基本的，下面我们来看一下更多的细节：


## 处理请求并调用下一个处理器

当 Vert.x Web 决定路由一个请求到匹配的 `route` 上，它会使用一个 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 调用对应处理器。

如果您不在处理器里结束这个响应，您需要调用 [`next`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--) 方法让其他匹配的 `Route` 来处理请求（如果有）。

您不需要在处理器执行完毕时调用 [`next`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--) 方法。您可以在之后您需要的时间点调用它：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // 由于我们会在不同的处理器里写入响应，因此需要启用分块传输
  // 仅当需要通过多个处理器输出响应时才需要
  response.setChunked(true);

  response.write("route1\n");

  // 5 秒后调用下一个处理器
  routingContext.vertx().setTimer(5000, tid -> routingContext.next());
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // 5 秒后调用下一个处理器
  routingContext.vertx().setTimer(5000, tid ->  routingContext.next());
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // 结束响应
  routingContext.response().end();
});
```

在上述的例子中，`route1` 向响应里写入了数据，5秒之后 `route2` 向响应里写入了数据，再5秒之后 `route3` 向响应里写入了数据并结束了响应。

*注意，所有发生的这些没有线程阻塞。*

## 使用阻塞式处理器

某些时候您可能需要在处理器里执行一些需要阻塞 Event Loop 的操作，比如调用某个传统的阻塞式 API 或者执行密集计算。

您不能在普通的处理器里执行这些操作，所以我们提供了向 `Route` 设置阻塞式处理器的能力。

阻塞式处理器和普通处理器的区别是 Vert.x 会使用 Worker Pool 中的线程而不是 Event Loop 线程来处理请求。

您可以使用 [`blockingHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#blockingHandler-io.vertx.core.Handler-) 方法来设置阻塞式处理器。下面是一个例子：

```java
router.route().blockingHandler(routingContext -> {

  // 执行某些同步的耗时操作
  service.doSomethingThatBlocks();

  // 调用下一个处理器
  routingContext.next();

});
```

默认情况下在一个 Context（Vert.x Core 的 `Context`，例如同一个 Verticle 实例） 上执行的所有阻塞式处理器的执行是顺序的，也就意味着只有一个处理器执行完了才会继续执行下一个。
如果您不关心执行的顺序，并且不介意阻塞式处理器以并行的方式执行，您可以在调用 [`blockingHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#blockingHandler-io.vertx.core.Handler-boolean-) 方法时将 `ordered` 设置为 f`alse`。

*注意，如果您需要在一个阻塞处理器中处理一个 multipart 类型的表单数据，您需要首先使用一个非阻塞的处理器来调用 `setExpectMultipart(true)`。* 下面是一个例子：

```java
router.post("/some/endpoint").handler(ctx -> {
  ctx.request().setExpectMultipart(true);
  ctx.next();
}).blockingHandler(ctx -> {
  // 执行某些阻塞操作
});
```

## 基于精确路径的路由

可以将 `Route` 设置为只匹配指定的 URI。在这种情况下它只会匹配路径和该路径一致的请求。

在下面这个例子中会被路径为 `/some/path/` 的请求调用。我们会忽略结尾的 `/`，所以路径 `/some/path` 或者 `/some/path//` 的请求也是匹配的：

```java
Route route = router.route().path("/some/path/");

route.handler(routingContext -> {
  // 所有以下路径的请求都会调用这个处理器:

  // `/some/path`
  // `/some/path/`
  // `/some/path//`
  //
  // 但不包括：
  // `/some/path/subdir`
});
```

## 基于路径前缀的路由

您经常需要为所有以某些路径开始的请求设置 `Route`。您可以使用正则表达式来实现，但更简单的方式是在声明 `Route` 的路径时使用一个 `*` 作为结尾。

在下面的例子中处理器会匹配所有 URI 以 `/some/path` 开头的请求。

例如 `/some/path/foo.html` 和 `/some/path/otherdir/blah.css` 都会匹配。

```java
Route route = router.route().path("/some/path/*");

route.handler(routingContext -> {
  // 所有路径以 `/some/path/` 开头的请求都会调用这个处理器处理，例如：

  // `/some/path`
  // `/some/path/`
  // `/some/path/subdir`
  // `/some/path/subdir/blah.html`
  //
  // 但不包括：
  // `/some/bath`
});
```

也可以在创建 `Route` 的时候指定任意的路径：

```java
Route route = router.route("/some/path/*");

route.handler(routingContext -> {
  // 这个路由器的调用规则和上面的例子一样
});
```

## 捕捉路径参数

可以通过占位符声明路径参数并在处理请求时通过 [`params`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--) 方法获取：

以下是一个例子：

```java
Route route = router.route(HttpMethod.POST, "/catalogue/products/:producttype/:productid/");

route.handler(routingContext -> {

  String productType = routingContext.request().getParam("producttype");
  String productID = routingContext.request().getParam("productid");

  // 执行某些操作...
});
```

占位符由 `:` 和参数名构成。参数名由字母、数字和下划线构成。

在上述的例子中，如果一个 POST 请求的路径为  `/catalogue/products/tools/drill123/`，那么会匹配这个 `Route`，并且会接收到参数 `productType` 的值为 `tools`，参数 `productID` 的值为 `drill123`。

## 基于正则表达式的路由

正则表达式同样也可用于在路由时匹配 URI 路径。

```java
Route route = router.route().pathRegex(".*foo");

route.handler(routingContext -> {

  // 以下路径的请求都会调用这个处理器:

  // /some/path/foo
  // /foo
  // /foo/bar/wibble/foo
  // /bar/foo

  // 但不包括:
  // /bar/wibble
});
```

或者在创建 `Route` 时指定正则表达式：

```java
Route route = router.routeWithRegex(".*foo");

route.handler(routingContext -> {

  // 这个路由器的调用规则和上面的例子一样

});
```

## 通过正则表达式捕捉路径参数

您也可以捕捉通过正则表达式声明的路径参数，下面是一个例子：

```java
Route route = router.routeWithRegex(".*foo");

// 这个正则表达式可以匹配路径类似于 `/foo/bar` 的请求
// `foo` 可以通过参数 param0 获取，`bar` 可以通过参数 param1 获取
route.pathRegex("\\/([^\\/]+)\\/([^\\/]+)").handler(routingContext -> {

  String productType = routingContext.request().getParam("param0");
  String productID = routingContext.request().getParam("param1");

  // 执行某些操作
});
```

在上面的例子中，如果一个请求的路径为 `/tools/drill123/`，那么会匹配这个 route，并且会接收到参数 `productType` 的值为 `tools`，参数 `productID` 的值为 `drill123`。

## 基于 HTTP Method 的路由

默认的，`Route` 会匹配所有 HTTP Method。

如果您需要 `Route` 只匹配指定的 HTTP Method，您可以使用 [`method`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#method-io.vertx.core.http.HttpMethod-) 方法。

```java
Route route = router.route().method(HttpMethod.POST);

route.handler(routingContext -> {

  // 所有的 POST 请求都会调用这个处理器

});
```

或者可以在创建这个 `Route` 时和路径一起指定：

```java
Route route = router.route(HttpMethod.POST, "/some/path/");

route.handler(routingContext -> {

  // 所有路径为 `/some/path/` 的 POST 请求都会调用这个处理器

});
```

如果您想让 `Route` 指定的 HTTP Method ，您也可以使用对应的 [`get`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#get--)、[`post`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#post--)、[`put`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#put--) 等方法。下面是一个例子：

```java
router.get().handler(routingContext -> {

  // 所有 GET 请求都会调用这个处理器

});

router.get("/some/path/").handler(routingContext -> {

  // 所有路径为 `/some/path/` 的 GET 请求都会调用这个处理器

});

router.getWithRegex(".*foo").handler(routingContext -> {

  // 所有路径以 `foo` 结尾的 GET 请求都会调用这个处理器

});
```

如果您想要让一个路由匹配不止一个 HTTP Method，您可以调用 [method](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#method-io.vertx.core.http.HttpMethod-) 方法多次：

```java
Route route = router.route().method(HttpMethod.POST).method(HttpMethod.PUT);

route.handler(routingContext -> {

  // 所有 GET 或 POST 请求都会调用这个处理器

});
```

## 路由顺序

默认的路由的匹配顺序与添加到 `Router` 的顺序一致。

当一个请求到达时，`Router` 会一步一步检查每一个 `Route` 是否匹配，如果匹配则对应的处理器会被调用。

如果处理器随后调用了 [`next`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#next--)，则下一个匹配的 `Route` 对应的处理器（如果有）会被调用，以此类推。

下面的例子展示了这个过程：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // 由于我们会在不同的处理器里写入响应，因此需要启用分块传输
  // 仅当需要通过多个处理器输出响应时才需要
  response.setChunked(true);

  response.write("route1\n");

  // 调用下一个匹配的 route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route2\n");

  // 调用下一个匹配的 route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // 结束响应
  routingContext.response().end();
});
```

在上面的例子里，响应中会包含：

```
route1
route2
route3
```

对于任意以 `/some/path` 开头的请求，`Route`会被依次调用。

如果您想覆盖路由默认的顺序，您可以通过 [`order`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#order-int-) 方法为每一个路由指定一个 integer 值。

当 `Route` 被创建时 `order` 会被赋值为其被添加到 `Router` 时的序号，例如第一个 `Route` 是 0，第二个是 1，以此类推。

您可以使用特定的顺序值覆盖默认的顺序。如果您需要确保一个 `Route` 在顺序 0 的 `Route` 之前执行，可以将其指定为负值。

让我们改变 `route2` 的值使其能在 `route1` 之前执行：

```java
Route route1 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route1\n");

  // 调用下一个匹配的 route
  routingContext.next();
});

Route route2 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  // 由于我们会在不同的处理器里写入响应，因此需要启用分块传输
  // 仅当需要通过多个处理器输出响应时才需要
  response.setChunked(true);

  response.write("route2\n");

  // 调用下一个匹配的 route
  routingContext.next();
});

Route route3 = router.route("/some/path/").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.write("route3");

  // 结束响应
  routingContext.response().end();
});

// 更改 route2 的顺序使其可以在 route1 之前执行
route2.order(-1);
```

此时响应内容会是：

```java
route2
route1
route3
```

如果两个匹配的 `Route` 有相同的顺序值，则会按照添加它们的顺序来调用。

您也可以通过 [`last`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#last--) 方法来指定 `Route` 最后执行。

## 基于请求媒体类型（MIME types）的路由

您可以使用 [`consumes`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#consumes-java.lang.String-) 方法指定 `Route` 匹配对应 MIME 类型的请求。

在这种情况下，如果请求中包含了消息头 `content-type` 声明了消息体的 MIME 类型。则它会与通过 [`consumes`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#consumes-java.lang.String-) 方法声明的值进行比较。

一般来说，`consumes` 描述了处理器能够处理的 MIME 类型。

MIME Type 的匹配过程是精确的：

```java
router.route().consumes("text/html").handler(routingContext -> {

  // 所有 `content-type` 消息头的值为 `text/html` 的请求会调用这个处理器

});
```

也可以匹配多个精确的值（MIME 类型）：

```java
router.route().consumes("text/html").consumes("text/plain").handler(routingContext -> {

  // 所有 `content-type` 消息头的值为 `text/html` 或 `text/plain` 的请求会调用这个处理器

});
```

基于通配符的子类型匹配也是支持的：

```java
router.route().consumes("text/*").handler(routingContext -> {

  // 所有 `content-type` 消息头的顶级类型为 `text` 的请求会调用这个处理器
  // 例如 `content-type` 消息头设置为 `text/html` 或 `text/plain` 都会匹配

});
```

您也可以用通配符匹配顶级的类型（top level type）：

```java
router.route().consumes("*/json").handler(routingContext -> {

  // 所有 `content-type` 消息头的子类型为 `json` 的请求会调用这个处理器
  // 例如 `content-type` 消息头设置为 `text/json` 或 `application/json` 都会匹配

});
```

如果您没有在 consumers 中包含 `/`，则意味着是一个子类型（sub-type）。

## 基于客户端可接受媒体类型（MIME types acceptable）的路由

HTTP 的 `accept` 消息头用于表示哪些 MIME 类型的响应是客户端可接受的。

一个 `accept` 消息头可以包含多个用 `,` 分隔的 MIME 类型。

如果在 `accept` 消息头中匹配了不止一个 MIME 类型，则可以为每一个 MIME 类型追加一个 `q` 值来表示权重。q 的取值范围由 0 到 1.0。缺省值为 1.0。

例如，下面的 `accept` 消息头表示客户端只接受 `text/plain` 类型的响应。

```
Accept: text/plain
```

以下 `accept` 表示客户端会无偏好地接受 `text/plain` 或 `text/html`。

```
Accept: text/plain, text/html
```

以下 `accept` 表示客户端会接受 `text/plain` 或 `text/html`，但会更倾向于 `text/html`，因为其具有更高的 `q` 值（默认值为 1.0）。

```
Accept: text/plain; q=0.9, text/html
```

在这种情况下，如果服务器可以同时提供 `text/plain` 和 `text/html`，它需要提供 `text/html`。

您可以使用 [`produces`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#produces-java.lang.String-) 来定义 `Route` 可以提供哪些 MIME 类型。例如以下处理器可以提供 MIME 类型为 `application/json` 的响应。

```java
router.route().produces("application/json").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();
  response.putHeader("content-type", "application/json");
  response.write(someJSON).end();

});
```

在这种情况下这个 `Route` 会匹配任何 `accept` 消息头匹配 `application/json` 的请求。例如：

```
Accept: application/json
Accept: application/*
Accept: application/json, text/html
Accept: application/json;q=0.7, text/html;q=0.8, text/plain
```

您也可以标记您的 `Route` 提供不止一种 MIME 类型。在这种情况下，您可以使用 [`getAcceptableContentType`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getAcceptableContentType--) 方法来找出真正被接受的 MIME 类型。

```java
router.route().produces("application/json").produces("text/html").handler(routingContext -> {

  HttpServerResponse response = routingContext.response();

  // 获取最终匹配到的 MIME type
  String acceptableContentType = routingContext.getAcceptableContentType();

  response.putHeader("content-type", acceptableContentType);
  response.write(whatever).end();
});
```

在上述例子中，如果您发送一个包含如下 `accept` 消息头的请求：

```
Accept: application/json; q=0.7, text/html
```

那么会匹配上面的 `Route`，并且 `acceptableContentType` 的值会是 `text/html` 因为其具有更高的 `q` 值。

## 组合路由规则

您可以用不同的方式来组合上述的路由规则，例如：

```java
Route route = router.route(HttpMethod.PUT, "myapi/orders")
                    .consumes("application/json")
                    .produces("application/json");

route.handler(routingContext -> {

  // 这会匹配所有路径以 `/myapi/orders` 开头，`content-type` 值为 `application/json` 并且 `accept` 值为 `application/json` 的 PUT 请求

});
```

## 启用和停用 Route

您可以通过 [`disable`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#disable--) 方法来停用一个 `Route`。停用的 `Route` 在匹配时会被忽略。

您可以用 [`enable`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#enable--) 方法来重新启用它。

### 上下文数据

在请求的生命周期中，您可以通过路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 来维护任何您希望在处理器之间共享的数据。

以下是一个例子，一个处理器设置了一些数据，另一个处理器获取它：

您可以使用 [`put`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#put-java.lang.String-java.lang.Object-) 方法向上下文设置任何对象，使用 [`get`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#get-java.lang.String-) 方法从上下文中获取任何对象。

一个路径为 `/some/path/other` 的请求会同时匹配两个 `Route`:

```java
router.get("/some/path/*").handler(routingContext -> {

  routingContext.put("foo", "bar");
  routingContext.next();

});

router.get("/some/path/other").handler(routingContext -> {

  String bar = routingContext.get("foo");
  // 执行某些操作
  routingContext.response().end();

});
```

另一种您可以访问上下文数据的方式是使用 [`data`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#data--) 方法。


## 转发

(4) 到目前为止，通过上述的路由机制您可以顺序地处理您的请求，但某些情况下您可能需要回退。由于处理器的顺序是动态的，路由上下文并没有暴露出任何关于前一个或后一个处理器的信息。唯一的方式是在当前的 `Router` 里重启 `Route`的流程。

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

您可以使用路径或者同时使用路径和方法来转发。*注意，基于方法的重定向可能会带来安全问题，例如将一个通常安全的 GET 请求可能会成为 DELETE。*

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

## 子路由

当您有很多处理器的情况下，合理的方式是将它们分隔为多个 `Router`。这也有利于您在多个不用的应用中通过设置不同的根路径来复用处理器。

您可以通过将一个 `Router` 挂载到另一个 `Router` 的挂载点上来实现。挂载的 Router 被称为子路由（Sub Router）。Sub router 上也可以挂载其他的 sub router。因此，您可以包含若干级别的 sub router。

让我们看一个 sub router 挂载到另一个 `Router` 上的例子：

这个 sub router 维护了一系列处理器，对应了一个虚构的 REST API。我们会将它挂载到另一个 `Router` 上。
例子忽略了 REST API 的具体实现：

```java
Router restAPI = Router.router(vertx);

restAPI.get("/products/:productID").handler(rc -> {

  // TODO 查找产品信息
  rc.response().write(productJSON);

});

restAPI.put("/products/:productID").handler(rc -> {

  // TODO 添加新的产品
  rc.response().end();

});

restAPI.delete("/products/:productID").handler(rc -> {

  // TODO 删除产品
  rc.response().end();

});
```

如果这个 `Router` 是一个顶级的 `Router`，那么例如 `/products/product1234` 这种 URL 的 GET/PUT/DELETE 请求都会调用这个 API。

如果我们已经有了一个网站包含以下的 `Router`：

```java
Router mainRouter = Router.router(vertx);

// 处理静态资源
mainRouter.route("/static/*").handler(myStaticHandler);

mainRouter.route(".*\\.templ").handler(myTemplateHandler);
```

我们可以将这个 sub router 通过一个挂载点挂载到主 router 上，这个例子使用了 `/preoductAPI`：

```java
mainRouter.mountSubRouter("/productsAPI", restAPI);
```

这意味着这个 REST API 现在可以通过这种路径访问：`/productsAPI/products/product1234`。

## 本地化

Vert.x Web 解析 `Accept-Language` 消息头并提供了一些识别客户端偏好的语言，以及提供通过 `quality` 排序的语言偏好列表的方法。

```java
Route route = router.get("/localized").handler( rc -> {
  //虽然通过一个 switch 循环有点奇怪，我们必须按顺序选择正确的本地化方式
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
  // 我们不知道用户的语言，因此返回这个信息：
  rc.response().end("Sorry we don't speak: " + rc.preferredLocale());
});
```

方法 [`acceptableLocales`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#acceptableLocales--) 会返回客户端能够理解的排序好的语言列表。
如果您只关心用户偏好的语言，那么使用 [`preferredLocale`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#preferredLocale--) 会返回列表的第一个元素。
如果用户没有提供，则返回空。


### 默认的 404 处理器

如果没有为请求匹配到任何路由，Vert.x Web 会声明一个 404 错误。

这可以被您自己实现的处理器处理，或者被我们提供的专用错误处理器（`failureHandler`）处理。
如果没有提供错误处理器，Vert.x Web 会发送一个基本的 404 (Not Found) 响应。

## 错误处理

和设置处理器处理请求一样，您可以设置处理器处理路由过程中的失败。

失败处理器和普通的处理器具有完全一样的路由匹配规则。

例如您可以提供一个失败处理器只处理在某个路径上发生的失败，或某个 HTTP 方法。

这允许您在应用的不同部分设置不同的失败处理器。

下面例子中的失败处理器只会在路由路径为 `/somepath/` 的 GET 请求失败时被调用：

```java
Route route = router.get("/somepath/*");

route.failureHandler(frc -> {

  // 如果在处理路径以 `/somepath/` 开头的请求过程中发生错误，会调用这个处理器

});
```

当一个处理器抛出异常，或者一个处理器通过了 [`fail`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#fail-int-) 方法指定了 HTTP 状态码时，会执行路由的失败处理。

从一个处理器捕捉到异常时会标记一个状态码为 `500` 的错误。

在处理这个错误时，[`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 会被传递到失败处理器里，失败处理器可以通过获取到的错误或错误编码来构造失败的响应内容。

```java
Route route1 = router.get("/somepath/path1/");

route1.handler(routingContext -> {

  // 这里抛出一个 RuntimeException
  throw new RuntimeException("something happened!");

});

Route route2 = router.get("/somepath/path2");

route2.handler(routingContext -> {
  // 这里故意将请求处理为失败状态
  // 例如 403 - 禁止访问
  routingContext.fail(403);

});

// 定义一个失败处理器，上述的处理器发生错误时会调用这个处理器
Route route3 = router.get("/somepath/*");

route3.failureHandler(failureRoutingContext -> {

  int statusCode = failureRoutingContext.statusCode();

  // 对于 RuntimeException 状态码会是 500，否则是 403
  HttpServerResponse response = failureRoutingContext.response();
  response.setStatusCode(statusCode).end("Sorry! Not today");

});
```

某些情况下失败处理器会由于使用了不支持的字符集作为状态消息而导致错误。在这种情况下，Vert.x Web 会将状态消息替换为状态码的默认消息。
这是为了保证 HTTP 协议的语义，而不至于崩溃并断开 socket 导致协议运行的不完整。


## 处理请求消息体

您可以使用消息体处理器 [`BodyHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html) 来获取请求的消息体，限制消息体大小，或者处理文件上传。

您需要保证消息体处理器能够匹配到所有您需要这个功能的请求。

由于它需要在所有异步执行之前处理请求的消息体，因此这个处理器要尽可能早地设置到 router 上。

```java
router.route().handler(BodyHandler.create());
```

### 获取请求的消息体

如果您知道消息体的类型是 JSON，您可以使用 [`getBodyAsJson`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBodyAsJson--)；如果您知道它的类型是字符串，您可以使用 [`getBodyAsString`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBodyAsString--)；否则可以通过 [`getBody`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getBody--) 作为 `Buffer` 来处理。

### 限制消息体大小

如果要限制请求消息体的大小，可以在创建消息体处理器时使用 [`setBodyLimit`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html#setBodyLimit-long-) 来指定消息体的最大字节数。这对于规避由于过大的消息体导致的内存溢出的问题很有用。

如果尝试发送一个大于最大值的消息体，则会得到一个 HTTP 状态码 `413 - Request Entity Too Large` 的响应。

默认的没有消息体大小限制。

### 合并表单属性

消息体处理器默认地会合并表单属性到请求的参数里。
如果您不需要这个行为，可以通过 [`setMergeFormAttributes`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/BodyHandler.html#setMergeFormAttributes-boolean-) 来禁用。

### 处理文件上传

消息体处理器也可以用于处理 Multipart 的文件上传。

当消息体处理器匹配到请求时，所有上传的文件会被自动地写入到上传目录中，默认的该目录为 `file-uploads`。

每一个上传的文件会被自动生成一个文件名，并可以通过 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 的 [`fileUploads`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#fileUploads--) 来获得。

以下是一个例子：

```java
router.route().handler(BodyHandler.create());

router.post("/some/path/uploads").handler(routingContext -> {

  Set<FileUpload> uploads = routingContext.fileUploads();

  // 执行上传处理

});
```

每一个上传的文件通过一个 [`FileUpload`](http://vertx.io/docs/apidocs/io/vertx/ext/web/FileUpload.html) 对象来描述，通过这个对象可以获得名称、文件名、大小等属性。

## 处理 Cookie

Vert.x Web 通过 Cookie 处理器 [`CookieHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/CookieHandler.html) 来支持 cookie。

您需要保证 cookie 处理器器能够匹配到所有您需要这个功能的请求。

```java
router.route().handler(CookieHandler.create());
```

### 操作 Cookie

您可以使用 [`getCookie`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getCookie-java.lang.String-) 来通过名称获取 cookie 值，或者使用 [`cookies`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#cookies--) 获取整个集合。

使用 [`removeCookie`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#removeCookie-java.lang.String-) 来删除 cookie。

使用 [`addCookie`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#addCookie-io.vertx.ext.web.Cookie-) 来添加 cookie。

当向响应中写入响应消息头时，cookie 的集合会自动被回写到响应里，这样浏览器就可以存储下来。

cookie 是使用 [`Cookie`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Cookie.html) 对象来表述的。您可以通过它来获取名称、值、域名、路径或 cookie 的其他属性。

以下是一个查询和添加 cookie 的例子：

```java
router.route().handler(CookieHandler.create());

router.route("some/path/").handler(routingContext -> {

  Cookie someCookie = routingContext.getCookie("mycookie");
  String cookieValue = someCookie.getValue();

  // 使用 cookie 执行某些操作

  // 添加一个 cookie，会自动回写到响应里
  routingContext.addCookie(Cookie.cookie("othercookie", "somevalue"));
});
```

## 处理会话

Vert.x Web 提供了开箱即用的会话(session)支持。

会话维持了 HTTP 请求和浏览器会话之间的关系，并提供了可以设置会话范围的信息的能力，例如一个购物篮。

Vert.x Web 使用会话 cookie(5) 来标示一个会话。会话 cookie 是临时的，当浏览器关闭时会被删除。

我们不会在会话 cookie 中设置实际的会话数据，这个 cookie 只是在服务器上查找实际的会话数据时使用的标示。这个标示是一个通过安全的随机过程生成的 UUID，因此它是无法推测的(6)。

Cookie 会在 HTTP 请求和响应之间传递。因此通过 HTTPS 来使用会话功能是明智的。如果您尝试直接通过 HTTP 使用会话，Vert.x Web 会给于警告。

您需要在匹配的 `Route` 上注册会话处理器 [`SessionHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/SessionHandler.html) 来启用会话功能，并确保它能够在应用逻辑之前执行。

会话处理器会创建会话 Cookie 并查找会话信息，您不需要自己来实现。

### 会话存储

您需要提供一个会话存储对象来创建会话处理器。会话存储用于维持会话数据。

会话存储持有一个伪随机数生成器（PRNG）用于安全地生成会话标示。PRNG 是独立于存储的，这意味着对于给定的存储 A 的会话标示是不能够派发出存储 B 的会话标示的，因为他们具有不同的种子和状态。

PRNG 默认使用混合模式，阻塞式地刷新种子，非阻塞式地生成随机数(7)。PRNG 会每隔 5 分钟使用一个新的 64 位的熵作为种子。这个策略可以通过系统属性来设置：

- `io.vertx.ext.auth.prng.algorithm` e.g.: SHA1PRNG
- `io.vertx.ext.auth.prng.seed.interval` e.g.: 1000 (every second)
- `io.vertx.ext.auth.prng.seed.bits` e.g.: 128

大多数用户并不需要配置这些值，除非您发现应用的性能被 PRNG 的算法所影响。

Vert.x Web 提供了两种开箱即用的会话存储实现，您也可以编写您自己的实现。

#### 本地会话存储

该存储将会话保存在内存中，并只在当前实例中有效。

这个存储适用于您只有一个 Vert.x 实例的情况，或者您正在使用粘性会话。也就是说您可以配置您的负载均衡器来确保所有请求（来自同一用户的）永远被派发到同一个 Vert.x 实例上。

如果您不能够保证这一点，那么就不要使用这个存储。这会导致请求被派发到无法识别这个会话的服务器上。

本地会话存储基于本地的共享 Map来实现，并包含了一个用于清理过期会话的回收器。

回收的周期可以通过 [`LocalSessionStore`.create](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/LocalSessionStore.html#create-io.vertx.core.Vertx-java.lang.String-long-) 来配置。

以下是一些创建 [`LocalSessionStore`](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/LocalSessionStore.html#create-io.vertx.core.Vertx-java.lang.String-long-) 的例子：

```java
SessionStore store1 = LocalSessionStore.create(vertx);

// 通过指定的 Map 名称创建了一个本地会话存储
// 这适用于您在同一个 Vert.x 实例中有多个应用，并且希望不同的应用使用不同的 Map 的情况
SessionStore store2 = LocalSessionStore.create(vertx, "myapp3.sessionmap");

// 通过指定的 Map 名称创建了一个本地会话存储
// 设置了会话的过期时间为 10 秒
SessionStore store3 = LocalSessionStore.create(vertx, "myapp3.sessionmap", 10000);
```

#### 集群会话存储

该存储将会话保存在分布式 Map 中，该 Map 可以在 Vert.x 集群中共享访问。

这个存储适用于您没有使用粘性会话的情况。比如您的负载均衡器会将来自同一个浏览器的不同请求转发到不同的服务器上。

通过这个存储，您的会话可以被集群中的任何节点访问。

如果要使用集群会话存储，您需要确保您的 Vert.x 实例是集群模式的。

以下是一些创建 [`ClusteredSessionStore`](http://vertx.io/docs/apidocs/io/vertx/ext/web/sstore/ClusteredSessionStore.html) 的例子：

```java
Vertx.clusteredVertx(new VertxOptions().setClustered(true), res -> {

  Vertx vertx = res.result();

  // 创建了一个默认的集群会话存储
  SessionStore store1 = ClusteredSessionStore.create(vertx);

  // 通过指定的 Map 名称创建了一个集群会话存储
  // 这适用于您在集群中有多个应用，并且希望不同的应用使用不同的 Map 的情况
  SessionStore store2 = ClusteredSessionStore.create(vertx, "myclusteredapp3.sessionmap");
});
```

### 创建会话处理器

当您创建会话存储之后，您可以创建一个会话处理器，并添加到 `Route` 上。您需要确保会话处理器在您的应用处理器之前被执行。

由于会话处理器需要使用 Cookie 来查找会话，因此您还需要包含一个 Cookie 处理器。这个 Cookie 处理器需要在会话处理器之前被执行。

以下是例子：

```java
Router router = Router.router(vertx);

// 我们首先需要一个 cookie 处理器
router.route().handler(CookieHandler.create());

// 用默认值创建一个集群会话存储
SessionStore store = ClusteredSessionStore.create(vertx);

SessionHandler sessionHandler = SessionHandler.create(store);

// 确保所有请求都会经过 session 处理器
router.route().handler(sessionHandler);

// 您自己的应用处理器
router.route("/somepath/blah/").handler(routingContext -> {

  Session session = routingContext.session();
  session.put("foo", "bar");
  // etc

});
```

会话处理器会自动从会话存储中查找会话（如果没有则创建），并在您的应用处理器执行之前设置在上下文中。

### 使用会话

在您的处理器中，您可以通过 [`session`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#session--) 方法来访问会话对象。

您可以通过 [`put`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#put-java.lang.String-java.lang.Object-) 方法来向会话中设置数据，通过 [`get`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#get-java.lang.String-) 方法来获取数据，通过 [`remove`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#remove-java.lang.String-) 方法来删除数据。

会话中的键的类型必须是字符串。本地会话存储的值可以是任何类型；集群会话存储的值类型可以是基本类型，或者 [`Buffer`](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)、[`JsonObject`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)、[`JsonArray`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 或可序列化对象。因为这些值需要在集群中进行序列化。

以下是操作会话数据的例子：

```java
router.route().handler(CookieHandler.create());
router.route().handler(sessionHandler);

// 您的应用处理器
router.route("/somepath/blah").handler(routingContext -> {

  Session session = routingContext.session();

  // 向会话中设置值
  session.put("foo", "bar");

  // 从会话中获取值
  int age = session.get("age");

  // 从会话中删除值
  JsonObject obj = session.remove("myobj");

});
```

在响应完成后会话会自动回写到存储中。

您可以使用  [`destroy`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#destroy--) 方法来销毁一个会话。这会将这个会话同时从上下文和存储中删除。*注意，在删除会话之后，下一次通过浏览器访问并经过会话处理器处理时，会自动创建新的会话。*

### 会话超时

如果会话在指定的周期内没有被访问，则会超时。

当请求到达，访问了会话，并且在响应完成向会话存储回写会话时，会话会被标记为被访问的。

您也可以通过 [`setAccessed`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Session.html#setAccessed--) 来人工指定会话被访问。

可以在创建会话处理器时配置超时时间。默认的超时时间是 30 分钟。

## 认证/授权

Vert.x Web 提供了若干开箱即用的处理器来处理认证和授权。

### 创建认证处理器

您需要一个 [`AuthProvider`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html) 实例来创建认证处理器。Auth Provider 用于为用户提供认证和授权。Vert.x 在 `vertx-auth` 项目中提供了若干开箱即用的 Auth Provider。完整的 Auth Provider 的配置和用法请参考 [Vertx Auth 的文档](../auth/Auth.md)。

以下是一个使用 Auth Provider 来创建认证处理器的例子：

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);
```

### 在您的应用中处理认证

我们来假设您希望所有路径为 `/private` 的请求都需要认证控制。为了实现这个，您需要确保您的认证处理器匹配这个路径，并在您的应用处理器之前执行：

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

// 所有路径以 `/private` 开头的请求会被保护
router.route("/private/*").handler(basicAuthHandler);

router.route("/someotherpath").handler(routingContext -> {

  // 此处是公开的，不需要登录

});

router.route("/private/somepath").handler(routingContext -> {

  // 此处需要登录

  // 这个值会返回 true
  boolean isAuthenticated = routingContext.user() != null;

});
```

如果认证处理器完成了授权和认证，它会向 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 中注入一个 [`User`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html) 对象。您可以通过 [`user`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#user--) 方法在您的处理器中获取到该对象。

如果您希望在回话中存储用户对象，以避免对所有的请求都执行认证过程，您需要使用会话处理器。确保它匹配了对应的路径，并且会在认证处理器之前执行。

一旦您获取到了 `user` 对象，您可以通过编程的方式来使用它的相关方法为用户授权。

如果您希望用户登出，您可以调用上下文的 [`clearUser`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#clearUser--) 方法。

### HTTP 基础认证

HTTP基础认证是适用于简单应用的简单认证手段。

在这种认证方式下， 证书会以非加密的形式在 HTTP 请求中传输。因此，使用 HTTPS 而非 HTTP 来实现您的应用是非常必要的。

当用户请求一个需要授权的资源，基础认证处理器会返回一个包含 `WWW-Authenticate` 消息头的 `401` 响应。浏览器会显示一个登录窗口并提示用户输入他们的用户名和密码。

在这之后，浏览器会重新发送这个请求，并将用户名和密码以 Base64 编码的形式包含在请求的  `Authorization` 消息头里。

当基础认证处理器收到了这些信息，它会使用用户名和密码调用配置的 [`AuthProvider`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html) 来认证用户。如果认证成功则该处理器会尝试用户授权，如果也成功了则允许这个请求路由到后续的处理器里处理。否则，会返回一个 `403` 的响应拒绝访问。

在设置认证处理器时可以指定一系列访问资源时需要的权限。

#### 重定向认证处理器

重定向认证处理器用于当未登录的用户尝试访问受保护的资源时将他们重定向到登录页上。

当用户提交登录表单，服务器会处理用户认证。如果成功，则将用户重定向到原始的资源上。

则您可以配置一个 [`RedirectAuthHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/RedirectAuthHandler.html) 对象来使用重定向处理器。

您还需要配置用于处理登录页面的处理器，以及实际处理登录的处理器。我们提供了一个内置的处理器 [`FormLoginHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/FormLoginHandler.html) 来处理登录的问题。

这里是一个简单的例子，使用了一个重定向认证处理器并使用默认的重定向 url `/loginpage`。

```java
router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));
router.route().handler(UserSessionHandler.create(authProvider));

AuthHandler redirectAuthHandler = RedirectAuthHandler.create(authProvider);

// 所有路径以 `/private` 开头的请求会被保护
router.route("/private/*").handler(redirectAuthHandler);

// 处理登录请求
// 您的登录页需要 POST 登录表单数据
router.post("/login").handler(FormLoginHandler.create(authProvider));

// 处理静态资源，例如您的登录页
router.route().handler(StaticHandler.create());

router.route("/someotherpath").handler(routingContext -> {
  // 此处是公开的，不需要登录
});

router.route("/private/somepath").handler(routingContext -> {

  // 此处需要登录

  // 这个值会返回 true
  boolean isAuthenticated = routingContext.user() != null;

});
```

### JWT 授权

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
  // 这是一个例子，认证会由另一个 provider 执行
  if ("paulo".equals(ctx.request().getParam("username")) && "secret".equals(ctx.request().getParam("password"))) {
    ctx.response().end(authProvider.generateToken(new JsonObject().put("sub", "paulo"), new JWTOptions()));
  } else {
    ctx.fail(401);
  }
});
```

注意，对于持有令牌的客户端，唯一需要做的是在 **所有** 后续的的 HTTP 请求中包含消息头 `Authoriztion` 并写入 `Bearer <token> `，例如：

```java
Router router = Router.router(vertx);

JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

router.route("/protected/*").handler(JWTAuthHandler.create(authProvider));

router.route("/protected/somepage").handler(ctx -> {
  // 一些处理过程
});
```

JWT 允许您向令牌中添加任何您需要的信息，只需要在创建令牌时向 `JsonObject` 参数中添加数据即可。这样做服务器上不存在任何的会话状态，您可以在不依赖集群会话数据的情况下对应用进行扩展。

```java
JsonObject authConfig = new JsonObject().put("keyStore", new JsonObject()
    .put("type", "jceks")
    .put("path", "keystore.jceks")
    .put("password", "secret"));

JWTAuth authProvider = JWTAuth.create(vertx, authConfig);

authProvider.generateToken(new JsonObject().put("sub", "paulo").put("someKey", "some value"), new JWTOptions());
```

在消费时用同样的方式：

```java
Handler<RoutingContext> handler = rc -> {
  String theSubject = rc.user().principal().getString("sub");
  String someKey = rc.user().principal().getString("someKey");
};
```

### 配置所需的权限

您可以对认证处理器配置访问资源所需的权限。

默认的，如果不配置权限，那么只要登录了就可以访问资源。否则，用户不仅需要登录，而且需要具有所需的权限。

以下的例子定义了一个应用，该应用的不同部分需要不同的权限。注意，权限的含义取决于您使用的的 Auth Provider。例如一些支持角色/权限的模型，另一些可能是其他的模型。

```java
AuthHandler listProductsAuthHandler = RedirectAuthHandler.create(authProvider);
listProductsAuthHandler.addAuthority("list_products");

// 需要 `list_products` 权限来列举产品
router.route("/listproducts/*").handler(listProductsAuthHandler);

AuthHandler settingsAuthHandler = RedirectAuthHandler.create(authProvider);
settingsAuthHandler.addAuthority("role:admin");

// 只有 `admin` 可以访问 `/private/settings`
router.route("/private/settings/*").handler(settingsAuthHandler);
```

## 静态资源服务

Vert.x Web 提供了一个开箱即用的处理器来提供静态的 Web 资源。您可以非常容易地编写静态的 Web 服务器。

您可以使用静态资源处理器 [`StaticHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html) 来提供诸如 `.html`、`.css`、`.js`  或其他类型的静态资源。

每一个被静态资源处理器处理的请求都会返回文件系统的某个目录或 classpath 里的文件。文件的根目录是可以配置的，默认为 `webroot`。

在以下的例子中，所有路径以 `/static` 开头的请求都会对应到 `webroot` 目录：

```java
router.route("/static/*").handler(StaticHandler.create());
```

例如，对于一个路径为 `/static/css/mystyles.css` 的请求，静态处理器会在该路径中查找文件 `webroot/css/mystyle.css`。

它也会在 classpath 中查找文件 `webroot/css/mystyle.css`。这意味着您可以将所有的静态资源打包到一个 jar 文件（或 fat-jar）里进行分发。

当 Vert.x 在 classpath 中第一次找到一个资源时，会将它提取到一个磁盘的缓存目录中以避免每一次都重新提取。

这个处理器能够处理范围请求。当客户端请求静态资源时，该处理器会添加一个范围单位的说明到响应的消息头 `Accept-Ranges` 里来通知客户端它支持范围请求。如果后续请求的消息头 `Range` 里包含了正确的单位以及起始、终止位置，则客户端将收到包含了的 `Content-Range` 消息头的部分响应。

### 配置缓存

默认的，为了让浏览器有效地缓存文件，静态处理器会设置缓存消息头。

Vert.x Web 会在响应里设置这些消息头：`cache-control`、`last-modified`、`date`。

`cache-control` 的默认值为 `max-age=86400`，也就是一天。可以通过 [`setMaxAgeSeconds`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setMaxAgeSeconds-long-) 方法来配置。

当浏览器发送了携带消息头 `if-modified-since` 的 GET 或 HEAD 请求时，如果对应的资源在该日期之后没有修改过，则会返回一个 `304` 状态码通知浏览器使用本地的缓存资源。

如果不需要缓存的消息头，可以通过 [`setCachingEnabled`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setCachingEnabled-boolean-) 方法将其禁用。

如果启用了缓存处理，则 Vert.x Web 会将资源的最后修改日期缓存在内存里，以此来避免频繁地访问取磁盘来检查修改时间。

缓存有过期时间，在这个时间之后，会重新访问磁盘检查文件并更新缓存。

默认的，如果您的文件永远不会发生变化，则缓存内容会永远有效。

如果您的文件在服务器运行过程中可能发生变化，您可以通过 [`setFilesReadOnly`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setFilesReadOnly-boolean-) 方法设置文件的只读属性为 false。

您可以通过 [`setMaxCacheSize`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setMaxCacheSize-int-) 方法来设置内存缓存的最大数量。通过 [`setCacheEntryTimeout`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setCacheEntryTimeout-long-) 方法来设置缓存的过期时间。

### 配置索引页

所有访问根路径 `/` 的请求会被定位到索引页。默认的该文件为 `index.html`。可以通过 [`setIndexPage`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setIndexPage-java.lang.String-) 方法来设置。

### 配置跟目录

默认的，所有资源都以 `webroot` 作为根目录。可以通过 [`setWebRoot`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setWebRoot-java.lang.String-) 方法来配置。

### 隐藏文件

默认的，处理器会为隐藏文件提供服务（文件名以 `.` 开头的文件）。

如果您不需要为隐藏文件提供服务，可以通过 [`setIncludeHidden`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setIncludeHidden-boolean-) 方法来配置。

### 列举目录

静态资源处理器可以用于列举目录的文件。默认情况下该功能是关闭的。可以通过 [`setDirectoryListing`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setDirectoryListing-boolean-) 方法来启用。

当该功能启用时，会根据客户端请求的消息头 `accept` 所表示的类型来返回相应的结果。

例如对于 `text/html` 标示的请求，会使用通过 [`setDirectoryTemplate`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/StaticHandler.html#setDirectoryTemplate-java.lang.String-) 方法设置的模板来渲染文件列表。

### 禁用磁盘文件缓存

默认情况下，Vert.x 会使用当前工作目录的子目录 `.vertx` 来在磁盘上缓存通过 classpath 服务的静态资源。这对于在生产环境中通过 fat-jar 来部署的服务是很重要的。因为每一次都通过 classpath 来提取文件是低效的。

这在开发时会导致一个问题，例如当您通过 IDE 的运行配置来启动您的应用时，如果您修改了文件，缓存的文件时不会被更新的。

您可以通过设置系统属性 `vertx.disableFileCaching` 为 `false` 来禁用文件缓存。

## 处理跨域资源共享

跨域资源共享（CORS，[Cross Origin Resource Sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)）是一个安全机制。该机制允许了浏览器在一个域名下访问另一个域名的资源。

Vert.x Web 提供了一个处理器 [`CorsHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/CorsHandler.html) 来为您处理 CORS 协议。

这是一个例子：

```java
router.route().handler(CorsHandler.create("vertx\\.io").allowedMethod(HttpMethod.GET));

router.route().handler(routingContext -> {

  // 您的应用处理

});
```

## 模板引擎

Vert.x Web 为若干流行的模板引擎提供了开箱即用的支持，通过这种方式来提供生成动态页面的能力。您也可以很容易地添加您自己的实现。

模板引擎 [`TemplateEngine`](http://vertx.io/docs/apidocs/io/vertx/ext/web/templ/TemplateEngine.html) 定义了使用模板引擎的接口，当渲染模板时会调用 [`render`](http://vertx.io/docs/apidocs/io/vertx/ext/web/templ/TemplateEngine.html#render-io.vertx.ext.web.RoutingContext-java.lang.String-io.vertx.core.Handler-) 方法。

最简单的使用模板的方式不是直接调用模板引擎，而是使用模板处理器 [`TemplateHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/TemplateHandler.html)。这个处理器会根据 HTTP 请求的路径来调用模板引擎。

默认的，模板处理器会在 `templates` 目录中查找模板文件。这是可以配置的。

该处理器会返回渲染的结果，并默认设置 `Content-Type` 消息头为 `text/html`。这也是可以配置的。

您需要在创建模板处理器时提供您需要使用的模板引擎的实例。

模板引擎的实现没有内嵌在 Vert.x Web 里，您需要配置您的项目来访问它们。Vert.x Web 提供了每一种模板引擎的配置。

以下是一个例子：

```java
TemplateEngine engine = HandlebarsTemplateEngine.create();
TemplateHandler handler = TemplateHandler.create(engine);

// 这会将所有以 `/dynamic` 开头的请求路由到模板处理器上
// 例如 /dynamic/graph.hbs 会查找模板 /templates/graph.hbs
router.get("/dynamic/*").handler(handler);

// 将所有以 `.hbs` 结尾的请求路由到模板处理器上
router.getWithRegex(".+\\.hbs").handler(handler);
```

### MVEL 模板引擎

您需要在您的项目中添加这些依赖来使用 MVEL 模板引擎：`io.vertx:vertx-web-templ-mvel:3.4.1`。通过这个方法来创建 MVEL 模板引擎的实例：`io.vertx.ext.web.templ.MVELTemplateEngine#create()`。

在使用 MVEL 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.templ` 的文件。

在 MVEL 模板中可以通过 `context` 变量来访问路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。这也意味着您可以基于上下文里的任何信息来渲染模板，包括请求、响应、会话或者上下文数据。

这是一个例子：

```mvel
The request path is @{context.request().path()}

The variable 'foo' from the session is @{context.session().get('foo')}

The value 'bar' from the context data is @{context.get('bar')}
```

关于如何编写 MVEL 模板，请参考 [MVEL 模板文档](http://mvel.codehaus.org/MVEL+2.0+Templating+Guide)。

### Jade 模板引擎

> 译者注：Jade 已更名为 Pug。

您需要在您的项目中添加这些依赖来使用 Jade 模板引擎：`io.vertx:vertx-web-templ-jade:3.4.1`。通过这个方法来创建 Jade 模板引擎的实例：`io.vertx.ext.web.templ.JadeTemplateEngine#create()`。

在使用 Jade 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.jade` 的文件。

在 Jade 模板中可以通过 `context` 变量来访问路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。这也意味着您可以基于上下文里的任何信息来渲染模板，包括请求、响应、会话或者上下文数据。

这是一个例子：

```jade
!!! 5
html
  head
    title= context.get('foo') + context.request().path()
  body
```

关于如何编写 Jade 模板，请参考 [Jade4j 文档](https://github.com/neuland/jade4j)。

### Handlebars 模板引擎

您需要在您的项目中添加这些依赖来使用 Handlebars：`io.vertx:vertx-web-templ-handlebars:3.4.1`。通过这个方法来创建 Handlebars 模板引擎的实例：`io.vertx.ext.web.templ.HandlebarsTemplateEngine#create()`。

在使用 Handlebars 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.hbs` 的文件。

Handlebars 不允许在模板中随意地调用对象的方法，因此我们不能像对待其他模板引擎一样将路由上下文传递到引擎里并让模板来识别它。

替代方案是，可以使用 [`data`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#data--) 来访问上下文数据。

如果您要访问某些上下文数据里不存在的信息，比如请求的路径、请求参数或者会话等，您需要在模板处理器执行之前将他们添加到上下文数据里，例如：

```handlebars
TemplateHandler handler = TemplateHandler.create(engine);

router.get("/dynamic").handler(routingContext -> {

  routingContext.put("request_path", routingContext.request().path());
  routingContext.put("session_data", routingContext.session().data());

  routingContext.next();
});

router.get("/dynamic/").handler(handler);
```

关于如何编写 Handlebars 模板，请参考 [Handlebars Java 文档](https://github.com/jknack/handlebars.java)。

### Thymeleaf 模板引擎

您需要在您的项目中添加这些依赖来使用 Thymeleaf：`io.vertx:vertx-web-templ-thymeleaf:3.4.1`。通过这个方法来创建 Thymeleaf 模板引擎的实例：`io.vertx.ext.web.templ.ThymeleafTemplateEngine#create()`。

在使用 Thymeleaf 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.html` 的文件。

在 Thymeleaf 模板中可以通过 `context` 变量来访问路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。这也意味着您可以基于上下文里的任何信息来渲染模板，包括请求、响应、会话或者上下文数据。

这是一个例子：

```thymeleaf
[snip]
<p th:text="${context.get('foo')}"></p>
<p th:text="${context.get('bar')}"></p>
<p th:text="${context.normalisedPath()}"></p>
<p th:text="${context.request().params().get('param1')}"></p>
<p th:text="${context.request().params().get('param2')}"></p>
[snip]
```

关于如何编写 Thymeleaf 模板，请参考 [Thymeleaf 文档](http://www.thymeleaf.org/)。

### Apache FreeMarker 模板引擎

您需要在您的项目中添加这些依赖来使用 Apache FreeMarker：`io.vertx:vertx-web-templ-freemarker:3.4.1`。通过这个方法来创建 Apache FreeMarker 模板引擎的实例：`io.vertx.ext.web.templ.FreeMarkerTemplateEngine#create()`。

在使用 Apache FreeMarker 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.ftl` 的文件。

在 Apache FreeMarker 模板中可以通过 `context` 变量来访问路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。这也意味着您可以基于上下文里的任何信息来渲染模板，包括请求、响应、会话或者上下文数据。

这是一个例子：

```freemarker
[snip]
<p th:text="${context.foo}"></p>
<p th:text="${context.bar}"></p>
<p th:text="${context.normalisedPath()}"></p>
<p th:text="${context.request().params().param1}"></p>
<p th:text="${context.request().params().param2}"></p>
[snip]
```

关于如何编写 Apache FreeMarker 模板，请参考 [Apache FreeMarker 文档](http://www.freemarker.org/)。

### Pebble 模板引擎

您需要在您的项目中添加这些依赖来使用 Pebble：`io.vertx:vertx-web-templ-pebble:3.4.0-SNAPSHOT`。通过这个方法来创建 Pebble 模板引擎的实例：`io.vertx.ext.web.templ.PebbleTemplateEngine#create()`。

在使用 Pebble 模板引擎时，如果不指定模板文件的扩展名，则默认会查找扩展名为 `.peb` 的文件。

在 Pebble 模板中可以通过 `context` 变量来访问路由上下文 [`RoutingContext`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html) 对象。这也意味着您可以基于上下文里的任何信息来渲染模板，包括请求、响应、会话或者上下文数据。

这是一个例子：

```pebble
[snip]
<p th:text="{{context.foo}}"></p>
<p th:text="{{context.bar}}"></p>
<p th:text="{{context.normalisedPath()}}"></p>
<p th:text="{{context.request().params().param1}}"></p>
<p th:text="{{context.request().params().param2}}"></p>
[snip]
```

关于如何编写 Pebble 模板，请参考 [Pebble 文档](http://www.mitchellbosecke.com/pebble/home/)。

## 错误处理

您可以用模板处理器来渲染错误信息，或者使用 Vert.x Web 内置的一个 ”漂亮“ 的、开箱即用的错误处理器来渲染错误页面。

这个处理器是 [`ErrorHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/ErrorHandler.html)。您只需要在需要覆盖到的路径上将它设置为失败处理器(9)来使用它。

## 请求日志

Vert.x Web 提供了一个用于记录 HTTP 请求的处理器 [`LoggerHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/LoggerHandler.html)。

默认的，请求会通过 Vert.x 日志来记录，或者也可以配置为 jul 日志、log4j 或 slf4j。详见 [`LoggerFormat`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/LoggerFormat.html)。

## 提供网页图标

Vert.x Web 通过内置的处理器 [`FaviconHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/FaviconHandler.html) 来提供网页图标。

图标可以指定为文件系统上的某个路径，否则 Vert.x Web 默认会在 classpath 上寻找 `favicon.ico` 文件。这意味着您可以将图标打包到您的应用的 jar 包里。

## 超时处理器

Vert.x Web 提供了一个超时处理器，可以在处理时间过长时将请求超时。

通过 [TimeoutHandler](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/TimeoutHandler.html) 对象来进行配置。

如果一个请求在响应之前超时，则会给客户端返回一个 `503` 的响应。

下面的例子设置了一个超时处理器。对于所有以 `/foo` 路径开头的请求，都会在执行 5 秒后自动超时。

```java
router.route("/foo/").handler(TimeoutHandler.create(5000));
```

## 响应时间处理器

该处理器会将从接收到请求到写入响应的消息头之间的毫秒数写入到响应的 `x-response-time` 里，例如：

```
x-response-time: 1456ms
```

## Content Type 处理器

该处理器 `ResponseContentTypeHandler` 会自动设置响应的 `Content-Type` 消息头。假设我们要构建一个 RESTful 的 Web 应用，我们需要在所有处理器里设置 `Content-Type`：

```java
router.get("/api/books").produces("application/json").handler(rc -> {
  findBooks(ar -> {
    if (ar.succeeded()) {
      rc.response().putHeader("Content-Type", "application/json").end(toJson(ar.result()));
    } else {
      rc.fail(ar.cause());
    }
  });
});
```

随着 API 接口数量的增长，设置 `Content-Type` 会变得很麻烦。可以通过在 `Route` 上添加 `ResponseContentTypeHandler` 来避免这个问题：

```java
router.route("/api/*").handler(ResponseContentTypeHandler.create());
router.get("/api/books").produces("application/json").handler(rc -> {
  findBooks(ar -> {
    if (ar.succeeded()) {
      rc.response().end(toJson(ar.result()));
    } else {
      rc.fail(ar.cause());
    }
  });
});
```

这个处理器会通过 [`getAcceptableContentType`](http://vertx.io/docs/apidocs/io/vertx/ext/web/RoutingContext.html#getAcceptableContentType--) 方法来选择适当的 `Content-Type`。因此，您可以很容易地使用同一个处理器来提供不同类型的数据：

```java
router.route("/api/*").handler(ResponseContentTypeHandler.create());
router.get("/api/books").produces("text/xml").produces("application/json").handler(rc -> {
  findBooks(ar -> {
    if (ar.succeeded()) {
      if (rc.getAcceptableContentType().equals("text/xml")) {
        rc.response().end(toXML(ar.result()));
      } else {
        rc.response().end(toJson(ar.result()));
      }
    } else {
      rc.fail(ar.cause());
    }
  });
});
```

## SockJS

SockJS 是一个客户端的 JavaScript 库。它提供了类似 WebSocket 的接口为您和 SockJS 服务端建立连接。您不必关注浏览器或网络是否真的是 WebSocket。

它提供了若干不同的传输方式，并在运行时根据浏览器和网络的兼容性来选择使用哪种传输方式处理。

所有这些对您是透明的，您只需要简单地使用类似 WebSocket 的接口。

请访问 [SockJS 官方网站](https://github.com/sockjs/sockjs-client) 来获取 SockJS 的详细信息。

### SockJS 处理器

Vert.x Web 提供了一个开箱即用的处理器 [`SockJSHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandler.html) 来让您在 Vert.x Web 应用中使用 SockJS。

您需要通过 [SockJSHandler.`create`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandler.html#create-io.vertx.core.Vertx-) 方法为每一个 SockJS 的应用创建这个处理器。您也可以在创建处理器时通过 [`SockJSHandlerOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandlerOptions.html) 对象来指定配置选项。

```java
Router router = Router.router(vertx);

SockJSHandlerOptions options = new SockJSHandlerOptions().setHeartbeatInterval(2000);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx, options);

router.route("/myapp/*").handler(sockJSHandler);
```

### 处理 SockJS 套接字

您可以在服务器端设置一个处理器，这个处理器会在每次客户端创建连接时被调用：

调用这个处理器的参数是一个 [`SockJSSocket`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSSocket.html) 对象。这是一个类似套接字的接口，您可以向使用 [`NetSocket`](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 和 [`WebSocket`](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html) 那样通过它来读写数据。它实现了 [`ReadStream`](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 和 [`WriteStream`](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 接口，因此您可以将它套用（pump）到其他的读写流上。

下面的例子中的 SockJS 处理器直接使用了它读取到的数据进行回写：

```java
Router router = Router.router(vertx);

SockJSHandlerOptions options = new SockJSHandlerOptions().setHeartbeatInterval(2000);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx, options);

sockJSHandler.socketHandler(sockJSSocket -> {

  // 将数据回写
  sockJSSocket.handler(sockJSSocket::write);
});

router.route("/myapp/*").handler(sockJSHandler);
```

### SockJS 客户端

在客户端 JavaScript 环境里您需要通过 SockJS 的客户端库来建立连接。

[SockJS 客户端的地址](http://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js)

完整的细节可以在 [SockJS 的网站](https://github.com/sockjs/sockjs-client) 中找到，简单来说您会像这样使用：

```javascript
var sock = new SockJS('http://mydomain.com/myapp');

sock.onopen = function() {
  console.log('open');
};

sock.onmessage = function(e) {
  console.log('message', e.data);
};

sock.onclose = function() {
  console.log('close');
};

sock.send('test');

sock.close();
```

### 配置 SockJS 处理器

可以通过 [`SockJSHandlerOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandlerOptions.html) 对象来配置这个处理器的若干选项。

- `insertJSESSIONID`

在 cookie 中插入一个 JSESSIONID，这样负载均衡器可以保证 SockJS 会话永远转发到正确的服务器上。默认值为 `true`。

- `sessionTimeout`

对于一个正在接收响应的客户端连接，如果一段时间内没有动作，则服务端会发出一个  `close` 事件。延时时间由这个配置决定。默认的服务端会在 5 秒之后发出这个 `close` 事件。(10)

- `heartbeatInterval`

我们会每隔一段时间发送一个心跳包，用来避免由于请求时间过长导致连接被代理和负载均衡器关闭。默认的每隔 25 秒发送一个心跳包，可以通过这个设置来控制频率。

- `maxBytesStreaming`

大多数流式传输方式会在客户端保存响应的内容并且不会释放派发消息所使用的内存。这些传输方式需要定期执行垃圾回收。`max_bytes_streaming` 设置了每一个 http 流式请求所需要发送的最小字节数。超过这个值则客户端需要打开一个新的请求。将这个值设置得过小会失去流式的处理能力，使这个流式的传输方式表现得像一个轮训的传输方式一样。默认值是 128K。

- `libraryURL`

对于没有提供原生的跨域通信支持的浏览器，会使用 iframe 来进行通信。SockJS 服务器会提供一个简单的页面（在目标域名上）并放置在一个不可见的 iframe 里。在 iframe 里运行的代码和 SockJS 服务器运行在同一个域名下，因此不用担心跨域的问题。这个 iframe 也需要加载 SockJS 的客户端 JavaScript 库，这个配置就是用于指定这个 URL 的。默认情况下会使用最新发布的压缩版本 [http://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js](http://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js)。

- `disabledTransports`

这个参数用于禁用某些传输方式。可能的值包括 WEBSOCKET、EVENT_SOURCE、HTML_FILE、JSON_P、XHR。

## SockJS 桥接 Event Bus

Vert.x Web 提供了一个内置的叫做 Event Bus Bridge 的 SockJS 套接字处理器。该处理器用于将服务器端的 Vert.x 的  Event Bus 扩展到客户端的 JavaScript 运行环境里。

这将创建一个分布式的 Event Bus。这个 Event Bus 不仅可以在多个 Vert.x 实例中使用，还可以通过运行在浏览器里的 JavaScript 访问。

由此，我们可以围绕浏览器和服务器构建一个庞大的分布式 Event Bus。只要服务器之间的链接存在，浏览器不需要每一次都与同一个服务器建立链接。

这些是通过 Vert.x 提供的一个简单的客户端 JavaScript 库 `vertx-eventbus.js` 来实现的。它提供了一系列和服务器端的 Vert.x Event Bus 类似的 API。通过这些 API 可以发送或发布消息，或注册处理器来接收消息。

一个 SockJS 套接字处理器会被安装到 [`SockJSHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandler.html) 上。这个处理器用于处理 SockJS 的数据并把它桥接到服务器端的  event bus 上。

```java
Router router = Router.router(vertx);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions();
sockJSHandler.bridge(options);

router.route("/eventbus/*").handler(sockJSHandler);
```

在客户端通过使用 `vertx-eventbus.js` 库来和 Event Bus 建立连接，并发送/接收消息：

```javascript
<script src="http://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js"></script>
<script src='vertx-eventbus.js'></script>

<script>

var eb = new EventBus('http://localhost:8080/eventbus');

eb.onopen = function() {

  // 设置了一个接收数据的处理器
  eb.registerHandler('some-address', function(error, message) {
    console.log('received a message: ' + JSON.stringify(message));
  });

  // 发送消息
  eb.send('some-address', {name: 'tim', age: 587});

}

</script>
```

这个例子做的第一件事是创建了一个 Event Bus 实例：

```javascript
var eb = new EventBus('http://localhost:8080/eventbus');
```

构造函数中的参数是连接到 Event Bus 使用的 URI。由于我们创建的桥接器是以 `eventbus` 作为前缀的，因此我们需要将 URI 指向这里。

在连接打开之前，我们什么也做不了。当它打开后，会回调 `onopen` 函数处理。

您可以通过依赖管理器来获取客户端库：

- Maven （在您的 `pom.xml` 文件里）

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-web</artifactId>
  <version>3.4.1</version>
  <classifier>client</classifier>
  <type>js</type>
</dependency>
```

- Gradle（在您的 `build.gradle` 文件里）

```gradle
compile 'io.vertx:vertx-web:3.4.1:client'
```

这个库也可以通过 [NPM](https://www.npmjs.com/package/vertx3-eventbus-client) 和 [Bower](https://github.com/vert-x3/vertx-bus-bower) 来获取。

*注意， 这个 API 在 3.0.0 和 3.1.0 版本之间发生了变化，请检查变更日志。老版本的客户端仍然兼容，但新版本提供了更多的特性，并且更接近服务端的 Vert.x Event Bus API。*

### 安全的桥接

如果您像上面的例子一样启动一个桥接器，并试图发送消息，您会发现您的消息神秘地失踪了。发生了什么？

对于大多数的应用，您应该不希望客户端的 JavaScript 代码可以发送任何消息到任何的服务端处理器或其他所有浏览器上。

例如，您可能在Event Bus 上注册了一个服务，用于访问和删除数据。但我们并不希望恶意的客户端能够通过这个服务来操作数据库中的数据。并且，我们也不希望客户端能够监听所有 event bus 上的地址。

为了解决这个问题，SockJS 默认的会拒绝所有的消息。您需要告诉桥接器哪些消息是可以通过的。（例外情况是，所有的回复消息都是可以通过的）。

换句话说，桥接器的行为像是配置了 deny-all 策略的防火墙。

为桥接器配置哪些消息允许通过是很容易的。

您可以通过调用桥接器时传入的  [`BridgeOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/BridgeOptions.html) 来配置匹配规则，指定哪些输入和输出的流量是允许通过的。

每一个匹配规则对应一个 [`PermittedOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/PermittedOptions.html) 对象：

[`setAddress`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/PermittedOptions.html#setAddress-java.lang.String-)

这个配置精确地定义了消息可以被发送到哪些地址。如果您需要通过精确的地址来控制消息的话，使用这个选项。

[`setAddressRegex`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/PermittedOptions.html#setAddressRegex-java.lang.String-)

这个配置通过正则表达式来定义消息可以被发送到哪些地址。如果您需要通过正则表达式来控制消息的话，使用这个选项。如果指定了 `address`，这个选项会被忽略。

[`setMatch`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/PermittedOptions.html#setMatch-io.vertx.core.json.JsonObject-)

这个配置通过消息的接口来控制消息是否可以发送。这个配置中定义的每一个字段必须在消息中存在，并且值一致。这个配置只能用于 JSON 格式的消息。

对于一个输入的消息（例如通过客户端 JavaScript 发送到服务器），当消息到达时，Vert.x Web 会检查每一条输入许可。如果存在匹配，则消息可以通过。

对于一个输出的消息（例如通过服务器端发送给客户端 JavaScript），当消息发送时，Vert.x Web 会检查每一条输出许可。如果存在匹配，则消息可以通过。

实际的匹配过程如下：

如果指定了 `address` 字段，并且消息的目标地址与 `address` 精确匹配，则匹配成功。

如果没有指定 `address` 而是指定了 `addressRegex` 字段，并且消息的目标地址匹配了这个正则表达式，则匹配成功。

如果指定了 `match` 字段，并且消息中包含了 match 对象中的所有键值对，则匹配成功。

以下是例子：

```java
Router router = Router.router(vertx);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);


// 允许客户端向地址 `demo.orderMgr` 发送消息
PermittedOptions inboundPermitted1 = new PermittedOptions().setAddress("demo.orderMgr");

// 允许客户端向地址 `demo.orderMgr` 发送消息
// 并且 `action` 的值为 `find`、`collecton` 的值为 `albums` 消息。
PermittedOptions inboundPermitted2 = new PermittedOptions().setAddress("demo.persistor")
    .setMatch(new JsonObject().put("action", "find")
        .put("collection", "albums"));

// 允许 `wibble` 值为 `foo` 的消息.
PermittedOptions inboundPermitted3 = new PermittedOptions().setMatch(new JsonObject().put("wibble", "foo"));

// 下面定义了如何允许服务端向客户端发送消息

// 允许向客户端发送地址为 `ticker.mystock` 的消息
PermittedOptions outboundPermitted1 = new PermittedOptions().setAddress("ticker.mystock");

// 允许向客户端发送地址以 `news.` 开头的消息（例如 news.europe, news.usa, 等）
PermittedOptions outboundPermitted2 = new PermittedOptions().setAddressRegex("news\\..+");

// 将规则添加到 BridgeOptions 里
BridgeOptions options = new BridgeOptions().
    addInboundPermitted(inboundPermitted1).
    addInboundPermitted(inboundPermitted1).
    addInboundPermitted(inboundPermitted3).
    addOutboundPermitted(outboundPermitted1).
    addOutboundPermitted(outboundPermitted2);

sockJSHandler.bridge(options);

router.route("/eventbus/*").handler(sockJSHandler);
```

### 消息授权

Event Bus 桥接器可以使用 Vert.x Web 的授权功能来配置消息的访问授权。同时支持输入和输出。

这可以通过向上文所述的匹配规则中加入额外的字段来指定该匹配需要哪些权限。

通过 [`setRequiredAuthority`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/PermittedOptions.html#setRequiredAuthority-java.lang.String-) 方法来指定对于一个登录用户，需要具有哪些权限才允许访问这个消息。

这是一个例子：

```java
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

// 仅当用户已登录并且拥有权限 `place_orders`
inboundPermitted.setRequiredAuthority("place_orders");

BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);
```

用户需要登录，并被授权才能够访问消息。因此，您需要配置一个 Vert.x 认证处理器来处理登录和授权。例如：

```java
Router router = Router.router(vertx);

// 允许客户端向地址 `demo.orderService` 发送消息
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

// 仅当用户已经登录并且包含 `place_orders` 权限
inboundPermitted.setRequiredAuthority("place_orders");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
sockJSHandler.bridge(new BridgeOptions().
        addInboundPermitted(inboundPermitted));

// 设置基础认证处理器

router.route().handler(CookieHandler.create());
router.route().handler(SessionHandler.create(LocalSessionStore.create(vertx)));

AuthHandler basicAuthHandler = BasicAuthHandler.create(authProvider);

router.route("/eventbus/*").handler(basicAuthHandler);


router.route("/eventbus/*").handler(sockJSHandler);
```

### 处理 Event Bus 桥接器事件

如果您需要在在桥接器发生事件的时候得到通知，您需要在调用 [`bridge`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/SockJSHandler.html#bridge-io.vertx.ext.web.handler.sockjs.BridgeOptions-io.vertx.core.Handler-) 方法时提供一个处理器。

任何发生的事件都会被传递到这个处理器。事件由对象 [`BridgeEvent`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/BridgeEvent.html) 来描述。

事件可能是以下的某一种类型：

- **SOCKET_CREATED**

  当新的 SockJS 套接字创建时会发生该事件。

- **SOCKET_IDLE**

  当 SockJS 的套接字的空闲事件超过出事设置会发生该事件。

- **SOCKET_PING**

  当 SockJS 的套接字的 ping 时间戳被更新时会发生该事件。

- **SOCKET_CLOSED**

  当 SockJS 的套接字关闭时会发生该事件。

- **SEND**

  当试图将一个客户端消息发送到服务端时会发生该事件。

- **PUBLISH**

  当试图将一个客户端消息发布到服务端时会发生该事件。

- **RECEIVE**

  当试图将一个服务器端消息发布到客户端时会发生该事件。

- **REGISTER**

  当客户端试图注册一个处理器时会发生该事件。

- **UNREGISTER**

  当客户端试图注销一个处理器时会发生该事件。

您可以通过 [`type`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/BridgeEvent.html#type--) 方法来获得事件的类型，通过 [`getRawMessage`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/sockjs/BridgeEvent.html#getRawMessage--) 方法来获得消息原始内容。

消息的原始内容是一个如下结构的 JSON 对象：

```text
{
  "type": "send"|"publish"|"receive"|"register"|"unregister",
  "address": the event bus address being sent/published/registered/unregistered
  "body": the body of the message
}
```

事件对象同时是一个 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html) 实例。当您完成了对消息的处理，您可以用参数 `true` 来完成这个 `Future` 以执行后续的处理。

如果您不希望事件继续处理，您可以用参数 `false` 来结束这个 `Future`。这个特性可以用于定制您自己的消息过滤器、细粒度的授权或指标收集。

在下面的例子里，我们拒绝掉了所有经过桥接器并且包含 “Armadillos” 一词的消息：

```java
Router router = Router.router(vertx);

// 允许客户端向地址 `demo.orderMgr` 发送消息
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.someService");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);

sockJSHandler.bridge(options, be -> {
  if (be.type() == BridgeEventType.PUBLISH || be.type() == BridgeEventType.RECEIVE) {
    if (be.getRawMessage().getString("body").equals("armadillos")) {
      // 拒绝该消息
      be.complete(false);
      return;
    }
  }
  be.complete(true);
});

router.route("/eventbus").handler(sockJSHandler);
```

下面的例子展示了如何配置并处理 `SOCKET_IDDLE` 事件。注意，`setPingTimeout(5000)` 的作用是当 ping 消息在 5 秒内没有从客户端返回时触发 SOCKET_IDLE 事件。

```java
// 初始化 SockJS 处理器
Router router = Router.router(vertx);

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted).setPingTimeout(5000);

sockJSHandler.bridge(options, be -> {
	if (be.type() == BridgeEventType.SOCKET_IDLE) {
	    // 执行某些处理
	}

 be.complete(true);
});

router.route("/eventbus").handler(sockJSHandler);
```

在客户端 JavaScript 环境里您使用 `vertx-eventbus.js` 来创建到 Event Bus 的连接并发送和接收消息：

```javascript
<script src="http://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js"></script>
<script src='vertx-eventbus.js'></script>

<script>

var eb = new EventBus('http://localhost:8080/eventbus', {"vertxbus_ping_interval": 300000}); // sends ping every 5 minutes.

eb.onopen = function() {

 // 设置一个接收消息的回调函数
 eb.registerHandler('some-address', function(error, message) {
   console.log('received a message: ' + JSON.stringify(message));
 });

 // 发送消息
 eb.send('some-address', {name: 'tim', age: 587});
}

</script>
```

在这个例子中，第一件事是创建了一个 Event Bus 实例：

```javascript
var eb = new EventBus('http://localhost:8080/eventbus', {"vertxbus_ping_interval": 300000});
```

构造函数的第二个参数是告诉 SockJS 的库每隔 5 分钟发送一个 ping 消息。由于服务器端配置了期望每隔 5 秒收到一条 ping 消息，因此会在服务器端触发 `SOCKET_IDLE` 事件。

 您也可以在处理事件时修改原始的消息内容，例如修改消息体。对于从客户端发送来的消息，您也可以修改消息的消息头，下面是一个例子：

```java
Router router = Router.router(vertx);

// 允许客户端向地址 `demo.orderService` 发送消息
PermittedOptions inboundPermitted = new PermittedOptions().setAddress("demo.orderService");

SockJSHandler sockJSHandler = SockJSHandler.create(vertx);
BridgeOptions options = new BridgeOptions().addInboundPermitted(inboundPermitted);

sockJSHandler.bridge(options, be -> {
  if (be.type() == BridgeEventType.PUBLISH || be.type() == BridgeEventType.SEND) {
    // 添加消息头
    JsonObject headers = new JsonObject().put("header1", "val").put("header2", "val2");
    JsonObject rawMessage = be.getRawMessage();
    rawMessage.put("headers", headers);
    be.setRawMessage(rawMessage);
  }
  be.complete(true);
});

router.route("/eventbus").handler(sockJSHandler);
```

## CSRF 跨站点请求伪造

CSRF 某些时候也被称为 XSRF。它是一种可以再未授权的网站获取用户隐私数据的技术。Vet.x-Web 提供了一个处理器 [`CSRFHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/handler/CSRFHandler.html) 是您可以避免跨站点的伪造请求。

这个处理器会向所有的 GET 请求的响应里加一个独一无二的令牌作为 Cookie。客户端会在消息头里包含这个令牌。由于令牌基于 Cookie，因此需要在 `Router` 上启用 Cookie 处理器。

当开发非单页面应用，并依赖客户端来发送 `POST` 请求时，这个消息头没办法在 HTML 表单里指定。为了解决这个问题，这个令牌的值也会通过表单属性来检查。这只会发生在请求中不存在这个消息头，并且表单中包含同名属性时。例如：

```html
<form action="/submit" method="POST">
<input type="hidden" name="X-XSRF-TOKEN" value="abracadabra">
</form>
```

您需要将表单的属性设置为正确的值。填充这个值唯一的办法是通过上下文来获取键 `X-XSRF-TOKEN` 的值。这个键的名称也可以在初始化 `CSRFHandler` 时指定。

```java
router.route().handler(CookieHandler.create());
router.route().handler(CSRFHandler.create("abracadabra"));
router.route().handler(rc -> {

});
```

## 虚机主机处理器

虚机主机处理器会验证请求的主机名。如果匹配成功，则转发这个请求到注册的处理器上。否则，继续在原先的处理器链中执行。

处理器通过请求的消息头 `Host` 来进行匹配，并支持基于通配符的模式匹配。例如 `*.vertx.io` 或完整的域名 `www.vertx.io`。

```java
router.route().handler(VirtualHostHandler.create("*.vertx.io", routingContext -> {
  // 如果请求访问虚机主机 `*.vertx.io` ，执行某些处理
}));
```

## OAuth2 认证处理器

`OAuth2AuthHandler` 帮助您快速地配置基于 OAuth2 协议的安全路由。这个处理器简化了获取 authCode 的流程。下面的例子用这个处理器实现了保护资源并通过 GitHub 来授权：

```java
OAuth2Auth authProvider = GithubAuth.create(vertx, "CLIENT_ID", "CLIENT_SECRET");

// 在服务器上创建 oauth2 处理器
// 第二个参数是您提供给您的提供商的回调 URL
OAuth2AuthHandler oauth2 = OAuth2AuthHandler.create(authProvider, "https://myserver.com/callback");

// 配置回调处理器来接收 GitHub 的回调
oauth2.setupCallback(router.route());

// 保护 `/protected` 路径下的资源
router.route("/protected/*").handler(oauth2);
// 在 `/protected` 路径下挂载某些处理器
router.route("/protected/somepage").handler(rc -> {
  rc.response().end("Welcome to the protected resource!");
});

// 欢迎页
router.get("/").handler(ctx -> {
  ctx.response().putHeader("content-type", "text/html").end("Hello<br><a href=\"/protected/somepage\">Protected by Github</a>");
});
```

`OAuth2AuthHandler` 会配置一个正确的 OAuth2 回调，因此您不需要处理授权服务器的响应。一个很重要的事情是，来自授权服务器的响应只有一次有效。也就是说如果客户端对回调 URL 发起了重载操作，则会因为验证错误而请求失败。

经验法则是：当有效的回调执行时，通知客户端跳转到受保护的资源上。

就 OAuth2 规范的生态来看，使用其他的 OAuth2 提供商需要作出少许的修改。为此，Vertx Auth 提供了若干开箱即用的实现：

- Azure Active Directory [`AzureADAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/AzureADAuth.html)
- Box.com [`BoxAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/BoxAuth.html)
- Dropbox [`DropboxAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/DropboxAuth.html)
- Facebook [`FacebookAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/FacebookAuth.html)
- Foursquare [`FoursquareAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/FoursquareAuth.html)
- Github [`GithubAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/GithubAuth.html)
- Google [`GoogleAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/GoogleAuth.html)
- Instagram [`InstagramAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/InstagramAuth.html)
- Keycloak [`KeycloakAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/KeycloakAuth.html)
- LinkedIn [`LinkedInAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/LinkedInAuth.html)
- Mailchimp [`MailchimpAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/MailchimpAuth.html)
- Salesforce [`SalesforceAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/SalesforceAuth.html)
- Shopify [`ShopifyAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/ShopifyAuth.html)
- Soundcloud [`SoundcloudAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/SoundcloudAuth.html)
- Stripe [`StripeAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/StripeAuth.html)
- Twitter [`TwitterAuth`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/oauth2/providers/TwitterAuth.html)

如果您需要使用一个上述未列出的提供商，您也可以使用基本的 API 来实现，例如：

```java
OAuth2Auth authProvider = OAuth2Auth.create(vertx, OAuth2FlowType.AUTH_CODE, new OAuth2ClientOptions()
    .setClientID("CLIENT_ID")
    .setClientSecret("CLIENT_SECRET")
    .setSite("https://accounts.google.com")
    .setTokenPath("https://www.googleapis.com/oauth2/v3/token")
    .setAuthorizationPath("/o/oauth2/auth"));

// 在域名 `http://localhost:8080` 上创建 oauth2 处理器
OAuth2AuthHandler oauth2 = OAuth2AuthHandler.create(authProvider, "http://localhost:8080");

// 配置需要的权限
oauth2.addAuthority("profile");

// 配置回调处理器来接收 Google 的回调
oauth2.setupCallback(router.get("/callback"));

// 保护 `/protected` 路径下的资源
router.route("/protected/*").handler(oauth2);
// 在 `/protected` 路径下挂载某些处理器
router.route("/protected/somepage").handler(rc -> {
  rc.response().end("Welcome to the protected resource!");
});

// 欢迎页
router.get("/").handler(ctx -> {
  ctx.response().putHeader("content-type", "text/html").end("Hello<br><a href=\"/protected/somepage\">Protected by Google</a>");
});
```

您需要手工提供所有关于您所使用的提供商的细节，但结果是一样的。

这个处理器会在您的应用上绑定回调的 URL。用法很简单，只需要为这个处理器提供一个路由(`Route`)，其他的配置都会自动完成。一个典型的情况是您的 OAuth2 提供商会需要您来提供您的应用的 callback url，则您的输入类似于 [`https://myserver.com/callback`](https://myserver.com/callback)。这是您的处理器的第二个参数。至此，您完成所有必须的配置，只需要通过 `setupCallback` 方法来启动它即可。

以上就是如何在您的服务器上绑定处理器 [https://myserver.com:8447/callback](https://myserver.com:8447/callback)。*注意，端口号可以不使用默认值。*

```java
OAuth2AuthHandler oauth2 = OAuth2AuthHandler.create(provider, "https://myserver.com:8447/callback");
// 允许该处理器为您处理回调地址
oauth2.setupCallback(router.route());
```

在这个例子中，`Route` 对象通过 `Router.route()` 创建。如果您需要完整的控制处理器的执行顺序（例如您期望它在处理链中首先被执行），您也可以先创建这个 `Route` 对象，然后将引用传进这个方法里。

### 混合 OAuth2 和 JWT

 一些 OAuth2 的提供商参考了 [RFC6750](https://tools.ietf.org/html/rfc6750) 规范，使用 JWT 令牌来作为访问令牌。这对于需要混合基于客户端的授权和基于 API 的授权很有用。例如您的应用提供了一些受保护的 HTML 文档，同时您又希望他可以作为 API 被消费。在这种情况下，一个 API 不能够很容易的处理 OAuth2 需要的重定向握手，但可以提供令牌(11)。

只要提供商被配置为支持 JWT，OAuth 处理器会自动处理这个问题。

这意味着您的 API 可以通过提供值为 `Bearer BASE64_ACCESS_TOKEN` 的消息头 `Authorization` 来访问受保护的资源。

## 注释

1. 指 HTTP 协议的 Method
2. [内容协商](https://zh.wikipedia.org/zh-hans/%E5%86%85%E5%AE%B9%E5%8D%8F%E5%95%86) 指允许同一个 URI 可以根据请求中的 Accept 字段来提供资源的不同版本。
3. accept 指 Router 的 [`accept`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html#accept-io.vertx.core.http.HttpServerRequest-) 方法。示例代码使用了 Java 8 Lambda 的 [方法引用](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html) 语法。
4. Reroute 一词没有找到合适的方式来描述，译为了 `转发`。此处有别于 HTTP 的 Redirect 或 Proxy 等概念，只是进程内的逻辑跳转。
5. 会话 Cookie 也即 Session Cookie，特指有效期为 `session` 的 Cookie。可参考 [MSDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Session_cookie)。
6. 或可称之为不可枚举的。可防止碰撞攻击。
7. 指通过 `vertx.executeBlocking` 方法来定期刷新生成器的种子，在 Event Loop 线程中同步执行生成随机数的过程。
8. 即 [`Route.failureHandler`](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html#failureHandler-io.vertx.core.Handler-)。
9. 实际上不同的 transport 具有不同的会话处理机制。sessionTimeout 主要针对轮询方式的 transport，例如 xhr。服务器端返回一个响应之后，客户端一旦接受了响应，会立刻再发一个 request 出来继续等下一个消息。如果超过了默认的 5 秒该会话没有收到新的请求，则会认为客户端断开了连接，会话过期。
10. 关于 OAuth2 如何通过 JWT 来进行授权，可以 [参考这里](https://tools.ietf.org/html/rfc7523)。

## 结语

`route` 一词同时具有名词和动词的含义。为了避免混淆，原文中所有使用名词的地方都统一按照专有名词 Route / route 处理。原文中的动词统一译为 `路由`。原文的最后几部分关于 `SockJS` 和 `OAuth2` 的内容写作风格明显和前文不同，而且有些地方描述的很简略（例如 OAuth 流程的细节、SockJS 的不同 Transport 之间的差异等）。本着翻译准确的原则，本译文没有进一步展开描述。
