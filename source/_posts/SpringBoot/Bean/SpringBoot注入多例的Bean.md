---
title: SpringBoot注入多例的Bean.md
date: 2020-10-14 21:21:35
categories: SpringBoot
tags:  Bean
---

### SpringBoot如何注入多例Bean？

SpringBoot默认是注入单例模式的Bean，可以在Bean上添加@Scope(value = "prototype")注解来注入多例的Bean。

```java
@Component
@Scope(value = "prototype")
public class CouponChecker {}
```

使用成员变量的方式注入CouponChecker实例。

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @Autowired
    CouponChecker couponChecker;

    @GetMapping("/prototype")
    public void test(){
        System.out.println(couponChecker);
    }
}
```

输出结果：

多次访问后，控制台打印如下。我们可以看到多次打印的hasd值都是一样的，那是不是说明@Scope注解没有生效，注入的还是单例的Bean呢？

```java
com.sise.ming.logic.CouponChecker@347a792b
com.sise.ming.logic.CouponChecker@347a792b
```

其实不然，此时已经是多例模式，但由于成员属性的注入只是在类实例实例化的时候注入一次，导出打印出的hash码是一致的。

如果将成员属性的注入方式改为方法属性的注入，可以看到打印的hash值不再一样。

```java
@RestController
@RequestMapping("/test")
public class TestController {
    
    @GetMapping("/prototype")
    public void test(@Autowired CouponChecker couponChecker){
        System.out.println(couponChecker);
    }
}
```

```java
com.sise.ming.logic.CouponChecker@43eab93e
com.sise.ming.logic.CouponChecker@26b55617
```



### 其他几种注入多例的方式

* 使用ObjectFactory<T>,缺点是使用的时候需要调用getObject方法

  ```java
  @RestController
  @RequestMapping("/test")
  public class TestController {
  
      @Autowired
      private ObjectFactory<CouponChecker> factory;
  
      @GetMapping("/prototype")
      public void test(){
          System.out.println(factory.getObject());
      }
  }
  
  ```

* 使用动态代理，注入的是类选择ScopedProxyMode.TARGET_CLASS，注入的是接口选择ScopedProxyMode.INTERFACES

  ```java
  @Component
  @Scope(value = "prototype",proxyMode = ScopedProxyMode.TARGET_CLASS)
  public class CouponChecker {}
  ```

  ```java
  @RestController
  @RequestMapping("/test")
  public class TestController {
  
      @Autowired
      private CouponChecker couponChecker;
  
      @GetMapping("/prototype")
      public void test(){
          System.out.println(couponChecker);
      }
  }
  ```

  

