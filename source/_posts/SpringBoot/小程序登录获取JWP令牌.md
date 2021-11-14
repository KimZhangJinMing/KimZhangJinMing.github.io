---
title: 小程序登录获取JWT令牌.md
date: 2020-10-08 20:12:35
categories: SpringBoot
tags: JWT,小程序
---

### 小程序登录获取微信服务器验证流程

![小程序登录验证机制](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/api-login.2fcc9f35.jpg)

小程序通过login方法获取到code，将code发送到服务器，服务器接收到code，加上appid和secret向微信服务器发送POST请求验证。

如果微信服务器验证通过，则开发者的服务器生成JWT令牌返回给前端，从而完成API权限的控制。

这里涉及到几个知识点：

* 如何将配置文件中的模板字符串替换成目标的字符串？
* 在服务端，如何发送POST请求？怎么解决SSL问题？
* 如何生成JWT令牌？



### 如何将配置文件中的模板字符串替换成目标的字符串？

引用同个配置文件中的其他配置项，可以通过${}方式来获取。

在配置文件中的模板字符串要被程序进行处理替换的字符串，可以使用{0}，{1}这种方式占位，在程序中使用MessageFormat来格式化。

```yml
wechat:
  appid: xxxxxxx
  secret: xxxxxxxxxx
  code2session: https://api.weixin.qq.com/sns/jscode2session?appid=${wechat.appid}&secret=${wechat.secret}&js_code={0}&grant_type=authorization_code
```

在程序在使用MessageFormat来格式化占位的字符串，方法形参是可变类型参数，对应着配置文件中的{0}，{1}等占位符。

```java
String url = MessageFormat.format(this.code2SessionUrl, code);
```



### 在服务端，如何发送POST请求？怎么解决SSL问题？

我们可以利用Spring中的RestTemplate来发送POST请求，并获取返回的结果。

getForObject的第二个参数是结果返回的类型，一般为json字符串，故用String.class.

**注意：可以通过new RestTemplate()来创建RestTempalte对象。在这里是使用了@Configuration将RestTemplate加入了IOC容器，并使用依赖注入的方式注入的。因为需要@Configration类解决SSL验证的问题。**

```java
String resultJson = restTemplate.getForObject(url, String.class);
```

使用RestTemplate解决SSL验证问题：

* 首先导入工具类SSL
* 创建@Configuration配置类，将处理过的RestTemplate加入到IOC容器中

SSL.java:

```java
/**
 *@Author Ming
 *@Date 2020/10/08 14:58
 *@Description 解决RestTemplate的SSL验证问题
 */
public class SSL extends SimpleClientHttpRequestFactory {

    @Override
    protected void prepareConnection(HttpURLConnection connection, String httpMethod)
            throws IOException {
        if (connection instanceof HttpsURLConnection) {
            prepareHttpsConnection((HttpsURLConnection) connection);
        }
        super.prepareConnection(connection, httpMethod);
    }

    private void prepareHttpsConnection(HttpsURLConnection connection) {
        connection.setHostnameVerifier(new SkipHostnameVerifier());
        try {
            connection.setSSLSocketFactory(createSslSocketFactory());
        }
        catch (Exception ex) {
            // Ignore
        }
    }

    private SSLSocketFactory createSslSocketFactory() throws Exception {
        SSLContext context = SSLContext.getInstance("TLS");
        context.init(null, new TrustManager[] { new SkipX509TrustManager() },
                new SecureRandom());
        return context.getSocketFactory();
    }

    private class SkipHostnameVerifier implements HostnameVerifier {

        @Override
        public boolean verify(String s, SSLSession sslSession) {
            return true;
        }

    }

    private static class SkipX509TrustManager implements X509TrustManager {

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) {
        }

    }

}

```

@Configuration配置类：

```java
@Configuration
public class RestTemplateConfiguration {

    @Configuration
    public class RestTemplateConfig {

        @Bean
        public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
            return new RestTemplate(factory);
        }

        @Bean
        public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
            SSL factory = new SSL();
            factory.setReadTimeout(5000);
            factory.setConnectTimeout(15000);//单位为ms
            return factory;
        }
    }
}

```



### 如何生成JWT令牌？

我们使用auth0的jwt库，来生成JWT令牌。

首先，导入Maven依赖：

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.8.1</version>
</dependency>
```

**实现JWT令牌的生成：**

withClaim方法主要是自定义信息，

withIssuedAt方法是令牌的生成时间Date，

withExpiresAt方法是令牌的过期时间Date，

sign方法最终生成一个JWT令牌字符串

```java
private static Optional<String> getToken(Long userId, int scopeLevel) {
    try {
        // 选择算法
        Algorithm algorithm = Algorithm.HMAC256(jwtKey);
        Time time = JwtToken.getTime();

        return Optional.of(JWT.create()
                           .withClaim("userId", userId)
                           .withClaim("scopeLevel", scopeLevel)
                           .withIssuedAt(time.getCurrentTime())
                           .withExpiresAt(time.getExpiredTime())
                           .sign(algorithm));
    } catch (Exception e) {
        return Optional.empty();
    }
}

```

**JWT令牌的验证：**

```java
/**
* 验证JWT令牌,并返回自定义的信息
* @param token JWT令牌
* @return Optional<Map<String, Claim>>
*/
public static Optional<Map<String, Claim>> getClaims(String token){
    try {
        Algorithm algorithm = Algorithm.HMAC256(JwtToken.jwtKey);

        JWTVerifier verifier = JWT.require(algorithm).build();
        DecodedJWT decodedJwt = verifier.verify(token);
        return Optional.of(decodedJwt.getClaims());
    } catch (Exception e) {
        return Optional.empty();
    }
}
```

