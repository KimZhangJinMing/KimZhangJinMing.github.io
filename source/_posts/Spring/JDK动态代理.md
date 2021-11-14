---
title: JDK动态代理.md
date: 2020-09-04 22:54:35
categories: Spring
tags: AOP
---

### 什么是JDK动态代理

JDK动态代理为实现了接口的类生成一个代理对象。

使用JDK提供的Proxy类可以生成代理对象，使用了Lambda表达式的写法：

```java
public static Calculate getProxy(Calculator calculator){

		ClassLoader loader = calculator.getClass().getClassLoader();
		Class<?>[] interfaces = calculator.getClass().getInterfaces();

		return (Calculate)Proxy.newProxyInstance(loader,interfaces,(proxy, method, args) -> {
             // 方法执行前的操作
			System.out.println(method.getName() + "开始执行，方法参数" + Arrays.asList(args));
		   	// 注意这里执行的是被代理对象的方法，而不是生成的代理对象的方法
			// calculator而不是proxy,proxy一般不做修改，如果不小心修改了，会出现递归的情况
			Object result = method.invoke(calculator, args);
             // 方法执行后的操作
			System.out.println(method.getName() + "结束执行");
			return result;
		});
	}
```

我们先来了解一下几个参数的含义：

* loader：类加载器，可以通过Class元类获取
* interfaces：被代理对象实现的接口，可以通过Class元类获取。正因为代理对象和被代理对象实现了同一个接口，返回的代理对象可以放心的强制类型转换。
* invocationHandler：函数式接口，定义了invoke方法，返回生成的代理对象。

```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

* proxy：生成的代理对象，供JDK使用，一般不修改它。如果不小心修改了它，会造成“递归”的现象。
* method：被代理对象要执行的方法，通过反射调用方法。
* args：被代理对象执行方法的参数。

有了被代理对象的方法Method和方法参数args，就可以通过反射调用method.invoke执行被代理对象上的方法。**注意的是，这里返回的代理对象是执行invoke方法的返回值。**

