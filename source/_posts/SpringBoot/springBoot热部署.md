---
title: SpringBoot热部署.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: SpringBoot热部署
---

> 导入devtools jar

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

这里要注意的是，一定要确保devtools导入成功。有时候在pom.xml文件中是没有报错的，但是在右侧的maven工具栏看是有红色下划线的，最好检查一下jar包是不是正确的导入了。

还有一点是一定要设置optional才生效

devtools监听的实际上是target目录下的class文件，只有class文件发生了改变，devtools才会重启服务器。那么，我们可以通过2种方法来更新target目录下的class文件。

- 手动build project
- 通过idea设置自动更新

>  idea设置自动热部署

![img](D:\ming\images\clipboard.png)

这样，当改动java文件时，idea会自动更新target目录下的class文件，从而触发devtools重启服务器，完成了热部署。