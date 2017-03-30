# Vert.x Web

Vert.x Web Client（Web客户端）是一个异步的HTTP和HTTP2客户端。

Web客户端使得发送HTTP请求以及从Web服务器接收HTTP响应变得更加便捷，同时提供了额外的高级功能，例如：

- Json体的编码和解码

- 请求和响应泵

- 请求参数的处理

- 统一的错误处理

- 提交表单

Web客户端的出现并未完全替代Vert.x Core中的[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)，而是基于该客户端，继承并扩展了它的设置以及便利的特性，例如请求连接池（Pooling），HTTP/2的支持，流水线／管线的支持等……当您需要对HTTP请求和响应做细微粒度控制时，您应当使用[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

另外Web客户端并未提供WebSocket API，此时您应当使用[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

# 使用Web客户端

如需使用Vert.x Web客户端，请先加入以下依赖：

- Maven（在 `pom.xml`文件中）：

  ```xml
  <dependency>

    <groupId>io.vertx</groupId>

    <artifactId>vertx-web-client</artifactId>

    <version>3.4.1</version>

  </dependency>
  ```

- Gradle（在`build.gradle`文件中）：

  ```gradle
  dependencies {

    compile 'io.vertx:vertx-web-client:3.4.1'

  }
  ```



# 对Vert.x Core HTTP客户端的回顾

Vert.x Web客户端使用Vert.x Core的API，如您对此还不熟悉，请先熟悉[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)的一些基本概念。

# 创建Web客户端

您可使用缺省设置创建一个[WebClient](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClient.html)：

```java
WebClient client = WebClient.create(vertx);
```

您亦可使用配置选项来创建客户端：

```java
WebClientOptions options = new WebClientOptions()
  .setUserAgent("My-App/1.2.3");
options.setKeepAlive(false);
WebClient client = WebClient.create(vertx, options);
```

Web Client配置选项继承自Http Client配置选项，所以您可选择其中一个。

如已在程序中创建Http Client，可用以下方式复用：

```java
WebClient client = WebClient.wrap(httpClient);
```

