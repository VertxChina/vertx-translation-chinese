# Vert.x Service Factories

> 打包和部署vert.x的独立服务。
> 翻译：赵亮，校对：Ranger Tsao、宋子豪

- [source][1]
## 使用手册
> vert.x service factory 是VerticleFactory 的一个实现，能根据服务id部署一个verticle 。
注意这个factory与vert.x服务代理没有直接关系，而是一个关于部署单个组件的设施。
服务名被用于查找JSON描述符文件，JSON描述符文件决定实际被部署的verticle，包含部署参数例如是否作为一个worker来运行等等。
服务使用者从实际被部署的verticle中解耦是很有用的，并且允许服务提供默认的部署参数和配置。

### Service 标识符
> 服务名字是一个简单的字符串，可以随用户定义，但是建议用户使用域名反转的方式（如java包的定义），这样避免你类路径中的其他服务同名。例如：

> - 推荐命名：`com.mycompany.services.clever-db-service`,`org.widgets.widget-processor`
> - 不推荐但是有效的命名：`accounting-service`,`foo`

### 使用
> 当部署一个服务时使用`service:`，选择服务verticle工厂。这个verticle可以通过编程的方式部署,例如:

```java
vertx.deployVerticle("service:com.mycompany.clever-db-service", options);
```
> 也可以通过命令行的方式部署：

```java
vertx run service:com.mycompany-clever-db-service
```

### 使其可用

> Vert.x 服务要实现 `VerticleFactory`，所以需要你的classpath中确保`vertx-service-factory` 的jar文件。首先你需要添加verticle factory的maven依赖，如果你使用的是fat jar的方式，你可以用如下依赖：

- Maven(在你的`pom.xml`中)
```
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-service-factory</artifactId>
  <version>3.2.1</version>
</dependency>
```
- Gradle(在你的`build.gradle`文件中)

```
compile 'io.vertx:vertx-service-factory:3.2.1'
```
你也可以通过编程的方式用[registerVerticleFactory][2]方法注册`VerticleFactory`实例
```
vertx.registerVerticleFactory(new ServiceVerticleFactory());
```
### 服务描述符
> 当部署一个服务时，这个服务工厂首先在classpath中查找这个文件描述符。这个文件描述器就是服务名加`.json`。

> 例如：对于一个服务名：`com.mycompany.clever-db-service`,那么他的服务描述符文件为：`com.mycompany.clever-db-service.json

> 文件描述符文件是一个简单的text文件，内容为有效的JSON对象。JSON中至少必须提供`main`属性，用于指定实际被部署的verticle，例如：

```json
{
 "main": "com.mycompany.cleverdb.MainVerticle"
}
```
或者
```json
{
  "main": "app.js"
}
```
或者你甚至可以重定向到一个不同的verticle工厂。例如，这个Maven verticle工厂在运行时动态的从Maven中加载服务：
```json
{
 "main": "maven:com.mycompany:clever-db:1,2::clever-db-service"
}
```
JSON还能提供`options`属性，可精确的被映射到`DeploymentOptions`对象中。
```json
{
  "main": "com.mycompany.cleverdb.MainVerticle",
  "options": {
    "config" : {
     "foo": "bar"
    },
    "worker": true,
    "isolationGroup": "mygroup"
  }
}
```
当使用服务描述符来部署一个服务时，任何属性如`worker`,`isolationGroup`等不能被在部署时传递的部署参数所覆盖。
但是`config`是一个例外。任意的config在部署时传递的任何配置都将覆盖任何呈现在文件描述符文件中对应的属性。

[1]: https://github.com/vert-x3/vertx-service-factory
[2]: http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#registerVerticleFactory-io.vertx.core.spi.VerticleFactory-
