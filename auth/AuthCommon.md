# Vert.x Auth Common

* 引用：[http://vertx.io/docs/vertx-auth-common/java/](http://vertx.io/docs/vertx-auth-common/java/)

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

<hr>

Vert.X Auth项目提供了处理认证和授权的功能，它可以被vertx-web项目使用，若要在自己的项目中运用它，则需要在构建描述信息的_dependencies_节点中添加如下信息：

* Maven（在`pom.xml`文件中）

        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-auth-common</artifactId>
            <version>3.2.1</version>
        </dependency>
* Gradle（在`build.gradle`文件中）

        compile io.vertx:vertx-auth-common:3.2.1

## 基本概念

认证_Authentication_：用于验证用户的标识和身份。

授权_Authorisation_：用于验证用户是否拥有访问系统的许可。

权限_Authority_：它取决于一些特定的系统实现但对特定的模型不做任何要求；比如：许可/角色`permissions/roles`模型，可以使事情变得灵活。在一些实现中一个权限可以用来表述一个许可，如：有权限访问所有打印机、或特定打印机。在另外一些实现中则必须支持角色信息，通常使用一些角色信息对权限进行前缀`role:`命名/标识，如：`role:admin`。还有一些实现也许会包含更加复杂以及不同的模型来表述权限信息。

如果要了解期望的特定认证提供者_Auth Provider_，则可以按照文档中的内容实施。

## 认证

如果要对用户进行认证可使用`Provider`中的[authenticate](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html#authenticate-io.vertx.core.json.JsonObject-io.vertx.core.Handler-)方法。

这个方法的第一个参数是一个`JSON`对象，它包含了认证用的信息，实际上这些信息取决于特定的实现；对于简单基于用户名/密码`username/password`的认证包含了如下信息：

    {
        “username": "tim",
        "password": "mypassword"
    }
对于一些基于`JWT Token`或`OAuth Bearer Token`的实现还会在认证信息中包含令牌_Token_信息。Vert.X中的认证是异步执行，在调用过程中，认证结果被传给结果处理器中的用户`User`对象里，异步调用结果包含了一个用户[User](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html)的实例，这个实例表述了已经认证过的用户信息，并且包含了这些用户允许执行的合法操作。

这里是一个简单的基于用户名/密码`username/password`认证用户的代码实现：

    JsonObject authInfo = new JsonObject().put("username", "tim").put("password", "mypassword");

        authProvider.authenticate(authInfo, res -> {
            if (res.succeeded()) {

                User user = res.result();
        
                System.out.println("User " + user.principal() + " is now authenticated");
            } else {
                res.cause().printStackTrace();
            }
    });

## 授权

一旦你拥有了一个用户[User](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html)的实例过后，你就可以调用它的方法对这个用户进行授权。

检查一个用户是否拥有特定权限需要使用它的[`isAuhorised`](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html#isAuthorised-java.lang.String-io.vertx.core.Handler-)方法。

上边所有操作的结果都是通过处理器`Handler`异步调用提供的。

这里是一个对用户授权的例子：

    user.isAuthorised("printers:printer1234", res -> {
        if (res.succeeded()) {

            boolean hasAuthority = res.result();

            if (hasAuthority) {
                System.out.println("User has the authority");
            } else {
                System.out.println("User does not have the authority");
            }
        } else {
            res.cause().printStackTrace();
        }
    });
另外一个例子对基于角色模型的用户进行授权就是使用`role:`前缀，请注意，就像上边讨论的一样，权限字符串如何被解释完全取决于底层实现，这里Vert.X不对解释细节提供假设。

### 权限缓存

用户对象可以缓存任何权限，之后的调用会检查是否拥有同样的权限执行其操作，这个结果实际上调用了底层提供者`Provider`的方法。

为了清空内部缓存，则你可以使用[clearCache](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html#clearCache--)方法。

### 用户凭证

对于已经认证过的用户，你可以调用[principal](http://vertx.io/docs/apidocs/io/vertx/ext/auth/User.html#principal--)方法获取用户的凭证信息，获取凭证信息的内容同样取决于底层实现。

## 创建自己的认证实现

如果你希望创建自己的认证提供者`Auth Provider`则需要创建一个类实现[AuthProvider](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AuthProvider.html)接口。

Vert.X提供了用户对象的抽象实现[AbstractUser](http://vertx.io/docs/apidocs/io/vertx/ext/auth/AbstractUser.html)，你可以创建这个类的子类对自己的用户`User`对象提供自定义实现。这个抽象实现中包含了缓存逻辑，所以你在实现的时候不需要考虑自己处理缓存问题。

如果你希望在集群环境中使用你的`User`对象，则需要保证这个用户对象实现了[ClusterSerializable](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/impl/ClusterSerializable.html)接口。

