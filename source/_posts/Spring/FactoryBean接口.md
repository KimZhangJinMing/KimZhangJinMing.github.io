---
title: FactoryBean接口.md
date: 2020-09-04 21:52:35
categories: Spring
tags: Bean
---

### FactoryBean接口简介

FactoryBean接口是Spring提供的工厂类接口，实现这个接口的实现类拥有一些Spring提供的Bean基本功能。

泛型定义的是产生对象的类型。

FactoryBean接口定义了3个方法：

* getObjectType：返回的是被创建对象的Class
* getObject：返回的是被创建对象的实例
* isSingleton：被创建的对象是否单例

FactoryBean是懒加载的，容器启动的时候，工厂不会生产对象，直到使用到对象的时候，才会创建对象。

```sql
/**
 *@Author Ming
 *@Date 2020/09/04 21:41
 *@Description FactoryBean接口,创建的对象是Book类型


 */
public class MyFactoryBean implements FactoryBean<Book> {
	@Override
	public Class<?> getObjectType() {
		return Book.class;
	}

	@Override
	public Book getObject() throws Exception {
		return new Book();
	}

	@Override
	public boolean isSingleton() {
		return false;
	}
}

```

### 使用XML方式生产bean

在xml中配置需要使用的工厂bean，就可以通过IOC容器获取到该工厂生产的对象。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="myFactoryBean" class="com.imooc.factory.MyFactoryBean"></bean>
</beans>
```

```java
public static void main(String[] args) {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/spring-config.xml");

		Object bean1 = applicationContext.getBean("myFactoryBean");
		Object bean2 = applicationContext.getBean("myFactoryBean");
		System.out.println(bean1);// com.imooc.entity.Book@1de0aca6
		System.out.println(bean2);// com.imooc.entity.Book@255316f2
	}
```

从运行结果看，通过myFactoryBean获取到的不是工厂的实现类，而是工厂产生的对象Book类型，而且由于isSingleton方法返回的是false，生产的对象不是单例的。

有什么办法可以获取到工厂的实现类本身吗？

在BeanFactory接口里定义了这么一个属性：

```java
public interface BeanFactory {

	/**
	 * Used to dereference a {@link FactoryBean} instance and distinguish it from
	 * beans <i>created</i> by the FactoryBean. For example, if the bean named
	 * {@code myJndiObject} is a FactoryBean, getting {@code &myJndiObject}
	 * will return the factory, not the instance returned by the factory.
	 */
	String FACTORY_BEAN_PREFIX = "&";
}
```

通过&id可以获取到工厂类本身，而不是工厂类产生的对象实例。且Spring中产生的bean默认是单例的。

```java
Object bean1 = applicationContext.getBean("&myFactoryBean");
Object bean2 = applicationContext.getBean("&myFactoryBean");
System.out.println(bean1);// com.imooc.factory.MyFactoryBean@1de0aca6
System.out.println(bean2);// com.imooc.factory.MyFactoryBean@1de0aca6
```

### 使用注解方法生产对象

同XML方式一样，只需要把实现的工厂类加入到IOC容器中，就可以通过容器获取工厂生产的对象。

这里使用@Component加入到IOC容器中。

```java
@Component
public class MyFactoryBean implements FactoryBean<Book> {
    @Override
	public Class<?> getObjectType() {
		return Book.class;
	}

	@Override
	public Book getObject() throws Exception {
		return new Book();
	}

	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

此时，是通过注解的方式获取ApplicationContext，不再是通过xml的ClassPathXmlApplicationContext.

```java
public static void main(String[] args) {
    	// 扫描com.imooc包下的所有注解
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.imooc");

		Object bean1 = applicationContext.getBean("myFactoryBean");
		Object bean2 = applicationContext.getBean("myFactoryBean");
		System.out.println(bean1);// com.imooc.entity.Book@4361bd48
		System.out.println(bean2);// com.imooc.entity.Book@53bd815b
	}
```

从结果可以看到，使用注解的方式同样可以获取到工厂类产生的对象。同样的，如果需要获取到工厂的实现类，只需要通过applicationContext.getBean("&myFactoryBean")获取。



