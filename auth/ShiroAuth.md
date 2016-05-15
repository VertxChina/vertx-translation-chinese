# Vert.x Shiro Auth

### Apache Shiro Auth提供者实现

#### 基本术语

* provider：提供者
* authentication：验证
* permissions：权限


这是一个使用[Apache Shiro](http://shiro.apache.org/)的auth提供者的实现。

要使用这个项目，将下面的依赖添加到构建描述符里的dependencies部分。

* Maven（在`pom.xml`文件里）

		<dependency>
			<groupId>io.vertx</groupId>
			<artifactId>vertx-auth-shiro</artifactId>
			<version>3.2.1</version>
		</dependency>
		
* Gradle(在`build.gradle`文件里)

		compile 'io.vertx:vertx-auth-shiro:3.2.1'
		
我们使用Shiro提供基于身份认证属性和LDAP开箱即用的支持，你也可以使用插件，在任何其他的期望用户名和密码作为凭据的Shiro Realm里。

使用`ShiroAuth`创建提供者的实例。使用`ShiroAuthRealmType`指定Shiro auth提供者的类型，并且也可以指定一个JSON对象的配置。

这是通过指定类型创建Shiro auth提供者的示例：

	JsonObject config = new JsonObject().put("properties_path", "classpath:test-auth.properties");
	AuthProvider provider = ShiroAuth.create(vertx, ShiroAuthRealmType.PROPERTIES, config);
	
### 验证

当使用这种实现作为认证时，它需要在认证信息里获取`username`和`password`：

	JsonObject authInfo = new JsonObject().put("username", "tim").put("password", "sausages");
	authProvider.authenticate(authInfo, res -> {
		if (res.succeeded()) {
			User user = res.result();
		} else {
			// Failed!
		}
	});
	
### 授权-权限-角色模型

尽管Vert.x auth本身并没有授权任何特定的许可（它们仅是不透明的字符串）模型，这个的实现和 用户/角色/权限 模型类似，一个用户可以有0到多个角色，一个角色可以有0到多个权限。

要验证一个用户是否有一个特定的权限，只需要简单的将权限传入到`isAuthorised`像接下来这样做：

	user.isAuthorised("newsletter:edit:13", res -> {
		if (res.succeeded()) {
			boolean hasPermission = res.result();
		} else {
			// Failed to
		}
	});
	
要验证一个用户是否有一个特定的角色，你需要在参数前面加上角色前缀。

	user.isAuthorised("role:manager", res -> {
		if (res.succeeded()) {
			boolean hasRole = res.result();
		} else {
			// Failed to
		}
	});
	
默认的角色前缀是`role:`。你可以设置`setRolePrefix`改变默认的。

#### The Shiro properties auth provider

这个auth提供者的实现是使用Apache Shiro从一个配置文件里获取 用户/角色/权限 信息。

注意，角色在API里并不是直接可用的，因为这个事实，vertx-auth尽可能的尝试轻便。然而，可以通过使用前缀`role:`或者通过`setRolePrefix`指定你想要的前缀在一个角色上执行断言。

默认的情况下，这个实现将会在类路径里查找一个名为`vertx-users.properties`的文件。如果你想要改变这个，你可以使用`properties_path`配置元素来定义属性文件的路径，默认的值是`classpath:vertx-users.properties`。

如果这个值得前缀是`classpath:`，将会在类路径查找那个名字的属性文件。如果这个值的前缀是`file:`，将在文件系统上指定一个具体的文件。如果这个值得前缀是`url:`，将会指定一个具体的URL来加载这个属性文件。

这个属性文件应该遵从下面的结构：

每一行应该要么包含一个用户的用户名、密码和角色，要么包含角色的权限。

一个用户的行应该是这样的结构：
user.{username}={password},{roleName1},{roleName2},...,{roleNameN}

一个角色行应该是这样的结构：
role.{roleName}={permissionName1},{permissionName2},...,{permissionNameN}

这是示例：

	user.tim = mypassword,administrator,developer
	user.bob = hispassword,developer
	user.joe = anotherpassword,manager
	role.administrator = *
	role.manager = play_golf,say_buzzwords
	role.developer = do_actual_work

当描述一个角色使用通配符`*`时，说明这个角色拥有所有的权限。

#### The Shiro LDAP auth Provider

LDAP auth realm从一个LDAP服务器上获取 用户/角色/权限 信息。

接下来的这些配置属性是用来配置一个LDAP realm：

* `ldap-user-dn-template`：这是用来决定实际的查找使用当通过一个特定的id来查找一个用户的时候。一个例子是`uid={0},ou=users,dc=foo,dc=com`-这个元素`{0}`是创建实际的查找时替换成用户的id。这个设置是强制的。
* `ldap_url`：这个url是设置LDAP服务器。这个url必须以`ldap://`开头，端口也必须要指定。这是一个示例`ldap://myldapserver.mycompany.com:10389`
* `ldap-authentication-mechanism`：TODO
* `ldap-context-factory-class-name`：TODO
* `ldap-pooling-enabled`：TODO
* `ldap-referral`：TODO
* `ldap-system-username`：TODO
* `ldap-system-password`：TODO

#### 使用另一个Shiro Realm

使用一个预先创建的Apache Shiro Realm对象创建一个auth提供者的示例是可以的。

像下面这样做的：

	AuthProvider provider = ShiroAuth.create(vertx, realm);
	
这个实现当前假定了在基本的验证中使用了用户名和密码。

<a href="mailto:julien@julienviet.com">Julien Viet</a><a href="http://tfox.org">Tim Fox</a>

##### translated by weiyiysw



