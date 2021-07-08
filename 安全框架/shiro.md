# shiro

### Maven

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.3.2</version>
</dependency>
```
### 主要对象

在shiro中主要有三个对象：

* Subject
  * 我们需要操作的类，代表当前用户，除了加密外的功能外，其他认证，权限控制等都通过操作`Subject`完成。其内部是调用`SucurityManager`的方法来完成操作。

```java
//获取Subject
Subject currentUser = SecurityUtils.getSubject();
```

* SecurityManager
  * shiro框架的核心类，`Subject` 代表了当前用户的安全操作，`SecurityManager` 则管理**所有**用户的安全操作。`Subject`调用`SecurityManager`的方法来完成操作。

* Realm
  * 实质上是一个dao，`SecurityManager`调用它来加载用户、角色、权限。若配置文件`ini`，默认有IniRealm加载，而在项目中一般我们自定义`Realm`读取数据库获取用户的角色，权限等。自定义Realm用于查询用户的角色和权限信息并保存到权限管理器

### 基本配置

##### 自定义CustomRealm类

> 自定义的realm一般继承`AuthorizingRealm`并实现登录验证和赋予角色权限的两个方法。
>
> doGetAuthorizationInfo：权限管理。
>
> doGetAuthenticationInfo：身份认证（登录）。

```java
//自定义realm，用于获取认证信息和权限信息
public class MyShiroRealm extends AuthorizingRealm {
    
    //注入service，以便查询数据库
    @AutoWired
    UserService userService；
    

    /* 
     * 授权操作
     * 获取当前用户的角色和权限
     * 将角色和权限封装返回
     */
    @Override
    public AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) { 
        // 获取当前用户
        String username = (String) principals. getPrimaryPrincipal();
        
        /* 
         * 通过用户名获取用户角色及用户权限
         * service用set集合，可以去重
         * 且SimpleAuthorizationInfo需要传递set集合
         */
        Set<String> roleNames = userService.findRoleByUsername(username);  
        Set<String> permissions = userService.findPermissionseByUsername(username);  
       
        // 添加角色信息
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo(roleNames);  
        // 添加权限信息
        info.setStringPermissions(permissions);  
        return info;  
    }

    /* 
     * 登录验证
     * SecurityManager拦截请求，获取页面输入的用户名和密码
     * 将用户名和密码封装到UsernamePasswordToken中
     * 传递给doGetAuthenticationInfo方法
     */
    @Override
    public AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken authcToken) throws AuthenticationException {
       	//获取用户名
        String useranme = (String)token.getPrincipal();
        //该用户名对应的用户
        User user = userService.findAllUserInfoByUsername(username);

        //取密码
        String password = user.getPassword();
        
        //如果没有找到用户，会抛出UnknownAccountException异常
        if (user == null) {
            return null;
        }

        //其为返回值的实现类，最后一个参数是该realm的标识，因为可能会有多个realm
        return new SimpleAuthenticationInfo(username, password, this.getClass().getName());
    }

}
```

##### ShiroConfig配置

```java
@Configuration
public class ShiroConfig {

    // 注册自定义过滤器,将配置的securityManager传入
    @Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean shiroFilter(@Qualifier(name="securityManager")SecurityManager securityManager) {
        
        //配置securityManager
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        
        //当需要认证时要跳转的页面的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        //设置权限不足跳转的url
        shiroFilterFactoryBean.setUnauthorizedUrl("/401");
        
        // 配置过滤器链
        // 认证过滤器 anno、authc、
        // 授权过滤器 authcBasic、logout、noSessionCreation、perms、port、rest、roles、ssl、user过滤器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        //authc:所有url都必须认证通过才可以访问; 
        //anon:所有url都都可以匿名访问
        
        //表示以下路径可以直接访问
        filterChainDefinitionMap.put("/webjars/**", "anon");
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/", "anon");
        filterChainDefinitionMap.put("/front/**", "anon");
        filterChainDefinitionMap.put("/api/**", "anon");
        filterChainDefinitionMap.put("/api/**", "anon");
        

        // 表示admin/*路径下只有admin角色可以访问
        filterChainDefinitionMap.put("/admin/**", "role:admin");
        filterChainDefinitionMap.put("/user/**", "authc");
        // 主要这行代码必须放在所有权限设置的最后，不然会导致所有 url 都被拦截 剩余的都需要认证
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;

    }

    // 注册安全管理器，将自定义realm传递到内部使用，一般为固定写法
    @Bean
    public SecurityManager securityManager(@Qualifier(name="MyShiroRealm") MyShiroRealm realm)) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        defaultSecurityManager.setRealm(realm);
        return defaultSecurityManager;
    }

    // 将上面自定义realm添加到容器中
    @Bean
    public MyShiroRealm myShiroRealm() {
        CustomRealm MyShiroRealm = new MyShiroRealm();
        return MyShiroRealm;
    }
}
```

* shiroFilter方法：

配置shiro的过滤器，可以设置登录页面（setLoginUrl）、权限不足跳转页面（setUnauthorizedUrl）、具体某些页面的权限控制或者身份认证。

* securityManager 方法：

securityManager是一个接口类，web项目，使用的DefaultWebSecurityManager 实现类，然后设置自己的Realm。

##### controller类

```java
@PostMapping(value = "/login")
@ResponseBody
public String login(@RequestParam("username") String username, @RequestParam("password") String password) {
    
    //SecurityUtils创建subject
    Subject subject = SecurityUtils.getSubject();
    
    //准备 token（令牌）
    UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    
    // 执行认证登陆
    try {
        subject.login(token);
    } catch (UnknownAccountException uae) {
        return "未知账户";
    } catch (IncorrectCredentialsException ice) {
        return "密码不正确";
    } catch (LockedAccountException lae) {
        return "账户已锁定";
    } catch (ExcessiveAttemptsException eae) {
        return "用户名或密码错误次数过多";
    } catch (AuthenticationException ae) {
        return "用户名或密码不正确！";
    }
    if (subject.isAuthenticated()) {
        return "登录成功";
    } else {
        token.clear();
        return "登录失败";
    }
}
```
##### 利用注解配置权限

其实，我们完全可以不用注解的形式去配置权限，因为在之前已经加过了：DefaultFilter类中有perms（类似于perms[user:add]）这种形式的。但是试想一下，这种控制的粒度可能会很细，具体到某一个类中的方法，那么如果是配置文件配，是不是每个方法都要加一个perms？但是注解就不一样了，直接写在方法上面，简单快捷。
很简单，只需要在config类中加入如下代码，就能开启注解：

```java
@Bean
public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
    return new LifecycleBeanPostProcessor();
}

/*
 * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions)
 * 需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
 * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)
 * AuthorizationAttributeSourceAdvisor)即可实现此功能
 */
@Bean
@DependsOn({"lifecycleBeanPostProcessor"})
public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
    DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
    advisorAutoProxyCreator.setProxyTargetClass(true);
    return advisorAutoProxyCreator;
}

@Bean
public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
    AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
    authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
    return authorizationAttributeSourceAdvisor;
}
```


使用注解配置
```java
@RequestMapping("/user")
@RestController
public class UserController {
    
    @RequiresPermissions("user:list")
    @RequestMapping("/show")
    public String showUser() {
        return "这是学生信息";
    }
}
```
常用注解

```
@RequiresAuthentication：表示当前Subject已经认证才可以调用该方法

@RequiresUser：表示当前Subject使用rememberme功能登录可以使用

@RequiresGuest：表示当前Subject没有身份验证或通过记住我登录过，即是游客身份可以访问此方法。

@RequiresRoles（value={"admin"，"user"），logical=
Logical.AND）：表示当前 Subject 需要角色admin以及user才能访问此方法

@RequiresPermissions（value={"user:a"，"user:b"}，logical=Logical.OR）：表示当前Subject 需要权限user:a或user:b。
```

### 加密

其实上面的功能已经基本满足我们的需求了，但是唯一一点美中不足的是，密码都是采用的明文方式进行比对的。那么shiro是否提供给我们一种密码加密的方式呢？答案是肯定。
	shiroConfig中加入加密配置：

```java
@Bean(name = "credentialsMatcher")
public HashedCredentialsMatcher hashedCredentialsMatcher() {
    HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
    // 散列算法:这里使用MD5算法;
    hashedCredentialsMatcher.setHashAlgorithmName("md5");
    // 散列的次数，比如散列两次，相当于 md5(md5(""));
    hashedCredentialsMatcher.setHashIterations(2);
    // storedCredentialsHexEncoded默认是true，此时用的是密码加密用的是Hex编码；false时用Base64编码
    hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
    return hashedCredentialsMatcher;
}
```

customRealm初始化的时候耶需要做一些改变：

```java
@Bean
public CustomRealm customRealm() {
    CustomRealm customRealm = new CustomRealm();
    // 告诉realm,使用credentialsMatcher加密算法类来验证密文
    customRealm.setCredentialsMatcher(hashedCredentialsMatcher());
    customRealm.setCachingEnabled(false);
    return customRealm;
}
```

用户注册的时候，程序将明文通过加密方式加密，存到数据库的是密文，登录时将密文取出来，再通过shiro将用户输入的密码进行加密对比，一样则成功，不一样则失败。
shiro提供了SimpleHash类帮助我们快速加密：

```java
public static String MD5Pwd(String username, String pwd) {
    // 加密算法MD5
    // salt盐 username + salt
    // 迭代次数
    String md5Pwd = new SimpleHash("MD5", pwd,
          ByteSource.Util.bytes(username + "salt"), 2).toHex();
    return md5Pwd;
}
```

也就是说注册的时候调用一下上面的方法得到密文之后，再存入数据库。
在CustomRealm进行身份认证的时候我们也需要作出改变：

```java
System.out.println("-------身份认证方法--------");
String userName = (String) authenticationToken.getPrincipal();
String userPwd = new String((char[]) authenticationToken.getCredentials());
//根据用户名从数据库获取密码
String password = "2415b95d3203ac901e287b76fcef640b";
if (userName == null) {
    throw new AccountException("用户名不正确");
} else if (!userPwd.equals(userPwd)) {
    throw new AccountException("密码不正确");
}
//交给AuthenticatingRealm使用CredentialsMatcher进行密码匹配
return new SimpleAuthenticationInfo(userName, password,
                                    ByteSource.Util.bytes(userName + "salt"), getName());
//这里唯一需要注意的是：你注册的加密方式和设置的加密方式还有Realm中身份认证的方式都是要一模一样的。
//本文中的加密 ：MD5两次、salt=username+salt加密。
```

### 过滤器链附录

|    默认拦截器名    | 拦截器类                                                     | 说明（括号里的表示默认值）                                   |
| :----------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **身份验证相关的** |                                                              |                                                              |
|       authc        | org.apache.shiro.web.filter.authc.FormAuthenticationFilter   | 基于表单的拦截器；如 “`/**=authc`”，如果没有登录会跳到相应的登录页面登录；主要属性：usernameParam：表单提交的用户名参数名（ username）；  passwordParam：表单提交的密码参数名（password）； rememberMeParam：表单提交的密码参数名（rememberMe）； loginUrl：登录页面地址（/login.jsp）；successUrl：登录成功后的默认重定向地址； failureKeyAttribute：登录失败后错误信息存储 key（shiroLoginFailure）； |
|     authcBasic     | org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter | Basic HTTP 身份验证拦截器，主要属性： applicationName：弹出登录框显示的信息（application）； |
|       logout       | org.apache.shiro.web.filter.authc.LogoutFilter               | 退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/）; 示例 “/logout=logout” |
|        user        | org.apache.shiro.web.filter.authc.UserFilter                 | 用户拦截器，用户已经身份验证 / 记住我登录的都可；示例 “/**=user” |
|        anon        | org.apache.shiro.web.filter.authc.AnonymousFilter            | 匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例 “/static/**=anon” |
|   **授权相关的**   |                                                              |                                                              |
|       roles        | org.apache.shiro.web.filter.authz.RolesAuthorizationFilter   | 角色授权拦截器，验证用户是否拥有所有角色；主要属性： loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例 “/admin/**=roles[admin]” |
|       perms        | org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter | 权限授权拦截器，验证用户是否拥有所有权限；属性和 roles 一样；示例 “/user/**=perms["user:create"]” |
|        port        | org.apache.shiro.web.filter.authz.PortFilter                 | 端口拦截器，主要属性：port（80）：可以通过的端口；示例 “/test= port[80]”，如果用户访问该页面是非 80，将自动将请求端口改为 80 并重定向到该 80 端口，其他路径 / 参数等都一样 |
|        rest        | org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter | rest 风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例 “/users=rest[user]”，会自动拼出“user:read,user:create,user:update,user:delete” 权限字符串进行权限匹配（所有都得匹配，isPermittedAll）； |
|        ssl         | org.apache.shiro.web.filter.authz.SslFilter                  | SSL 拦截器，只有请求协议是 https 才能通过；否则自动跳转会 https 端口（443）；其他和 port 拦截器一样； |
|      **其他**      |                                                              |                                                              |
| noSessionCreation  | org.apache.shiro.web.filter.session.NoSessionCreationFilter  | 不创建会话拦截器，调用 subject.getSession(false) 不会有什么问题，但是如果 subject.getSession(true) 将抛出 DisabledSessionException 异常； |