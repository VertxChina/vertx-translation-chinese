# Vert.x OAuth 2

* 引用：[http://vertx.io/docs/vertx-auth-oauth2/java/](http://vertx.io/docs/vertx-auth-oauth2/java/)

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
* __Credential__：证书
* __Token__：令牌

Vert.X的这个组件包含了标准的OAuth2的实现。若要在自己的项目中使用它，则需要在构建描述信息的_dependencies_节点中添加如下信息：

* Maven（在`pom.xml`文件中）

        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-auth-oauth2</artifactId>
            <version>3.2.1</version>
        </dependency>
* Gradle（在`build.gradle`文件中）

        compile io.vertx:vertx-auth-oauth2:3.2.1

如果用户以第三方应用的方式访问想要的资源，OAuth2可以对这些用户进行授权，任何时候可以根据用户想要都有可能启用或禁用它的访问权限。

Vert.X中的OAuth2支持下边三种流程：

* Authorization Code：授权码流程（对服务器和App可持久化存储信息）
* Password Credentials：密码证书流程（之前的流程无法使用或开发阶段使用）
* Client Credentials：客户端证书流程（客户端可仅仅可凭借客户端整数申请访问令牌【Access Token】）

## 授权码——Authorization Code Flow

授权码授权类型可以用来获取访问令牌（Access Token）和刷新令牌（Refresh Token），对安全性要求高的客户端（Confidential Client）是很不错的（Optimized）一种方式。作为一个基于重定向的流程，客户端必须能和资源拥有者的用户代理交互（通常是浏览器），同时要能接受从授权服务器（Authorization Server）通过重定向发送过来的请求。

更多：[OAuth2 Section 4.1](http://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.1)

## 密码证书——Password Credentials Flow

资源拥有者密码证书授权类型比较适合于这样一种情况：当资源拥有者和客户端之间有可信任的关系时使用，如设备操作系统、或高特权应用。授权服务器应该在很特殊的情况时启用这种授权类型，并且仅仅当其他应用都不可见时允许使用这种类型。

这种类型对于客户端要获取资源拥有者证书（用户名和密码，通常使用交互式表单）是合适的。它用于从已经存在的直接使用认证模式的如Basic或Digest的客户端向OAuth2认证方式迁移的时候，它会将证书直接转换成OAuth2中的访问令牌。

更多：[OAuth2 Section 4.3](http://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.3)

## 客户端证书——Client Credentials Flow

当客户端想要申请访问控制之下受保护的资源时，或者其他资源拥有者先前被授权服务器托管时（这种方式超出了本文范畴），客户端仅仅可以使用客户端证书（或者其他表示认证含义的信息）申请访问令牌。

客户端证书类型必须用于且仅用于机密客户端。

更多：[OAuth2 Section 4.4](http://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.4)

## 初探

下边是基于GitHub中使用Vert.X的OAuth2 Provider的认证示例实现代码：

    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.AUTH_CODE, new JsonObject()
        .put("clientID", "YOUR_CLIENT_ID")
        .put("clientSecret", "YOUR_CLIENT_SECRET")
        .put("site", "https://github.com/login")
        .put("tokenPath", "/oauth/access_token")
        .put("authorizationPath", "/oauth/authorize")
    );

    // when there is a need to access a protected resource or call a protected method,
    // call the authZ url for a challenge

    String authorization_uri = oauth2.authorizeURL(new JsonObject()
        .put("redirect_uri", "http://localhost:8080/callback")
        .put("scope", "notifications")
        .put("state", "3(#0/!~"));

    // when working with web application use the above string as a redirect url

    // in this case GitHub will call you back in the callback uri one should now complete the handshake as:


    String code = "xxxxxxxxxxxxxxxxxxxxxxxx"; // the code is provided as a url parameter by github callback call

    oauth2.getToken(new JsonObject().put("code", code).put("redirect_uri", "http://localhost:8080/callback"), res -> {
        if (res.failed()) {
            // error, the code provided is not valid
        } else {
            // save the token and continue...
        }
    });

## 授权码流程

授权码流程主要包含两部分内容：

1. 第一步，你的应用客户端向用户申请允许访问它们的数据，如果用户审批后，OAuth2服务器发送给客户端一个授权码；
2. 第二步，客户端将这个授权码和客户端密钥放到POST请求中发送给授权服务器（Authority Server）得到访问令牌；

示例代码：

    JsonObject credentials = new JsonObject()
        .put("clientID", "<client-id>")
        .put("clientSecret", "<client-secret>")
        .put("site", "https://api.oauth.com");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.AUTH_CODE, credentials);

    // Authorization oauth2 URI
    String authorization_uri = oauth2.authorizeURL(new JsonObject()
        .put("redirect_uri", "http://localhost:8080/callback")
        .put("scope", "<scope>")
        .put("state", "<state>"));

    // Redirect example using Vert.x
    response.putHeader("Location", authorization_uri)
        .setStatusCode(302)
        .end();

    JsonObject tokenConfig = new JsonObject()
        .put("code", "<code>")
        .put("redirect_uri", "http://localhost:3000/callback");

    // Callbacks
    // Save the access token
    oauth2.getToken(tokenConfig, res -> {
        if (res.failed()) {
            System.err.println("Access Token Error: " + res.cause().getMessage());
        } else {
            // Get the access token object (the authorization code is given from the previous step).
            AccessToken token = res.result();
        }
    });

## 密码证书流程

资源拥有者密码证书授权类型比较适合于这样一种情况：当资源拥有者和客户端之间有可信任的关系时使用，如设备操作系统、或高特权应用。授权服务器应该在很特殊的情况时启用这种授权类型，并且仅仅当其他应用都不可见时允许使用这种类型。

    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.PASSWORD);

    JsonObject tokenConfig = new JsonObject()
        .put("username", "username")
        .put("password", "password");

    // Callbacks
    // Save the access token
    oauth2.getToken(tokenConfig, res -> {
        if (res.failed()) {
            System.err.println("Access Token Error: " + res.cause().getMessage());
        } else {
            // Get the access token object (the authorization code is given from the previous step).
            AccessToken token = res.result();

            oauth2.api(HttpMethod.GET, "/users", new JsonObject().put("access_token", token.principal().getString("access_token")), res2 -> {
                // the user object should be returned here...
            });
        }
    });

## 客户端证书流程

这种类型对于客户端要获取资源拥有者证书是合适的。

    JsonObject credentials = new JsonObject()
        .put("clientID", "<client-id>")
        .put("clientSecret", "<client-secret>")
        .put("site", "https://api.oauth.com");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

    JsonObject tokenConfig = new JsonObject();

    // Callbacks
    // Save the access token
    oauth2.getToken(tokenConfig, res -> {
        if (res.failed()) {
            System.err.println("Access Token Error: " + res.cause().getMessage());
        } else {
            // Get the access token object (the authorization code is given from the previous step).
            AccessToken token = res.result();
        }
    });

## 访问令牌对象

当一个令牌过期后你需要刷新令牌，OAuth2提供了访问令牌类AccessToken，它包含了很多实用的方法可以在令牌过期过后对令牌进行刷新。

    if (token.expired()) {
        // Callbacks
        token.refresh(res -> {
            if (res.succeeded()) {
                // success
            } else {
                // error handling...
            }
        });
    }

当你使用令牌访问完成过后想要注销，你可以撤销（Revoke）访问令牌和刷新令牌。

    token.revoke("access_token", res -> {
        // Session ended. But the refresh_token is still valid.

        // Revoke the refresh_token
        token.revoke("refresh_token", res1 -> {
            System.out.println("token revoked.");
        });
    });

## 其他通用OAuth2的Provider配置示例

### Google

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://accounts.google.com")
        .put("tokenPath", "https://www.googleapis.com/oauth2/v3/token")
        .put("authorizationPath", "/o/oauth2/auth");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

### GitHub

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://github.com/login")
        .put("tokenPath", "/oauth/access_token")
        .put("authorizationPath", "/oauth/authorize");
    
    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

### Linkedin

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://www.linkedin.com")
        .put("authorizationPath", "/uas/oauth2/authorization")
        .put("tokenPath", "/uas/oauth2/accessToken");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

### Twitter

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://api.twitter.com")
        .put("authorizationPath", "/oauth/authorize")
        .put("tokenPath", "/oauth/access_token");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

### Facebook

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://www.facebook.com")
        .put("authorizationPath", "/dialog/oauth")
        .put("tokenPath", "https://graph.facebook.com/oauth/access_token");

    
    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);

### JBoss Keycloak

    JsonObject credentials = new JsonObject()
        .put("clientID", "CLIENT_ID")
        .put("clientSecret", "CLIENT_SECRET")
        .put("site", "https://www.your-keycloak-server.com")
        .put("authorizationPath", "/realms/" + realm + "/protocol/openid-connect/auth")
        .put("tokenPath", "/realms/" + realm + "/protocol/openid-connect/token");

    // Initialize the OAuth2 Library
    OAuth2Auth oauth2 = OAuth2Auth.create(vertx, OAuth2FlowType.CLIENT, credentials);