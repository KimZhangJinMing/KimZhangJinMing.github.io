### 踩坑集锦

#### 1.1 No AuthenticationProvider found for org.springframework.security.authentication.UsernamePasswordAuth

有可能的原因是重写了configure方法，而方法体内是空的。这是一种非常傻逼的操作...



#### 1.2 java.lang.IllegalStateException: UserDetailsService is required.

出现这种错误的原因可能是在配置文件中配置了用户名和密码...,而且用户名和密码是可以进行登陆的。不过后台会报错，只需将配置文件中的用户名密码配置在config方法中即可。

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user")
        .password("123")
        .roles("admin");
}
```



### 1. Remember-Me的使用

如果需要实现`自动登陆`的功能，且在关闭浏览器后重开，或者重启服务器后还能自动登陆，可以使用Spring Security的Remember-me功能。只需要配置一下rememberMe()即可。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest()
        .authenticated()
        .and()
        .formLogin()
        .permitAll()
        .and()
        .rememberMe()
        .and()
        .csrf()
        .disable();
}
```

当配置rememberMe()后，自动登陆功能就可以实现了：

* 在登陆页面，会出现Remember me on this computer.的选项。如果勾选了这个选项，发送POST请求的/login会在body中增加一个remember-me:on的参数。`如果是自定义的登陆页面,想使用remember-me的功能,传递的key值就应该是remember-me`.

* 当勾选了后，再次访问其他接口，Cookie中会携带remember-me:

```txt
Cookie: JEECGINDEXSTYLE=ace;JSESSIONID=93573581FD0676F64B7D495F9968D6AE; remember-me=dXNlcjoxNjE1OTYzNzQ0Mzk5OjhiYTBhZjkxYzlhNzBhYjlkMzQwNGMyNWM2OWFhYzdk
```



接下来，我们来研究一下remember-me的组成，这串字符串是经过Base64编码后的，我们写段小代码来还原它的真面目：

```java
@Test
public void test() {
    byte[] bytes =
      Base64.getDecoder().decode("dXNlcjoxNjE1OTYyNzY2NDEyOmQyMWJkNmI5YjY3YjM0MmRkN2UwZjIzMjkxN2FmZjRl");
    System.out.println(new String(bytes));
}
```

输出结果为:分割的字符串，其中：

- 第一段为`用户名`，也就是在登陆界面登陆的用户名
- 第二段为`过期时间`，默认是两周
- 第三段为`MD5计算的散列值`。他的明文格式是 `username + ":" + tokenExpiryTime + ":" + password + ":" + key`，最后的 key 是一个散列盐值，可以用来防治令牌被修改

```txt
user:1615962766412:d21bd6b9b67b342dd7e0f232917aff4e
```



那么，如果用户勾选了Remember-me的选项后，登陆流程是这样子的：

用户勾选remember-me选项后，/login请求会带上remember-on的参数。在浏览器关闭后，或服务器重启后，用户再去访问接口，此时会携带Cookie中的remember-me到服务端。服务端拿到这个remember-me，就可以解析用户名和过期时间，再根据用户名查询到用户密码，然后通过MD5散列函数计算出散列值，将计算出的散列值与浏览器传递的散列值进行对比，就能确认这个令牌是否有效。



### 2. Remember-Me的生成与校验

#### 2.1 生成过程

按照remember-me的格式，生成的过程需要的有`用户名`，`过期时间`,`密码(MD5加密中使用)`。

remember-me参数的生成是在TokenBasedRememberMeServices类中的onLoginSuccess方法中:

```java
public void onLoginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication successfulAuthentication) {
    // 1. 获取用户名和密码
    String username = this.retrieveUserName(successfulAuthentication);
    String password = this.retrievePassword(successfulAuthentication);
    if (!StringUtils.hasLength(username)) {
        this.logger.debug("Unable to retrieve username");
    } else {
        // 登陆成功后,密码可能被擦除了。从数据库中获取密码
        if (!StringUtils.hasLength(password)) {
            UserDetails user = this.getUserDetailsService().loadUserByUsername(username);
            password = user.getPassword();
            if (!StringUtils.hasLength(password)) {
                this.logger.debug("Unable to obtain password for user: " + username);
                return;
            }
        }
		// 2.计算过期时间
        int tokenLifetime = this.calculateLoginLifetime(request, successfulAuthentication);
        long expiryTime = System.currentTimeMillis();
        // 如果过期的时间没有设置，默认为当前的时间加上两周
        expiryTime += 1000L * (long)(tokenLifetime < 0 ? 1209600 : tokenLifetime);
        // 3.MD5生成散列值
        String signatureValue = this.makeTokenSignature(expiryTime, username, password);
        // 4.设置到cookie中
        this.setCookie(new String[]{username, Long.toString(expiryTime), signatureValue}, tokenLifetime, request, response);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Added remember-me cookie for user '" + username + "', expiry: '" + new Date(expiryTime) + "'");
        }

    }
}
```

makeTokenSignature使用MD5生成散列值，可以看到散列值的明文格式。

同时也说明了MD5加密使用`MessageDigest`，也要注意使用Hex.encode进行编码，否则输出是乱码。

```java
protected String makeTokenSignature(long tokenExpiryTime, String username, String password) {
    String data = username + ":" + tokenExpiryTime + ":" + password + ":" + this.getKey();

    try {
        MessageDigest digest = MessageDigest.getInstance("MD5");
        return new String(Hex.encode(digest.digest(data.getBytes())));
    } catch (NoSuchAlgorithmException var7) {
        throw new IllegalStateException("No MD5 algorithm available!");
    }
}
```

散列函数中的key如果没有设置，默认是在 RememberMeConfigurer#getKey 方法中进行设置的，它的值是一个 UUID 字符串。

```java
private String getKey() {
    if (this.key == null) {
        if (this.rememberMeServices instanceof AbstractRememberMeServices) {
            this.key = ((AbstractRememberMeServices)this.rememberMeServices).getKey();
        } else {
            this.key = UUID.randomUUID().toString();
        }
    }

    return this.key;
}
```

由于我们自己没有设置 key，key 默认值是一个 UUID 字符串，这样会带来一个问题，就是如果服务端重启，这个 key 会变，这样就导致之前派发出去的所有 remember-me 自动登录令牌失效(`测试好像并不会？？？`)，所以，我们可以指定这个 key。指定方式如下：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest()
        .authenticated()
        .and()
        .formLogin()
        .permitAll()
        .and()
        .rememberMe()
        .key("ming")
        .and()
        .csrf()
        .disable();
}
```



#### 2.2 校验过程

RememberMeAuthenticationFilter#doFilter方法:

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
    throws IOException, ServletException {
    if (SecurityContextHolder.getContext().getAuthentication() != null) {
        this.logger.debug(LogMessage
                          .of(() -> "SecurityContextHolder not populated with remember-me token, as it already contained: '"
                              + SecurityContextHolder.getContext().getAuthentication() + "'"));
        chain.doFilter(request, response);
        return;
    }
    // 获取不到用户信息,进行自动登陆的处理
    Authentication rememberMeAuth = this.rememberMeServices.autoLogin(request, response);    		......
```

AbstractRememberMeServices#autoLogin:

真正校验remember-me参数的方法是processAutoLoginCookie。它是一个抽象方法，实现有2个：

* TokenBasedRememberMeServices
* PersistentTokenBasedRememeberMeServices

这里使用的是TokenBasedRememberMeServices。而PersistentTokenBasedRememeberMeServices是持久化令牌使用的，后续会介绍。

```java
public final Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {
    // 提取cookie
    String rememberMeCookie = extractRememberMeCookie(request);
    if (rememberMeCookie == null) {
        return null;
    }
    this.logger.debug("Remember-me cookie detected");
    if (rememberMeCookie.length() == 0) {
        this.logger.debug("Cookie was empty");
        cancelCookie(request, response);
        return null;
    }
    try {
        // 解码
        String[] cookieTokens = decodeCookie(rememberMeCookie);
        // 对remember-me参数的校验,包括用户,过期时间等
        UserDetails user = processAutoLoginCookie(cookieTokens, request, response);
        this.userDetailsChecker.check(user);
        this.logger.debug("Remember-me cookie accepted");
        return createSuccessfulAuthentication(request, user);
    }
```

可以看到，核心就是提取出 cookie 信息，并对 cookie 信息进行解码，解码之后，再调用 processAutoLoginCookie 方法去做校验。

### 3. 风险以及解决方法

如果我们开启了 RememberMe 功能，最最核心的东西就是放在 cookie 中的令牌了，这个令牌突破了 session 的限制，即使服务器重启、即使浏览器关闭又重新打开，只要这个令牌没有过期，就能访问到数据。

一旦令牌丢失，别人就可以拿着这个令牌随意登录我们的系统了，这是一个非常危险的操作。

但是实际上这是一段悖论，为了提高用户体验（少登录），我们的系统不可避免的引出了一些安全问题，不过我们可以通过技术将安全风险降低到最小。

#### 3.1 持久化令牌
持久化令牌就是在基本的自动登录功能基础上，又增加了新的校验参数，来提高系统的安全性，这一些都是由开发者在后台完成的，对于用户来说，登录体验和普通的自动登录体验是一样的。

在持久化令牌中，新增了两个经过 MD5 散列函数计算的校验参数，一个是 series，另一个是 token。其中，series 只有当用户在使用用户名/密码登录时，才会生成或者更新，而 token 只要有新的会话，就会重新生成，这样就可以避免一个用户同时在多端登录，就像手机 QQ ，一个手机上登录了，就会踢掉另外一个手机的登录，这样用户就会很容易发现账户是否泄漏。

要想使用持久化令牌，我们就需要一张表来保存令牌。可以自定义表，也可以使用Spring Security提供的JdbcTokenRepositoryImpl。根据JdbcTokenRepositoryImpl定义的操作sql，创建表的sql如下：
```
CREATE TABLE `persistent_logins` (  
 `username` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,  
 `series` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,  
 `token` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,  
 `last_used` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,  
 PRIMARY KEY (`series`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

用来保存令牌的处理类是PersistentRememberMeToken。

```java
public class PersistentRememberMeToken {
	private final String username;
	private final String series;
	private final String tokenValue;
	private final Date date; // 上一次自动登陆的时间
    //省略 getter
}
```

此外，我们还需要添加JDBC和Mysql的依赖。并在配置文件中配置数据库相关的属性:

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/security?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
```
Spring Security的配置类也需要添加JdbcRepositoryImpl和tokenRepository:
```java
 @Autowired
 DataSource dataSource;

 @Bean
 JdbcTokenRepositoryImpl jdbcTokenRepository() {
     final JdbcTokenRepositoryImpl repository = new JdbcTokenRepositoryImpl();
     repository.setDataSource(dataSource);
     return repository;
 }

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .formLogin()
            .permitAll()
            .and()
            .rememberMe()
            .key("ming")
            .tokenRepository(jdbcTokenRepository())
            .and()
            .csrf()
            .disable();
}
```

访问流程和使用RememberMe的时候没有什么不同(用户的体验是一样的)。唯一不同的是持久化令牌的方式把remember-me保存到了数据库中。

前端传递的参数remember-me经过Base64解码后:
```
uJQ60QhTdmucg5jnmPjhDg%3D%3D:tuQyXHvjgimTtFI7jLnQgw%3D%3D
```
`%3D`代表`=`。此时查看数据库中的表，多了一条记录：
|  series| token |
|--|--|
| uJQ60QhTdmucg5jnmPjhDg== | tuQyXHvjgimTtFI7jLnQgw==|

数据库中的记录和我们看到的 remember-me 令牌解析后是一致的。



#### 3.2 持久化令牌生成流程

持久化令牌的生成流程在PersistentTokenBasedRememberMeServices#onLoginSuccess:

与TokenBasedRememberMeServices不同的是，PersistentTokenBasedRememberMeServices不需要获取用户的密码，series和token都是调用SecureRandom 随机生成的。不同于我们以前用的 Math.random 或者 java.util.Random 这种伪随机数，SecureRandom 则采用的是类似于密码学的随机数生成规则，其输出结果较难预测，适合在登录这样的场景下使用。

```java
@Override
protected void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,
                              Authentication successfulAuthentication) {
    String username = successfulAuthentication.getName();
    this.logger.debug(LogMessage.format("Creating new persistent login for user %s", username));
    // 生成令牌对象
    PersistentRememberMeToken persistentToken = new PersistentRememberMeToken(username, generateSeriesData(),generateTokenData(), new Date());
    try {
        // 向数据库中添加
        this.tokenRepository.createNewToken(persistentToken);
        // 添加cookie
        addCookie(persistentToken, request, response);
    }
    catch (Exception ex) {
        this.logger.error("Failed to save persistent token ", ex);
    }
}

// 随机生成series，Base64编码
protected String generateSeriesData() {
    byte[] newSeries = new byte[this.seriesLength];
    this.random.nextBytes(newSeries);
    return new String(Base64.getEncoder().encode(newSeries));
}

// 随机生成token，Base64编码
protected String generateTokenData() {
    byte[] newToken = new byte[this.tokenLength];
    this.random.nextBytes(newToken);
    return new String(Base64.getEncoder().encode(newToken));
}
```



#### 3.3 持久化令牌校验流程

持久化令牌的校验核心在PersistenctTokenBasedRememberMeServices#processAutoLoginCookie:

```java
@Override
protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request,
                                             HttpServletResponse response) {
    if (cookieTokens.length != 2) {
        throw new InvalidCookieException("Cookie token did not contain " + 2 + " tokens, but contained '"+ Arrays.asList(cookieTokens) + "'");
    }
    String presentedSeries = cookieTokens[0];
    String presentedToken = cookieTokens[1];
    // 从数据库中获取令牌对象
    PersistentRememberMeToken token = this.tokenRepository.getTokenForSeries(presentedSeries);
    if (token == null) {
        // No series match, so we can't authenticate using this cookie
        throw new RememberMeAuthenticationException("No persistent token found for series id: " + presentedSeries);
    }
    // 检验token，如果查出来的 token 和前端传来的 token 不相同，说明账号可能被人盗用（别人用你的令牌登录之后，token 会变）。此时根据用户名移除相关的 token，相当于必须要重新输入用户名密码登录才能获取新的自动登录权限。
    if (!presentedToken.equals(token.getTokenValue())) {
        // Token doesn't match series value. Delete all logins for this user and throw
        // an exception to warn them.
        this.tokenRepository.removeUserTokens(token.getUsername());
        throw new CookieTheftException(this.messages.getMessage(
            "PersistentTokenBasedRememberMeServices.cookieStolen",
            "Invalid remember-me token (Series/token) mismatch. Implies previous cookie theft attack."));
    }
    // 校验token过期时间
    if (token.getDate().getTime() + getTokenValiditySeconds() * 1000L < System.currentTimeMillis()) {
        throw new RememberMeAuthenticationException("Remember-me login has expired");
    }
    // Token also matches, so login is valid. Update the token value, keeping the
    // *same* series number.
    // 保持series不变，更新token值
    this.logger.debug(LogMessage.format("Refreshing persistent login token for user '%s', series '%s'",token.getUsername(), token.getSeries()));
    PersistentRememberMeToken newToken = new PersistentRememberMeToken(token.getUsername(), token.getSeries(),generateTokenData(), new Date());
    try {
        this.tokenRepository.updateToken(newToken.getSeries(), newToken.getTokenValue(), newToken.getDate());
        // 重新添加到cookie中
        addCookie(newToken, request, response);
    }
    catch (Exception ex) {
        this.logger.error("Failed to update token: ", ex);
        throw new RememberMeAuthenticationException("Autologin failed due to data access problem");
    }
    // 根据用户名查询用户信息，再走一波登录流程。
    return getUserDetailsService().loadUserByUsername(token.getUsername());
}
```



#### 3.4 二次校验

为了让用户使用方便，我们开通了自动登录功能，但是自动登录功能又带来了安全风险，一个规避的办法就是如果用户使用了自动登录功能，我们可以只让他做一些常规的不敏感操作，例如数据浏览、查看，但是不允许他做任何修改、删除操作，如果用户点击了修改、删除按钮，我们可以跳转回登录页面，让用户重新输入密码确认身份，然后再允许他执行敏感操作。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/hello").authenticated()
        .antMatchers("/admin").fullyAuthenticated()
        .antMatchers("/remember").rememberMe()
        .and()
        .formLogin()
        .permitAll()
        .and()
        .rememberMe()
        .tokenRepository(jdbcTokenRepository())
        .and()
        .csrf()
        .disable();
}
```

1. /remember接口是需要 rememberMe 才能访问。
2. /admin 是需要 fullyAuthenticated，fullyAuthenticated 不同于 authenticated，fullyAuthenticated 不包含自动登录的形式，而 authenticated 包含自动登录的形式。
3. /hello是 authenticated 就能访问，也就是账号密码登陆和自动登陆都可以。