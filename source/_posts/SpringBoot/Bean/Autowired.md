---
title: Autowaired.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: 注解
---

面向抽象的编程，通常，我们会在类中定义接口，而注入的是接口的实现类。

举个栗子：

```java
@RestController
@RequestMapping("/v1/banner")
public class BannerController {

    @Autowired
    private  Skill diana;

    @GetMapping(value = "test")
    public String test() {
        diana.R();
        return "ok";
    }
}
```

这里的Skill是接口。如果它有两个实现类Diana和Itera，那么到底哪个实现类会被注入呢？

注入的时候可能会发生的几种情况：

1.找不到任何一个bean ，报错

2.只找到一个bean，直接注入

3.找到多个bean，并不一定会报错，按照字段名字推断选择哪个bean

其实，总结起来就是两种方式：

1.通过类型注入

2.通过name注入 

通过类型注入的优先级 > 通过name注入。@Autowaired默认是通过类型注入。

当注入Skill的实现类时，发现了有多个实现类，此时通过类型注入不能决定应该注入哪个实现类。

既然通过类型决定不了，就会通过字段名去推断要注入哪个类。而这种通过类型和name的注入是被动地注入，我们也可以使用@Qualify("diana")来指明主动地注入某一个实现类。

```java
	@Autowired
    @Qualifier("itrea")
    private  Skill diana;
```

即使这里的字段名是diana，而我们使用了@Qualify主动注入了itera。最终注入的实现类还是itrea，调用的方法还是itrea的方法。