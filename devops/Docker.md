# Vert.x Docker

> 翻译：[Ranger Tsao](https://github.com/boliza)

# 简介

[Docker](https://www.docker.com/) 是一个可以将应用部署在其中的轻量级、隔离的容器。应用程序并行运行在隔离的 Linux 容器中。如果从未使用过 Docker ，可以根据[官方教程](https://docs.docker.com/get-started/)，轻松入门

Vert.x 提供两个 Docker 镜像给开发人员运行部署程序，分别是：

- `vertx/vertx3` 基础镜像，需要进行一些扩展才可以运行程序
- `vertx/vertx3-exec` 给宿主系统提供 `vertx` 命令行

这两个 Docker 镜像均发布在 [Docker Hub](https://hub.docker.com/u/vertx/) 中。

> 现在公开镜像的仓库已经更名为 Docker Store

本指南将介绍如何使用这 Docker 镜像，以及如何使用 Maven 自动生成 Docker 镜像，生成 Fabric8 元数据并使用 `fat jar` 。

# 基础镜像

`vertx/vertx3` 能让开发人员轻松的在 Docker 容器中运行 Vert.x 应用。为实现该目的，开发人员须通过创建自定义 `Dockerfile` 的形式，继承 `vertx/vertx3` 。这样便可以通过容器中的 `vertx` 命令启动应用程序。

## 在 Docker 容器中部署 JavaScript Verticle

编写一个简单的 JS Verticle ，代码如下：

**hello-verticle.js**
```javascript
vertx.createHttpServer().requestHandler(function (request) {
    request.response().end("Hello world");
}).listen(8080);
```

在同一目录，创建一个 Dockerfile ，代码如下：

**Dockerfile**
```docker
# 继承 vert.x 镜像                          (1)
FROM vertx/vertx3

# 设置要部署的 verticle 名字                 (2)
ENV VERTICLE_NAME hello-verticle.js

# 设置 verticles 存放路径                    (3)
ENV VERTICLE_HOME /usr/verticles

EXPOSE 8080

# 拷贝 verticle 至容器中                     (4)
COPY $VERTICLE_NAME $VERTICLE_HOME/

# 启动 verticle                             (5)
WORKDIR $VERTICLE_HOME
ENTRYPOINT ["sh", "-c"]
CMD ["exec vertx run $VERTICLE_NAME -cp $VERTICLE_HOME/*"]
```

步骤说明：

1. 继承 vert.x 镜像
2. 设置要部署的 verticle 名字
3. 设置 verticles 存放路径
4. 拷贝 verticle 至容器中
5. 启动 verticle

接下来就可以构建 Docker 镜像了：

```bash
docker build -t sample/vertx-javascript .
```

最后执行 `docker run -t -i -p 8080:8080 sample/vertx-javascript` 便可以启动应用了。

这时候可以打开浏览器验证成果了，在浏览器输入地址 http://localhost:8080 （如果是 docker 引擎是 boot2docker 的话，则输入 http://192.168.59.103:8080 ）。

> '-t -i' 参数意味着是互动模式。按 `CTRL + C` 可以停止容器。点击 [`Docker run reference`](https://docs.docker.com/engine/reference/run/) 查阅 Docker `run` 命令详细信息。

可能已经注意到 `Dockerfile` 中的 `EXPOSE 8080` 和运行命令中的 `-p 8080:8080` 。第一个是可选的信息，告诉应用程序想要监听8080端口。第二个是强制性的，并指示码头工具将8080端口从主机转发到容器的8080端口。

> 可能还注意到复杂且令人费解的应用程序的启动方式。不是直接调用 `vertx`，而是与 `exec` 一起使用 `sh -c`。 `sh -c` 是转向Docker限制，而不是在 `CMD` 中扩展变量。关于 `Docker` 构建器文档的更多细节，查阅[文档](https://docs.docker.com/engine/reference/builder/cmd) 。 `exec` 是使 `vertx` 命令进程替换 `shell` ，以便获取pid 1并接收信号，比如在运行 `docker stop` 时，获取 `SIGTERM` 。如果没有 `exec` ，那么 `shell `会随着 `vertx` 命令进程一直运行，而且 `vertx` 命令难以得到信号，从而阻止正常关闭。

## 在 Docker 容器中部署 Groovy Verticle

同部署 JavaScript Verticle，区别在与 Verticle 代码及文件后缀名。在上述例子中，`hello_verticle.js` 在此则是 `hello_verticle.groovy` ，代码如下：

**hello-verticle.groovy**
```groovy
vertx.createHttpServer().requestHandler({ request ->
    request.response().end("Groovy world")
}).listen(8080)
```

> 注意在后续编写 `Dockerfile` 时 引入的 Verticle 名字 则是 `hello_verticle.groovy` 。

## 在 Docker 容器中部署 Ruby Verticle

同部署 JavaScript Verticle，区别在与 Verticle 代码及文件后缀名。在上述例子中，`hello_verticle.js` 在此则是 `hello_verticle.rb` ，代码如下：

**hello-verticle.rb**
```ruby
$vertx.create_http_server().request_handler() { |request|
    request.response().end("A ruby world full of gems")
}.listen(8080)
```

> 注意在后续编写 `Dockerfile` 时 引入的 Verticle 名字 则是 `hello_verticle.rb` 。

## 在 Docker 容器中部署 Java Verticle

同样，部署 Java Verticle 与前面的例子依然差不多，区别在于将 `verticle jar` 文件复制到容器。先看 Java 代码：

**io.vertx.sample.hello.HelloVerticle**
```java
package io.vertx.sample.hello;

import io.vertx.core.AbstractVerticle;

public class HelloVerticle extends AbstractVerticle {

  @Override
  public void start() throws Exception {
    vertx.createHttpServer().requestHandler(request -> {
      request.response().end("Hello Java world");
    }).listen(8080);
  }
}
```

显而易见，本 `verticle` 被打包到 `target/hello-verticle-1.0.-SNAPSHOT.jar` 文件中。所以 `Dockerfile` 需要复制这个文件，同时也必须告知 `vertx` 需要执行的 `verticle` 类名：

**Dockerfile**
```docker
# Extend vert.x image
FROM vertx/vertx3

#                                                       (1)
ENV VERTICLE_NAME io.vertx.sample.hello.HelloVerticle
ENV VERTICLE_FILE target/hello-verticle-1.0-SNAPSHOT.jar

# Set the location of the verticles
ENV VERTICLE_HOME /usr/verticles

EXPOSE 8080

# Copy your verticle to the container                   (2)
COPY $VERTICLE_FILE $VERTICLE_HOME/

# Launch the verticle
WORKDIR $VERTICLE_HOME
ENTRYPOINT ["sh", "-c"]
CMD ["exec vertx run $VERTICLE_NAME -cp $VERTICLE_HOME/*"]
```

1. 同前面的例子不一样，需要设置 verticle 类名及对应的文件名
2. 拷贝 jar 文件至 `$VERTICLE_HOME`

同理，执行命令依然没有什么变化：

```bash
> docker build -t sample/vertx-java .
....
> docker run -t -i -p 8080:8080 sample/vertx-java
```

## 配置

在前面所提到的 `Dockerfile` 并没有对 Vert.x 进行相关配置。下面的几个章节，从 5 个方面着重介绍如何配置。

### 配置 JVM 参数

可以通过设置 JAVA_OPTS 环境变量来配置 Java 虚拟机，代码如下：

```docker
ENV JAVA_OPTS "-Dfoo=bar"
```

### 配置 vertx 参数

可以使用 `VERTX_OPTS` 环境变量配置 Vert.x 特定的系统变量：

```docker
ENV VERTX_OPTS "-Dvertx.options.eventLoopPoolSize=26 -Dvertx.options.deployment.worker=true"
```

### Classpath

使用 Vert.x 命令的 `-cp` 参数或 设置 `CLASSPATH` 环境变量来配置应用程序的类路径：

```docker
ENV CLASSPATH "/usr/verticles/libs/foo.jar:/usr/verticles/libs/bar.jar:"
```

### 日志

要配置 `logging.properties` 文件（自定义 JUL 日志记录器），请设置 `VERTX_JUL_CONFIG` 环境变量：

```docker
COPY ./logging.properties $VERTICLE_HOME/                       (1)
ENV VERTX_JUL_CONFIG $VERTICLE_HOME/logging.properties          (2)
```

1. 拷贝 `logging.properties` 日志配置文件
2. 设置 `VERTX_JUL_CONFIG` 环境变量

### 集群

可以提供自定义的 `cluster.xml `文件，并将其添加到类路径中。要从 `$VERTICLE_HOME` 中包含的所有文件构建动态类路径，可以使用：

```docker
COPY ./cluster.xml $VERTICLE_HOME/
# ...
CMD [export CLASSPATH=`find $VERTICLE_HOME -printf '%p:' | sed 's/:$//'`; exec vertx run $VERTICLE_NAME"]
```

注意导出 `CLASSPATH = ...;` 部分在 `CMD` 指令中。它从 `$VERTICLE_HOME` 目录的内容构建 `CLASSPATH` 变量。这种技巧对于计算大型和动态类路径非常有用。

## 通过 Maven 构建镜像

在Maven构建过程中，有几个 `Maven`插件来构建 `Docker` 映像。此示例采用来自 Spotify 的 [`docker-maven-plugin`](https://github.com/spotify/docker-maven-plugin) 。

首先，同往常一样创建 Java 项目。源代码位于 `src/main/java` ，然后创建一个 `src/main/docker` 目录，并在其中创建一个 `Dockerfile` ，目录结构如下：

```
.
├── pom.xml
├── src
│   └── main
│       ├── docker
│       │   └── Dockerfile
│       └── java
│           └── io
│               └── vertx
│                   └── example
│                       └── HelloWorldVerticle.java
├── target
```

然后在 `pom.xml` 文件增加相应的配置，代码如下：

```xml
<groupId>com.spotify</groupId>
<artifactId>docker-maven-plugin</artifactId>
<version>0.2.8</version>
<executions>
  <execution>
    <id>docker</id>
    <phase>package</phase>
    <goals>
      <goal>build</goal>
    </goals>
  </execution>
</executions>
<configuration>
  <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
  <!-- Configure the image name -->
  <imageName>sample/vertx-hello</imageName>
  <resources>
    <resource>
      <targetPath>/verticles</targetPath>
      <directory>${project.build.directory}</directory>
      <includes>
        <include>${project.artifactId}-${project.version}.jar</include>
      </includes>
    </resource>
    <!-- don't forget to also add all the dependencies required by your application -->
  </resources>
</configuration>
</plugin>
```

该插件会将列出的内容复制到 `target/docker` 中。所有的资源文件 都被复制到 `targetPath` 中。所以编辑 `src /main/docker/Dockerfile` 并添加以下内容：

```docker
FROM vertx/vertx3

ENV VERTICLE_HOME /usr/verticles
ENV VERTICLE_NAME io.vertx.example.HelloWorldVerticle

COPY ./verticles $VERTICLE_HOME

ENTRYPOINT ["sh", "-c"]
CMD ["exec vertx run $VERTICLE_NAME -cp $VERTICLE_HOME/*"]
```

这和上面的例子中的 `Dockerfile` 文件内容基本相同。稍微有点区别的是，插件已将文件拷贝至 `Dockerfile` 所在的目录中。

通过命令 `mvn clean package` 便可构建 Docker 镜像了。

## 构建 Fabric 8 平台使用的镜像

[Fabric 8](https://fabric8.io/) 是一个开源集成开发平台，为基于 Kubernetes 和 OpenShift V3 的微服务提供管理、持续发布、 iPaas 设施。可以在 Fabric 8 执行 包含 Vert.x 应用程序的 Docker 镜像。不过需要做一些额外工作，添加额外的元数据。在这个例子中，将使用 RolandHuß 的[docker-maven-plugin](https://github.com/rhuss/docker-maven-plugin)。

首先来看目录结构：

```

├── pom.xml
├── src
│   └── main
│       ├── docker
│       │   └── assembly.xml
│       └── java
│           └── io
│               └── vertx
│                   └── example
│                       └── HelloWorldVerticle.java
└── target
```

与 Spotify 的 maven 插件不同，此插件以一个 `assembly.xml` 作为打包描述文件。该文件列出了需要复制到 docker 容器的所有文件，如：

```xml
<assembly>
   <dependencySets>
     <dependencySet>
       <includes>
         <include>:${project.artifactId}</include>
       </includes>
       <outputDirectory>.</outputDirectory>
     </dependencySet>
   </dependencySets>
 </assembly>
```

`Dockerfile在` 其余部分的配置在 `pom.xml` 文件中进行配置：

```xml
<plugin>
  <groupId>org.jolokia</groupId>
  <artifactId>docker-maven-plugin</artifactId>
  <version>0.11.5</version>
  <executions>
    <execution>
      <id>build</id>
      <phase>package</phase>
      <goals>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <images>
      <image>
        <name>${docker.image}</name>
        <build>
          <from>vertx/vertx3</from>
          <tags>
            <tag>${project.version}</tag>
          </tags>
          <ports>
            <port>8080</port>
          </ports>
          <command>vertx run io.vertx.example.HelloWorldVerticle -cp
            /usr/verticles/${project.artifactId}-${project.version}.jar
          </command>
          <assembly>
            <mode>dir</mode>
            <basedir>/usr/verticles</basedir>
            <descriptor>assembly.xml</descriptor>
          </assembly>
        </build>
      </image>
    </images>
  </configuration>
 </plugin>
```

如需更精细地配置容器，请参考[手册](https://dmp.fabric8.io/)。 `Dockerfile` 中的所有指令都可以在插件中进行设置。

> 在上面的 `pom.xml` 文件使用名为 `docker.image` 的属性来设置镜像名称。不要忘记将它添加至 `pom.xml` 文件中

一旦有了这个配置，便需要 [`fabric8-maven-plugin`](https://github.com/fabric8io/fabric8-maven-plugin) 插件来生成 Fabric8 所需的元数据：

```xml
<plugin>
<groupId>io.fabric8</groupId>
<artifactId>fabric8-maven-plugin</artifactId>
<version>2.1.4</version>
<executions>
  <execution>
    <id>json</id>
    <phase>generate-resources</phase>
    <goals>
      <goal>json</goal>
    </goals>
  </execution>
  <execution>
    <id>attach</id>
    <phase>package</phase>
    <goals>
      <goal>attach</goal>
    </goals>
  </execution>
</executions>
</plugin>
```

一旦设置，就可以使用：`mvn clean package` 指令来构建 `docker` 映像。它将创建 Fabric8 所需的 `kubernates.json` 文件。通过指令 `docker push $DOCKER_REGISTRY/sample/vertx-hello` 就可以将创建的镜像推送到由 Fabric8 提供的 Docker 注册表上。

同时不要忘记将 `DOCKER_REGISTRY` 的 URL 设置为指向由 Fabric8 管理的注册表。最后一步是应用它：

```bash
mvn io.fabric8:fabric8-maven-plugin:2.1.4:apply
```

# 可执行镜像

`vertx/vertx3-exec` 镜像为其容器提供了 `vertx` 命令行功能，因此在机器上不需要安装 Vert.x ，只需要使用本 docker 镜像即可。

举个栗子：

```bash
> docker run -i -t vertx/vertx3-exec -version
3.4.1
```

运行 verticle ：

```bash
docker run -i -t -p 8080:8080 \
    -v $PWD:/verticles vertx/vertx3-exec \
    run io.vertx.sample.RandomGeneratorVerticle \
    -cp /verticles/MY_VERTICLE.jar
```

## 自定义技术栈

`vertx/vertx3-exec` 镜像提供默认的完整的 Vert.x 堆栈。如果需要自定义此堆栈，并创建自己的 `exec` 映像。首先，创建一个 `vertx-stack.json` 文件：

```json
{
  "variables": {
    "vertx.version": "3.3.3"
  },
  "dependencies": [
    {
      "groupId": "io.vertx",
      "artifactId": "vertx-web",
      "version": "${vertx.version}",
      "included": true
    },
    {
      "groupId": "io.vertx",
      "artifactId": "vertx-lang-js",
      "version": "${vertx.version}",
      "included": true
    }
  ]
}
```

可以在文件中列出所需的任何依赖关系，而不仅仅是 Vert.x 组件（有关详细信息，请参阅 [Stack Manager](StackManager.md) 文档）。

最后编写 `Dockerfile` ：

```docker
FROM vertx/vertx3-exec                                     (1)

COPY vertx-stack.json ${VERTX_HOME}/vertx-stack.json       (2)

RUN vertx resolve && rm -rf ${HOME}/.m2                    (3)
```

1. 继承 `vertx/vertx3-exec` 镜像
2. 替换 `vertx-stack.json`
3. 解析依赖

构建 Docker 镜像：

```docker
docker build -t mycompany/my-vertx3-exec .
```

执行 Verticle ：

```bash
docker run -i -t -p 8080:8080 \
    -v $PWD:/verticles mycompany/my-vertx3-exec \
    run io.vertx.sample.RandomGeneratorVerticle \
    -cp /verticles/MY_VERTICLE.jar
```

# 部署 fat jar

可以将一个打包成 `Fat jar` 的 Vert.x 应用程序部署到 docker 容器中。为此，不需要 Vert.x 提供的镜像，直接使用基本的 Java 镜像即可。举个栗子：

```docker
FROM openjdk:8-jre-alpine                                           (1)

ENV VERTICLE_FILE hello-verticle-fatjar-3.0.0-SNAPSHOT-fat.jar      (2)

# Set the location of the verticles
ENV VERTICLE_HOME /usr/verticles

EXPOSE 8080

# Copy your fat jar to the container
COPY target/$VERTICLE_FILE $VERTICLE_HOME/                          (3)

# Launch the verticle
WORKDIR $VERTICLE_HOME
ENTRYPOINT ["sh", "-c"]
CMD ["exec java -jar $VERTICLE_FILE"]                               (4)

```

1. 扩展 OpenJDK 8 镜像
2. 将 `VERTICLE_FILE` 设置为指向 *fat jar*
3. 从 target 目录中复制 *fat jar* 。如果不使用 Maven ，请根据实际情况修改
4. 使用 `java` 指令启动应用程序

它基本上是与以前的 `Dockerfile` 类似。区别在于，这次我们扩展的是 `java:8` 而不是 `vertx/vertx3` 镜像。 然后我们将 *fat jar* 复制到容器中，并使用 `java` 指令来启动。当然，上述所有配置设置仍然有效的。

最后构建并启动容器：

```docker
> docker build -t sample/vertx-java-fat .
....
> docker run -t -i -p 8080:8080 sample/vertx-java-fat
```

---

[原文档](http://vertx.io/docs/vertx-docker)更新于2017-03-15 15:54:14 CET