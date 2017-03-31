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

为使用Vert.x Web客户端，您需要加入以下依赖到build描述文件中的依赖组中：

- Maven（在您的 `pom.xml`）：

  ```xml
  <dependency>

    <groupId>io.vertx</groupId>

    <artifactId>vertx-web-client</artifactId>

    <version>3.4.1</version>

  </dependency>
  ```

- Gradle（在您的`build.gradle`文件中）：

  ```gradle
  dependencies {

    compile 'io.vertx:vertx-web-client:3.4.1'

  }
  ```

  ​

