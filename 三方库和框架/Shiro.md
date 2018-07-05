# Shiro

## 1、简介

Apache开源的认证、授权、企业会话管理、安全加密等的框架。相比于Spring Security，Shiro具有下面几个优势：
1).使用起来更加简单、灵活；2).可以脱离Spring使用，而Spring Security不可脱离Spring；3).权限管理的粒度比较粗，可以个性化定制。

### 1.1 整体架构

![Shiro的整体架构](res/shiro_framework.png)

以上是Shiro的整体架构，最上面的一层Subject表示当前的用户。Security Manager是Shiro用户管理的核心。Cryptogaphy主要用来对数据进行加密。
在Security Manager部分，Authenticator用来做用户认证，Authorizer用来做授权管理，Session Manager用来做会话管理，它通过Session DAO获取会话信息。
Cache Manager用来做缓存管理，用来缓存用户的身份和认证信息等。
下面的Realms一层类似于一层中间桥梁，用于从数据源中获取授权信息等。

### 1.2 简单的示例

这里我们使用一个简单的测试程序看下Shiro中的各个模块的一些作用。

首先，我们需要搭建一个开发框架，这里我们需要引入两个依赖，一个用来进行单元测试，一个就是Shiro的核心包：

```
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.4.0</version>
    </dependency>
```

然后，我们创建如下的单元测试：

```
public class AuthenticationTest {

    private SimpleAccountRealm realm = new SimpleAccountRealm();

    @Before
    public void prepare() {
        realm.addAccount("WngShhng", "123456");
    }

    @Test
    public void testAuthentication() {
        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        securityManager.setRealm(realm);
        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();
        AuthenticationToken authenticationToken = new UsernamePasswordToken("WngShhng", "123456");
        subject.login(authenticationToken);
        System.out.println("Is authenticated" + subject.isAuthenticated());
    }
}
```

这里我们先创建了一个Realm对象，并在单元测试方法被调用之前向其中添加一些账号信息。
然后，我们获取了一个`SecurityManager`实例并将上面定义的Realm注入到其中。
然后，我们将上面定义的`SecurityManager`通过`SecurityUtils`的`setSecurityManager`方法注入。
之后，我们通过设置完毕的`SecurityUtils`获取一个`Subject`对象。
之后，我们使用用户和密码创建一个Token，并调用上面的`Subject`的`login`方法尝试进行登录。
最终的登录信息存储在subject中，我们可以通过其中的一些方法来获取，比如上面的`isAuthenticated()`就是用来判断用户是否被认证成功的。

## 2、Shiro认证 授权和自定义Realm

上面我们已经见识过了Shiro的Realm的一种简单的使用方式`SimpleAccountRealm`，实际上它还有许多不同的Realm可以供我们使用。
通过上面的代码，我们应该对Realm有一个初步的印象，即不严谨地讲，Realm就是用来从指定的数据源中获取用户的权限、角色等相关的信息的。
它不是数据源而是封装了一些操作指定类型的数据源的方法。下面我们看以下Shiro中各种不同类型的Realm的使用方式。

### 2.1 IniRealm

IniRealm用来从`ini`类型的文件中加载用户的信息，我们先在resources文件夹下面创建一个名为`users.ini`的文件，其中加入下面几行代码：

```
[users]
WngShhng=123456,admin
[roles]
admin=user:delete
```

这里我们按照IniRealm的要求的格式定义了一些用户的信息，然后我们就可以使用该文件中存储的信息进行单元测试：

```
public class IniRealmTest {

    @Test
    public void testWithIniFiles() {
        IniRealm iniRealm = new IniRealm("classpath:users.ini");

        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        securityManager.setRealm(iniRealm);

        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();

        AuthenticationToken authenticationToken = new UsernamePasswordToken("WngShhng", "123456");
        // 校验用户是否已经登录（是否存在该账号的记录）
        subject.login(authenticationToken);
        System.out.println("Is authenticated" + subject.isAuthenticated());
        // 校验用户角色
        subject.checkRole("admin");
        subject.checkPermission("user:delete");
        subject.checkPermission("user:update");
    }
}
```

从这里，我们看出这里的代码和1.2中的代码大部分都一样，只有设置Realm部分，这里是从ini文件中读取的用户信息。
除了用户的身份信息，这里我们还增加了用户的角色和权限等信息的校验。具体该文件中如何配置用户信息可以查看相关的文档。

### 2.2 JdbcRealm

JdbcRealm就是将数据源设置成了数据库，这里我们使用MySQL作为数据源。我们在程序中作如下的配置:

1.首先，我们要使用MySQL连接器，并且使用阿里的Druid的数据连接池：

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```

然后我们进行如下的测试：

```
public class JdbcRealmTest {

    private DruidDataSource druidDataSource = new DruidDataSource();

    {
        druidDataSource.setUrl("jdbc:mysql://localhost:3306/shiro_test?serverTimezone=GMT%2B8");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("xxxxxx");
    }

    @Test
    public void testJdbcRealm() {
        // 查看JdbcRealm的代码里面定义了一些SQL，其实本质上就是通过这些SQL来查询用户信息的
        JdbcRealm jdbcRealm = new JdbcRealm();
        jdbcRealm.setDataSource(druidDataSource);

        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        securityManager.setRealm(jdbcRealm);

        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("WngShhng", "123456");
        subject.login(token);
    }
}

```

从上面的代码可以看出，这里的配置方式和上面的基本一样。只是这里我们将Realm的数据源切换成了MySQL数据库。

在执行上面的单元测试之前，我们还需要在数据库中添加一张表：

```
create table if not exists users (
    username varchar(255),
	password varchar(255)
);
```

然后我们向该数据库中插入一条数据：

```
insert into users("WngShhng", "123456");
```

这样我们执行单元测试就可以通过了。

我们本身没有写什么SQL，在进行校验的时候这些逻辑都被封装进了JdbcRealm里面，你可以进入JdbcRealm查看里面的代码：

```
    protected static final String DEFAULT_AUTHENTICATION_QUERY = "select password from users where username = ?";
    protected static final String DEFAULT_SALTED_AUTHENTICATION_QUERY = "select password, password_salt from users where username = ?";
    protected static final String DEFAULT_USER_ROLES_QUERY = "select role_name from user_roles where username = ?";
    protected static final String DEFAULT_PERMISSIONS_QUERY = "select permission from roles_permissions where role_name = ?";
	......
```

所以，当我们不去设置SQL等的信息的时候，默认会去执行上面的SQL，而我们创建表的时候的SQL也是从上面的几行SQL中总结出来的。

如果要使用权限的信息还要设置JdbcRealm的开关为打开状态：

```
jdbcRealm.setPermissionsLookupEnabled(true)
```

上面我们使用的Shiro为我们提供的默认的SQL来进行数据查询的，我们也可以自定义一些SQL来进行查询，比如

```
jdbcRealm.setUserRolesQuery("select ....");
```

JdbcRealm当中还有许多类似的方法可供我们使用，我们可以将这些SQL注入到JdbcRealm当中，然后Shiro就会使用我们指定的SQL进行数据查询。

### 2.3 自定义Realm

除了使用Shiro为我们提供的Realm，我们还可以自定义Realm，我们可以使用自定义Realm来集成其他的框架，比如使用一些缓存来作为数据源等，
这极大地提升了我们设置数据源的灵活性。这里我们通过一个简单的例子来演示一下Shiro中自定义Realm的内容。

通过查看JdbcRealm的代码，我们知道它是继承自AuthorizingRealm，所以在自定义Realm的时候，我们也从该类进行拓展。

```
public class CustomRealm extends AuthorizingRealm {

    // 用来模拟用户信息数据源
    private Map<String, String> maps = new HashMap<>();

    {
        maps.put("WngShhng", "123456");
        super.setName("customRealm");
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String userName = (String) authenticationToken.getPrincipal();
        String password = getPasswordByUyUserName(userName);

        // 通过将包装之后的AuthenticationInfo返回，用于检查用户的验证信息
        return new SimpleAuthenticationInfo(userName, password, "customRealm");
    }

    private String getPasswordByUyUserName(String userName) {
        return maps.get(userName);
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();

        // 这里用来模拟从指定数据源中获取指定用户的角色和权限信息
        Set<String> roles = getRolesByUserName(userName);
        Set<String> permissions = getPermissionByUserName(userName);

        // 授权信息，将包装之后的AuthorizationInfo返回，内部设置了该用户的权限和角色信息
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setRoles(roles);
        authorizationInfo.setStringPermissions(permissions);

        return authorizationInfo;
    }

    // 模拟获取指定用户的角色
    private Set<String> getRolesByUserName(String userName) {
        Set<String> roles = new HashSet<>();
        roles.add("Admin");
        roles.add("User");
        return roles;
    }

    // 模拟获取指定用户的权限
    private Set<String> getPermissionByUserName(String userName) {
        Set<String> permissions = new HashSet<>();
        permissions.add("do-1");
        permissions.add("do-2");
        permissions.add("do-3");
        return permissions;
    }
}
```

AuthorizingRealm的两个方法的作用不难理解，它们用来根据指定的用户名来获取该用户的密码、角色和权限等，然后将获取到的结果包装到一个对象中返回。
随后，Shiro会根据我们返回的对象来判断该用户是否具有指定的角色和权限等信息。

根据上面的定义，我们进行如下的单元测试：

```
public class CustomRealmTest {

    @Test
    public void testCustomRealm() {
        CustomRealm customRealm = new CustomRealm();

        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        securityManager.setRealm(customRealm);

        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();

        AuthenticationToken authenticationToken = new UsernamePasswordToken("WngShhng", "123456");
        // 校验用户是否已经登录（是否存在该账号的记录）
        subject.login(authenticationToken);
        System.out.println("Is authenticated" + subject.isAuthenticated());
        // 校验用户角色
        subject.checkRoles("Admin", "User");

        subject.checkPermissions("do-1", "do-2");
    }
}
```

从上面的代码我们可以看出，自定义Realm的时候主要是根据传入的用户信息在覆写的两个方法中进行查询并返回授权和验证信息，
关键的一步在于从某个数据源中取用户的信息。所以，通过自定义Realm，我们可以很方便地使用各种数据源与Shiro集成。



