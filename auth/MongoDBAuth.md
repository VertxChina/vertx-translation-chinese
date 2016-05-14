# Vert.x MongoDB Auth

* 引用：[http://vertx.io/docs/vertx-auth-mongo/java/](http://vertx.io/docs/vertx-auth-mongo/java/)


_基本术语_

* __Authentication__：认证
* __Authorisation__：授权
* __Authority__：权限
* __Permission__：许可
* __Role__：角色
* __User__：用户
* __Token__：令牌
* __Principal__：凭证
* __Handler__：处理器
* __Collection__：集合，MongoDB专用（概念稍同Database的Table）
* __Document__：文档，MongoDB专用（概念稍同Database中Row）

Vert.X中提供了一个[AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html)的实现，它可以让你使用[MongoClient](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html)针对MongoDb数据库执行认证和授权。若要在自己的项目中使用它，则需要在构建描述信息的_dependencies_节点中添加如下信息：

* Maven（在`pom.xml`文件中）

        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-auth-mongo</artifactId>
            <version>3.2.1</version>
        </dependency>
* Gradle（在`build.gradle`文件中）

        compile io.vertx:vertx-auth-mongo:3.2.1

如果要创建一个客户端实例，你首先需要一个[MongoClient](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html)的实例，要知道如何创建这个实例可按照文档中的内容实施。

一旦你创建了一个[MongoClient](http://vertx.io/docs/apidocs/io/vertx/ext/mongo/MongoClient.html)实例后，就可以按照下边的代码创建[MongoAuth](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html)实例：

    MongoClient client = MongoClient.createShared(vertx, mongoClientConfig);
    JsonObject authProperties = new JsonObject();
    MongoAuth authProvider = MongoAuth.create(client, authProperties);

创建好上边的实例过后，你就可以使用任何[AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html)针对MongoDB执行认证和授权功能了。

Vert.X的默认标准配置（Out Of the Box）中包含了"user"集合（Collection），用户名字段使用"username"进行存储和读取。

为了避免在"user"集合中出现重复的用户名，应该在"user"集合中给"username"添加唯一索引（Unique Index），你可以在MongoDB服务器中运行下边的片段来完成此操作：

    db.user.createIndex( { username: 1 }, { unique: true } )
MongoDB的特性是先查询username字段中的值是否已经存在，然后会插入一个Document，并不能作为一个原子性操作，基于这个原因你需要添加上边的唯一索引，使用了这个索引过后你的代码会尝试先插入一行数据，如果出现了重复记录则会失败。

根据你自己的需要，你同样可以使用下边的方法改变MongoDB中使用的默认集合（Collection）和列（Column）名称等信息：

* [setCollectionName](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setCollectionName-java.lang.String-)
* [setUsernameField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setUsernameField-java.lang.String-)
* [setPasswordField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setPasswordField-java.lang.String-)
* [setPermissionField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setPermissionField-java.lang.String-)
* [setRoleField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setRoleField-java.lang.String-)

Vert.X默认实现中的密码在数据库中使用了SHA-512算法加密后进行存储，之后会连接对应的`salt`值，这个`salt`值和密码存储在同一个表里。你可以调用[setSaltField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setSaltField-java.lang.String-)方法来改变存储`salt`值的字段名称，它的默认值是"salt"，你同样可以使用[setSaltStyle](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashStrategy.html#setSaltStyle-io.vertx.ext.auth.mongo.HashSaltStyle-)来改变一些行为。通过调用[getHashStrategy](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#getHashStrategy--)方法还可以读取Hash策略设置。

MongoDB中可以设置下边几种`SALT`的值（参考下边WARNING）：

* [NO_SALT](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashSaltStyle.html#NO_SALT)：密码就不会执行加密以明文的方式存储。
* [COLUMN](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashSaltStyle.html#COLUMN)：Vert.X会为每个用户创建一个"salt“值，并且存储在用户表定义的列中；
* [EXTERNAL](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashSaltStyle.html#EXTERNAL)：Vert.X仅仅将密码最终加密结果存储在数据库中，"salt“值可以在外部使用，并且可以调用[setExternalSalt](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashStrategy.html#setExternalSalt-java.lang.String-)方法进行设置。

如果你想要重写上述行为，则你可以调用[setHashStrategy](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setHashStrategy-io.vertx.ext.auth.mongo.HashStrategy-)方法设置新的Hash策略，并且提供变更过的Hash策略及配置信息。

__！WARNING__

_强烈建议在设置“salt"值时候使用[EXTERNAL](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/HashSaltStyle.html#EXTERNAL)选项。`NO_SALT`选项仅仅推荐在开发阶段中使用，不仅仅如此，`COLUMN`方式也不推荐，因为它会导致`salt`和密码都存储在了同一个地方！_

## 认证

如果认证使用了默认的MongoDB实现，认证信息中用了`username`和`password`字段：

    JsonObject authInfo = new JsonObject()
        .put("username", "tim")
        .put("password", "sausages");
    authProvider.authenticate(authInfo, res -> {
        if (res.succeeded()) {
            User user = res.result();
        } else {
            // Failed!
        }
    });
如果想要替换上边的`username`和`password`两个默认字段名，你可以使用下边两个方法：

* [setUsernameCredentialField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setUsernameCredentialField-java.lang.String-)
* [setPasswordCredentialField](http://vertx.io/docs/apidocs/io/vertx/ext/auth/mongo/MongoAuth.html#setPasswordCredentialField-java.lang.String-)

## 授权 - Permission/Role模型

尽管Vert.X自身并不要求使用特定的许可模型（它本身只是使用了不透明的字符串），但MongoDB认证中的实现使用了比较熟悉的：用户/角色/许可模型，这样在应用里你可以使用一个或者多个角色，而一个角色也可以拥有一个或者多个许可。

如果要验证一个用户是否拥有特定的许可，则要将许可信息传递到[isAuthorised](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html#isAuthorised-java.lang.String-io.vertx.core.Handler-)中：

    user.isAuthorised("commit_code", res -> {
        if (res.succeeded()) {
            boolean hasPermission = res.result();
        } else {
            // Failed to
        }
    });

如果要验证一个用户是否属于特定角色，则可以使用MongoDB认证前缀（MongoAuth.ROLE_PREFIX）法给角色带上前缀表示：

    user.isAuthorised(MongoAuth.ROLE_PREFIX + "manager", res -> {
        if (res.succeeded()) {
            boolean hasRole = res.result();
        } else {
            // Failed to
        }
    });
