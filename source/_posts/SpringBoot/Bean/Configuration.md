---
title: Configuration.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags:  注解
---

@Configuration也是一个将bean加入到IOC容器的注解。那么它和其他的模式注解有什么区别吗？

其他的模式注解只能将一个类加入到IOC容器，也能通过配置文件修改类的属性。但是它的缺点是不能应对一个接口有多个实现类的变化。假如需要注入的是另外的一个实现类，就很难实现了。

而@Configuration与@Bean的配合使用，不仅能通过配置文件修改类的属性，还能在将类加入到IOC容器前做一些逻辑处理。这种方式不仅可以应该属性带来的变化，还能通过条件注解来应对注入不同实现类的变化。

举个栗子：

比如现在有一个Connect接口，其有两个实现类MySql和Oracle。两个实现类都有各自的host和port属性。

```java
public interface Connect {
    void connect();
}
```

```java
public class MySql implements Connect {

    private String host;
    private Integer port;

    public MySql(String host, Integer port) {
        this.host = host;
        this.port = port;
    }

    @Override
    public void connect() {
        System.out.println(this.host + ":" + this.port);
    }
}
```

```java
public class Oracle implements Connect {

    private String host;
    private Integer port;

    public Oracle(String host, Integer port) {
        this.host = host;
        this.port = port;
    }

    @Override
    public void connect() {
        System.out.println(this.host + ":" + this.port);
    }
}

```

现在要将Mysql注入到IOC容器中，使用传统的模式注解和@Configuration都是可以完成的，也可以通过配置文件注入属性。

```java
@Configuration
public class Config {

    @Value("${mysql.host}")
    private String host;
    @Value("${mysql.port}")
    private Integer port;

    @Bean
    public Connect init(){
        return new MySql(this.host,this.port);
    }
}
```

假如现在需要注入的是Oracle的实现类，那么传统的模式注解就无法完成了。而@Configuration可以通过条件注解注入不同的实现类。

总结：@Configuration将bean加入IOC的方式更加灵活，它可以做逻辑处理，也可以一起注入多个Bean。它通过条件注解和配置文件注入属性，能够应对两种情况所带来的变化。