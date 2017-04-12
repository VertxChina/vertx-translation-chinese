# Vert.x Web - Draft

## 中英对照表


## 正文

**Vert.x-Web 是一系列用于基于 Vert.x 构建 web 应用的构建模块。**

可以把它想象成一把构建现代的、可扩展的 web 应用的瑞士军刀。

Vert.x core 提供了一系列底层的功能用于操作 HTTP，对于一部分应用来是足够的。

Vert.x-Web 基于 Vert.x core，提供了一系列更丰富的功能用于更容易地开发实际的 Web 应用。

它继承了 Vert.x 2.x 里的 [Yoke](http://pmlopes.github.io/yoke/)，灵感来自于 Node.js 的框架 [Express](http://expressjs.com/) 和 Ruby 的框架 [Sinatra](http://www.sinatrarb.com/) 等等。

Vert.x-Web 的设计是强大的，非侵入式的，并且是完全可插拔的，你只需要使用你需要的部分。Vert.x-Web 不是一个容器

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
* HTPP基本授权
* 基于重定向的授权
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

Vert.x-Web 的大多数特性被实现为了处理器，因此你随时可以实现你自己的处理器。我们预计随着时间的推移会有更多的处理器被实现。

我们会在本手册里讨论所有上述特性。

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

Vert.x-Web 使用了 Vert.x core 暴露的 API，所以熟悉 Vert.x core 编写 HTTP 服务器的基本概念是很有价值的。

Vert.x core 的 [HTTP 文档](../core/Core.md)关于这方面有很多细节。

下面是一个使用 Vert.x core 编写的 hello world web 服务器，并不涉及 Vert.x-Web：

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

我们创建了一个 HTTP 服务器，并设置了一个请求处理器。当有请求到达的时候，这个处理器就会被调用。

当这些发生的时候我们只是设置了 Content Type 为 `text/plain` 并写入了 `Hello World!` 然后结束了处理。

之后，我们告诉服务器监听 `8080` 端口（默认的主机名是 `localhost`）

你可以执行这个，并将你的浏览器指向 [http://localhost:8080](http://localhost:8080) 来验证它是否如预期的一样工作。


### Vert.x-Web 的基本概念

[Router](http://vertx.io/docs/apidocs/io/vertx/ext/web/Router.html) （路由器）是 Vert.x-Web 的核心概念之一。它是一个维护了零或多个 [Route](http://vertx.io/docs/apidocs/io/vertx/ext/web/Route.html)（路由）的对象。

Router 接受 HTTP 请求，并查找首个匹配该请求的 Route，然后将请求传递给这个 Route。

Route 可以持有一个与之关联的处理器用于接受请求。你可以针对这个请求做一些事情，然后结束它或者把请求传递给下一个匹配的处理器。




## 引用


## 注释

1. "方法" 指 http 协议的 method
1. [内容协商](https://zh.wikipedia.org/zh-hans/%E5%86%85%E5%AE%B9%E5%8D%8F%E5%95%86) 指允许同一个 URI 可以根据请求中的 Accept 字段来提供资源的不同版本。

## 结语

