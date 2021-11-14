---
title: Conditional.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: 注解
---

条件注解可以解决策略模式多种实现类的问题。

使用自定义条件注解需要自己编写一个类实现conidtion接口，并在配置类中使用@conditional注解。

```java
	@Bean
    @Conditional(DatabaseCondition.class)
    public Connect mysql(){
        return new MySql(this.host,this.port);
    }
```

也可以使用spring自带的条件注解：

```java
 	@Bean
    @ConditionalOnProperty(value = "database",havingValue = "mysql",matchIfMissing = true)
    public Connect mysql(){
        return new MySql(this.host,this.port);
    }
```

value的值是配置文件中的key。matchIfMissing相当于默认值。当在配置文件中找不到database的配置项时，默认将MySql这个类加入到IOC容器中。