---
title: ResponseBody.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: 注解
---

假如说后端想返回一个string类型字符串"你好,SpringBoot"给前端。那么，有以下两种方案:

> 通过HttpServletResponse对象

```java
@GetMapping(value = "test")
public void test(HttpServletResponse response) throws IOException {
    response.getWriter().write("你好 ,SpringBoot！");
}
```



我们通过postman来看一下前端接收到的结果和Header中的值。

<img src="D:\ming\images\clipboard-1590410602713.png" alt="img" style="zoom:67%;" />

<img src="D:\ming\images\clipboard-1590410602714.png" alt="img" style="zoom: 67%;" />

当我们用HttpServletResponse返回数据的时候，应该手动设置content-type，编码等。

```java
@GetMapping(value = "test")
public void test(HttpServletResponse response) throws IOException {
    response.setHeader("content-type","text/plains");
    response.setCharacterEncoding("UTF-8");
    response.getWriter().write("你好 ,SpringBoot！");
}
```





当然也可以通过一行代码完成设置:

```java
response.setHeader("content-type","text/plains;charset=UTF-8");
```

这样设置后，返回的结果编码和content-type都正常了。

<img src="D:\ming\images\clipboard-1590410602715.png" alt="img" style="zoom:67%;" />

通过@ResponseBody注解

```java
@GetMapping(value = "test")
@ResponseBody
public String test() {
    return "你好，SpringBoot";
}
```

从结果来看，@responseBody会自动帮我们设置content-type和编码：

<img src="D:\ming\images\clipboard-1590410602716.png" alt="img" style="zoom:67%;" />

以上返回的是字符串，那如果返回的是一个Object呢？

```java
@GetMapping(value = "test")
@ResponseBody
public Map<String,Object> test() {
    Map<String,Object> map = new HashMap<>();
    map.put("id","1");
    map.put("name","王小喵");
    return map;
}
```

我们来看一下返回结果会是什么? 可以看到，@ResponseBody注解自动添加了content-type，还将对象自动转换成了json格式

<img src="D:\ming\images\clipboard-1590410602716.png" alt="img" style="zoom:67%;" />

<img src="D:\ming\images\clipboard-1590410602716.png" alt="img" style="zoom:67%;" />

> 总结：

使用@ResponseBody来代替HttpServletResponse。HttpServletResponse需要自己设置编码等很多属性，而@ResponseBody会自动帮我们回写content-type、编码等值