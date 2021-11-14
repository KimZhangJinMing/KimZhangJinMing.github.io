---
title: 确定bean的运行时类型.md
date: 2020-09-15 22:19:35
categories: Spring
tags: IOC
---

### Spring中创建bean的方式

#### xml配置文件

1. 通过构造方法创建bean

如果不知道构造方法，默认使用无参构造方法：

```xml
<bean id="book" class="com.column.entity.Book"></bean>
```

**class属性是必须的,如果没有指定class属性，会报以下错误:**

```java
Error creating bean with name 'book' defined in class path resource [spring-config.xml]: Instantiation of bean failed; nested exception is java.lang.IllegalStateException: No bean class specified on bean definition
```

**bean的id只能指定一个且不能重复，但是name属性可以指定多个，使用，或；分隔。**

name属性是对bean取别名，通过name和通过id获取的bean是同一个bean。

```java
Bean name 'book' is already used in this <beans> element
```

若需要使用有参构造方法：

```xml
<bean id="book" name="b1;b2" class="com.column.entity.Book">
    <constructor-arg name="name" value="Java"></constructor-arg>
</bean>
```



2. 通过静态方法创建bean

   ```xml
   <!--    配置静态工厂，静态工厂创建对象的静态方法    -->
   <bean id="bookStaticFactory" class="com.column.factory.BookStaticFactory" factory-method="getBookInstance"></bean>
   <!--    指定静态工厂创建bean    -->
   <bean id="book" class="com.column.entity.Book" factory-bean="bookStaticFactory"></bean>
   ```

   **factory-method这个属性是配置在静态工厂的bean上的。**

   **bean标签指定的class类型可以和静态工厂返回对象的类型不一样，在bean标签指定的class类型是通过构造方法创建对象时的实际类型，如果是通过静态工厂创建的对象，bean的实际类型是静态工厂返回对象的类型。**

   **静态工厂本身的对象不能通过&beanName来获取，因为它没有实现FactoryBean接口。**



3. 通过实例工厂创建bean

   ```xml
   <!--配置实例工厂-->
   <bean id="bookInstanceFactory" class="com.column.factory.BookInstanceFactory"></bean>
   <!--指定实例工厂创建bean-->
   <bean id="book" factory-bean="bookInstanceFactory" factory-method="getBookInstance"></bean>
   ```

   实例工厂的配置和静态工厂有一点不同，**实例工厂的factory-method属性是配置在要创建的bean上的，而静态工厂是配置的工程类的bean上**

   **实例工厂不需要指定bean的class属性。**

   **实例工厂获取被创建的对象是通过对象的bean id获取，而不是通过工厂的bean id。而静态工厂获取被创建的对象是通过静态工厂的bean id获取。**

   **通过实例工厂bean id获取到的是实例工厂本身。**

   **实例工厂可以创建不同的对象，只要指定一个实例工厂，工厂里可以有多个实例方法来创建对象。**

   ```xml
   <!--配置实例工厂-->
   <bean id="bookInstanceFactory" class="com.column.factory.BookInstanceFactory"></bean>
   <!--指定实例工厂的getBookInstance方法创建book实例-->
   <bean id="book" factory-bean="bookInstanceFactory" factory-method="getBookInstance"></bean>
   <!--指定实例工厂的getUserInstance方法创建user实例-->
   <bean id="user" factory-bean="bookInstanceFactory" factory-method="getUserInstance"/>
   ```

   

4.通过FactoryBean接口创建对象

```xml
<bean id="bookFactory" class="com.column.factory.BookFactory"></bean>

<bean id="book" class="com.column.entity.Book" factory-bean="bookFactory"/>
```

**需要指定class属性，否则抛出异常。**

**这样配置，可以使用bean的构造方法来创建bean，也可以通过bookFactory来创建bean。使用bean的构造方法创建bean的实际类型取决于class属性，使用bookFactory创建bean的实际类型取决于bookFactory中getObjectType方法的返回值。**

**FactoryBean的实际类型是创建的对象的类型，如果需要获取FactoryBean的实例，需要使用&+beanName的方式。**





