# Vert.x JDBC Auth

* 引用：[http://vertx.io/docs/vertx-auth-jdbc/java/](http://vertx.io/docs/vertx-auth-jdbc/java/)

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

Vert.X中提供了一个使用[JDBCClient](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html)的[AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html)实现，它针对任何兼容JDBC的关系数据库执行认证和授权。若要在自己的项目中使用它，则需要在构建描述信息的_dependencies_节点中添加如下信息：

* Maven（在`pom.xml`文件中）

        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-auth-jdbc</artifactId>
            <version>3.2.1</version>
        </dependency>
* Gradle（在`build.gradle`文件中）

        compile io.vertx:vertx-auth-jdbc:3.2.1

如果要创建一个客户端实例，你首先需要一个[JDBCClient](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html)的实例，要知道如何创建这个实例可按照文档中的内容实施。

一旦你创建了一个[JDBCClient](http://vertx.io/docs/apidocs/io/vertx/ext/jdbc/JDBCClient.html)实例后，就可以按下边代码创建[JDBCAuth](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html)实例：

    JDBCClient jdbcClient = JDBCClient.createShared(vertx, jdbcClientConfig);

    JDBCAuth authProvider = JDBCAuth.create(jdbcClient);

创建好上边的实例过后你就可以如使用任何[AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html)执行认证和授权功能了。

Vert.X的默认标准配置（Out Of the Box）中包含了某些针对认证和授权的信息查询，如果你想要使用不同的数据库模式（Schema），这些查询内容可以通过下边几个方法进行更改：

* [setAuthenticationQuery](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html#setAuthenticationQuery-java.lang.String-)
* [setPermissionQuery](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html#setPermissionsQuery-java.lang.String-)
* [setRolesQuery](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html#setRolesQuery-java.lang.String-)

Vert.X默认实现中的密码在数据库中使用了SHA-512算法加密后进行存储，之后会连接对应的`salt`值，这个`salt`值和密码存储在同一个表里。

如果你想要重写这些行为，则可以重写[setHashStrategy](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html#setHashStrategy-io.vertx.ext.auth.jdbc.JDBCHashStrategy-)方法去修改Hash策略的设置。

__！WARNING__

_强烈建议在存储密码时使用哈希算法加密过后保存在数据库中，这个哈希值是在创建这一行记录时基于`salt`值计算的，应用中应该使用强壮的密码算法，在存储密码时绝对不要使用明文。_

## 认证

如果要使用默认的认证实现，认证信息中用了`username`和`password`字段进行表述：

    JsonObject authInfo = new JsonObject().put("username", "tim").put("password", "sausages");

    authProvider.authenticate(authInfo, res -> {
        if (res.succeeded()) {
            User user = res.result();
        } else {
            // Failed!
        }
    });

## 授权 - Permission/Role模型

尽管Vert.X Auth自身并不要求使用特定的许可模型（它本身只是使用了不透明的字符串），但默认的实现使用了比较熟悉的：用户/角色/许可模型，这样在应用里你可以使用一个或者多个角色，而一个角色也可以拥有一个或者多个许可。

如果要验证一个用户是否拥有特定的许可，则需要将许可信息传递到[isAuthorised](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html#isAuthorised-java.lang.String-io.vertx.core.Handler-)中：

    user.isAuthorised("commit_code", res -> {
        if (res.succeeded()) {
            boolean hasPermission = res.result();
        } else {
            // Failed to
        }
    });

如果要验证一个用户是否属于特定角色，则可以使用前缀法给角色带上前缀表示：

    user.isAuthorised("role:manager", res -> {
        if (res.succeeded()) {
            boolean hasRole = res.result();
        } else {
            // Failed to
        }
    });

Vert.X中的默认角色前缀使用了`role:`，这个值可通过[setRolePrefix](http://vertx.io/docs/apidocs/io/vertx/ext/auth/jdbc/JDBCAuth.html#setRolePrefix-java.lang.String-)进行更改。


