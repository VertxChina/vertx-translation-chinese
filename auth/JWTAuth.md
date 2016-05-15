## JWT auth

* 引用：[http://vertx.io/docs/vertx-auth-jwt/java/](http://vertx.io/docs/vertx-auth-jwt/java/)

### 基本术语
* Authentication ：认证
* Authorisation：授权
* Authenticity：真实性
* Permission： 许可
* Token： 令牌
* Provider：提供者


使用JSON web tokens实现Auth。

### The JWT auth Provider

这个组件包含了一个开箱即用的JWT实现。

要使用这个项目，将下面的依赖添加到构建描述符里的`dependencies`部分。

* Maven（在你的`pom.xml`文件里）

		<dependency>
			<groupId>io.vertx</groupId>
			<artifactId>vertx-auth-jwt</artifactId>
			<version>3.2.1</version>
		</dependency>
		 
* Gradle(在你的`build.gradle`文件里)

		compile 'io.vertx:vertx-auth-jwt:3.2.1'
		
JSON Web 令牌是一种简单的方法来发送明文信息（通常是URL），其内容可以被验证为是可信的。像下面的这些场景JWT是非常适用的：

* 在一个单一的报名场景中，你想要有一个隔离的认证服务用可被信任的方式发送用户信息
* 无状态的Server API，非常适用于简单的页面应用
* 等等

在决定使用JWT之前，需要重点注意的是JWT并不加密payload，它仅对payload签名。你不应该使用JWT发送任何私密信息，相反你应该发送是不是私密的但要被验证的信息。举个例子，使用JWT发送一个签名过的用户id来表明这个用户已经登录了的做法非常对的。相反发送一个用户的密码的做法是非常非常错误的。

JWT主要的优点有：

* 它可以让你验证令牌的真实性
* 它有一个JSON结构，可以包含任何你想要的变量和大量的数据
* 它完全是无状态的

你可以使用`JWTAuth`来创建一个提供者的实例，并指定一个JSON对象的配置。这是创建一个JWT auth提供者的示例代码：

	JsonObject config = new JsonObject().put("keyStore", new JsonObject()
		.put("path", keystore.jceks)
		.put("type", "jceks")
		.put("password", "secret"));
		
	AuthProvider provider = JWTAuth.create(vertx, config);
	
JWT用法的典型流程是，在您的应用程序有一个终点颁发令牌，这个终点应在SSL模式下运行，终点这通过用户名和密码验证完请求用户之后，表示你将这样做生成令牌：

	JsonObjetc config = new JsonObject().put("keyStore", new JsonObject()
		.put("path", "keystore.jceks")
		.put("type", "jceks")
		.put("password", "secret"));
		
	JWTAuth provider = JWTAuth.create(vertx, config);
	
	// 在验证的终点上，一旦你通过它的用户名/密码验证了用户的id
	if ("paulo".equal(username) && "super_secret".equals(password)) {
		String token = provider.generateToken(new JsonObject().put("sub", "paulo"), new JWTOptions());
		// 这时任何请求受保护的资源你应该传入这个字符串到http头授权部分，像这样：
		// 授权：Bearer <token>
	}
	
TODO展示了使用JWT的验证和授权的例子并解释了如何通过传入到授权的许可字符映射到JW令牌中的声明。

### THE JWT keystore file

auth提供者需要在类路径或文件系统上加载一个keystore文件，keystore文件要么有一个`mac`要么有一个`Signature`用于签名和验证生成的令牌，。

默认的情况下，实践将会查找这些别名，然而并不是所有的算法都需要存在。作为一个好的实践`HS256`应该存在：

	`HS256`:: HMAC using SHA-256 hash algorithm
	`HS384`:: HMAC using SHA-384 hash algorithm
	`HS512`:: HMAC using SHA-512 hash algorithm
	`RS256`:: RSASSA using SHA-256 hash algorithm
	`RS384`:: RSASSA using SHA-384 hash algorithm
	`RS512`:: RSASSA using SHA-512 hash algorithm
	`ES256`:: ECDSA using P-256 curve and SHA-256 hash algorithm
	`ES384`:: ECDSA using P-384 curve and SHA-384 hash algorithm
	`ES512`:: ECDSA using P-521 curve and SHA-512 hash algorithm
	
当keystore没有被提供，执行将会回滚到一个不安全的模式同时签名并不会被验证，如果payload被外部的方法签名或加密后的情况下这是非常有用的。

### 生成一个新的Keystore文件

生成keystore文件唯一需要的工具是`keytool`，运行的时候，你可以指定你需要使用的算法：

	keytool -genseckey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg HMacSHA256 -keysize 2048 -alias HS256 -keypass secret
	keytool -genseckey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg HMacSHA384 -keysize 2048 -alias HS384 -keypass secret
	keytool -genseckey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg HMacSHA512 -keysize 2048 -alias HS512 -keypass secret
	keytool -genkey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg RSA -keysize 2048 -alias RS256 -keypass secret -sigalg SHA256withRSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	keytool -genkey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg RSA -keysize 2048 -alias RS384 -keypass secret -sigalg SHA384withRSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	keytool -genkey -keystore keystore.jceks -storetype jceks -storepass secret -keyalg RSA -keysize 2048 -alias RS512 -keypass secret -sigalg SHA512withRSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	keytool -genkeypair -keystore keystore.jceks -storetype jceks -storepass secret -keyalg EC -keysize 256 -alias ES256 -keypass secret -sigalg SHA256withECDSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	keytool -genkeypair -keystore keystore.jceks -storetype jceks -storepass secret -keyalg EC -keysize 256 -alias ES384 -keypass secret -sigalg SHA384withECDSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	keytool -genkeypair -keystore keystore.jceks -storetype jceks -storepass secret -keyalg EC -keysize 256 -alias ES512 -keypass secret -sigalg SHA512withECDSA -dname "CN=,OU=,O=,L=,ST=,C=" -validity 360
	
<a href="mailto:julien@julienviet.com">Julien Viet</a><a href="http://tfox.org">Tim Fox</a><a href="mailto:pmlopes@gmail.com">Paulo Lopes</a>


##### translated by weiyiysw