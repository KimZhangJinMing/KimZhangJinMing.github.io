---
title: SpringBoot配置.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: 配置
---

现有一个实体类Person,现在我们需要把实体类的属性从配置文件中导入：

@ConfigurationProperties是从配置文件中导入属性的关键注解。prefix指定导入的是配置文件的person开头的属性。

@Getter、@Setter是必须的

@Component也是必须的。必须是IOC容器中的类，才能从配置文件中导入属性。在使用的时候，也必须是通过@Autowaired注入的，如果通过new创建的对象，不会注入配置文件中的属性。

```java
@Component
@ConfigurationProperties(prefix = "person")
@Getter
@Setter
@ToString
public class Person {
    private String name;
    private Integer age;
    private Boolean isBoy;
    private Date birthday;
    private Date birthday;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}

```

yml配置文件的写法：

* 注意每个值前面的空格。
* 属性左边只要对齐就可以，对缩进的字符个数没有限制。
* 键值对和对象的写法是一样的。因为对象也是键值对

```yaml
person:
  name: lisi
  age: 20
  is-boy: true
  birthday: 2020/02/01
  maps: {k1: v1,k2: 17}
  lists: [aaa,bbb,ccc]
  dog: {name: mydog,age: 6}
  
```

关于map，list，还有另外一种写法：

注意Person类中是驼峰命名的方式，在yml中可以是-的写法。

```yaml
person:
  name: lisi
  age: 20
  is-boy: true
  birthday: 2020/02/01
  maps:
    k1: v1
    k2: 17
  lists:
    - aaa
    - bbb
    - ccc
  dog:
    name: mydog
    age: 6

```

properties文件的写法：

```properties
person.name=zhangsan
person.age=18
person.is-boy=true
person.birthday=2020/03/03
person.maps.k1=v1
person.maps.k2=18
person.lists=aaa,bbb,ccc
person.dog.name=mydog
person.dog.age=7
```

运行结果：

虽然yml中是is-boy的写法，但是输出的结果还是Person类中的isBoy。同时注意到，日期格式与配置文件中的不同。

```json
{
  "name": "lisi",
  "age": 20,
  "isBoy": true,
  "birthday": "2020-01-31T16:00:00.000+0000",
  "maps": {
    "k1": "v1",
    "k2": 17
  },
  "lists": [
    "aaa",
    "bbb",
    "ccc"
  ],
  "dog": {
    "name": "mydog",
    "age": 6
  }
}
```

***

### 关于配置文件优先级问题

当同时存在yml文件和properties文件时，properties文件的优先级大于yml文件。

***

### 关于中文乱码问题

配置文件的默认编码是ASCII，而在idea中，设置了文件的编码是UTF8，就会导致乱码。

需要在Setting -> File Encodings中设置在运行时将UTF8编码转换成ASCII编码：

![image-20200620205229515](D:\ming\images\image-20200620205229515.png)

***

### 使用@Value注入配置文件中的值

* @Value注解不可以注入复杂类型(map,list,对象等)的数据。
* @Value可以使用spel表达式进行计算。比如#{11 * 2}
* @Value不支持数据验证
* @Value不支持**松散语法**，比如配置文件的is-boy，使用@Value注解不能写成isBoy。但是配置文件中的isBoy，使用@Value却可以写成is-boy。  黑人问号？？？

```java
@Component
@Getter
@Setter
@ToString
public class Person {
    @Value("${person.name}")
    private String name;
    @Value("#{11 * 2}")
    private Integer age;
    @Value("${person.is-boy}")
    private Boolean isBoy;
    @Value("${person.birthday}")
    private Date birthday;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

### 数据验证

只有使用@ConfigurationProperties才能使用数据验证。@Value的数据校验是无效的。

数据验证需要@Validated注解。

```java
@Component
@Getter
@Setter
@ToString
@Validated
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    @Max(value = 3)
    private Integer age;
    private Boolean isBoy;
    private Date birthday;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

使用@Value注解时，使用数据验证。controller访问会出现奇怪的结果。  而通过单元测试可以发现通过@Value的注入方式，数据校验是无效的。

```java
{ 
  "name": "李四",
  "age": 22,
  "isBoy": true,
  "birthday": "2020-01-31T16:00:00.000+0000",
  "maps": null,
  "lists": null,
  "dog": null,
  "frozen": false,
  "proxiedInterfaces": [
    
  ],
  "proxyTargetClass": true,
  "exposeProxy": false,
  "advisors": [
    {
      "order": 2147483647,
      "advice": {
        
      },
      "pointcut": {
        "classFilter": {
          
        },
        "methodMatcher": {
          "runtime": false
        }
      },
      "perInstance": true
    }
  ],
  "targetSource": {
    "target": {
      "name": "李四",
      "age": 22,
      "isBoy": true,
      "birthday": "2020-01-31T16:00:00.000+0000",
      "maps": null,
      "lists": null,
      "dog": null
    },
    "static": true,
    "targetClass": "com.sise.jpa.model.Person"
  },
  "targetClass": "com.sise.jpa.model.Person",
  "preFiltered": false
}
```

***

### @ConfigurationProperties与@Value注解的区别

|                | @Configuration           | @Valye         |
| -------------- | ------------------------ | -------------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定注入 |
| 松散语法       | 支持                     | 不支持         |
| SpEL           | 不支持                   | 支持           |
| JSR303数据校验 | 支持                     | 不支持         |
| 复杂类型注入   | 支持                     | 不支持         |

不管是yml配置文件还是properties配置文件，都能获取到配置文件中的值。

* 如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value。
* 如果说，我们专门编写了一个Bean来和配置文件进行映射，我们就直接使用@ConfigurationProperties。

***

### @PropertyScore与@ImportSource

@PropertyScore用来加载指定的配置文件。

* 只支持.properties文件，不支持.yml文件
* 如果application.yml或者application.properties中存在与自定义配置文件相冲突的属性，yml或properties的文件优先级高于自定义的配置文件，导致@PropertyScore失效。

```java
@Component
@Getter
@Setter
@ToString
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private Integer age;
    private Boolean isBoy;
    private Date birthday;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

@ImportSource用来导入Spring的配置文件，使配置文件中的内容生效。该注解标注在一个配置类上。

先自己创建一个spring的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloService" class="com.sise.jpa.helloService"></bean>
</beans>
```

在SpringBoot的入口类中使用@ImportSource注解：

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:spring.xml"})
public class JpaApplication {
    public static void main(String[] args) {
        SpringApplication.run(JpaApplication.class, args);
    }
}

```

***

### 配置文件属性占位符

* 使用随机数

```properties
${random.value},${random.int(10)},${random.uuid}
```

* 占位符获取之前配置的值，如果没有值可以用：指定默认值

```properties
person.dog.name=${person.name:Tom}_dog
```

***

### Profile

> 多Profile文件

默认配置文件使用application.properties

我们可以为开发环境和生产环境指定配置文件，文件名可以指定为application-{profile}.properties/yml

> yml多文档块

每个---分隔的是一个文档块，相当于重新创建一个配置文件。

```yml
server:
  port: 8080
spring:
  profiles:
    active: dev
---

server:
  port: 8081
spring:
  profiles: dev

---
server:
  port: 8082
spring:
  profiles: prod
```

> 激活指定的配置文件

* 在主配置文件中指定：值为profile

  ```properties
  spring.profile.active=dev
  ```

* 使用yml多文档块

* 使用命令行

  ```properties
  java -jar xxx.jar --spring.profiles.active=dev
  ```

* 使用虚拟机参数

  虚拟机参数都使用-D开头。

  ```properties
  -Dspring.profiles.active=dev
  ```

***

### 配置文件的加载位置

SpringBoot启动会扫描以下位置的properties/yml文件：

* 项目路径下/config/
* 项目路径下/
* classpath/config/
* classpath/

以上是按照优先级从高到低的顺序，所有位置的配置文件**都会被加载**，高优先级的配置文件内容会覆盖低优先级的配置内容，形成**互补配置**。

我们也可以通过在启动springBoot项目的时候指定spring.config.location来覆盖默认的配置。

通过指定spring.config.additional-locational来增强默认的配置，与默认配置形成互补。

**如果指定了外部的配置文件，则外部配置文件加载顺序优先级比上面四种情况高。**

**如果在application.properties/yml文件中指定了spring.config.location是无效的。**

```properties
java -jar jpa-0.0.1-SNAPSHOT.jar --spring.config.additional-location=C:\User
s\zjm16\Desktop\application.yml

```

***

### 自动配置原理

1. SpringBoot启动的时候加载主配置类，主配置类开启了自动配置功能@EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

2. @EnableAutoConfiguration作用：

* 自动扫描与启动类同级的包以及子包

* 利用AutoConfigurationImportSelector导入一些第三方组件的配置类

  ```java
  @Override
  	public String[] selectImports(AnnotationMetadata annotationMetadata) {
  		if (!isEnabled(annotationMetadata)) {
  			return NO_IMPORTS;
  		}
  		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
  				.loadMetadata(this.beanClassLoader);
  		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
  				annotationMetadata);
  		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
  	}
  ```

  selectImport中去加载类路径下的资源：(autoconfiguration包i下WEB-INF/spring.factories文件)

  ```java
  AutoConfigurationMetadataLoader
  				.loadMetadata(this.beanClassLoader)
  // 加载的资源的路径
  protected static final String PATH = "META-INF/spring-autoconfigure-metadata.properties";
  ```

  spring-autoconfigure-metadata.properties文件中有很多xxxAutoConfiguration的配置类。这些配置类根据条件将需要的组件加入到IOC容器中。

  ```properties
  org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration=
  org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration=
  org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration=
  org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration=
  ```

  我们以HttpEncodingAutoConfiguration为例：

  ```java
  @Configuration(proxyBeanMethods = false) // 声明这是一个配置类，与@Bean注解一起使用可以向IOC容器中添加组件
  @EnableConfigurationProperties(HttpProperties.class)  // 开启配置文件与实体类的映射
  @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) // 条件注解，判断当前是不是web应用，是web应用才将bean添加到IOC容器中。可以看出虽然SpringBoot加载了很多配置类，但只有满足条件的配置类才会被加入到IOC容器当中
  @ConditionalOnClass(CharacterEncodingFilter.class) // 判断当前CharacterEncodingFilter是否已经存在
  @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) // 判断配置文件中是否配置了spring.http.encoding.enabled属性，如果没有配置，设置默认值为true
  public class HttpEncodingAutoConfiguration {
      
      private final HttpProperties.Encoding properties;
      
      // 可以看到bean中属性来自HttpProperties类，而HttpProperties类的属性与配置文件中的配置形成映射
      @Bean
  	@ConditionalOnMissingBean
  	public CharacterEncodingFilter characterEncodingFilter() {
  		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
  		filter.setEncoding(this.properties.getCharset().name());			   		  filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
  		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
  		return filter;
  	}
      
  }
  ```

  HttpProperties：

  使用@ConfigurationProperties与配置文件的属性进行关联。配置文件中可配置的属性在HttpProperties类的成员属性能反映出来。

  ```java
  @ConfigurationProperties(prefix = "spring.http")
  public class HttpProperties {
      private boolean logRequestDetails;
      private final Encoding encoding = new Encoding();
  }
  ```

  精髓：

  1）SpringBoot启动会加载大量的自动配置类

  2）我们看我们需要的功能有没有SpringBoot默认写好的自动配置类

  3）我们再来看这个自动配置类中到底配置了哪些组件（只要有我们需要的组件，我们就不再需要配置了）

  4）给容器中自动配置类添加组件的时候，会从ppoperties类中获取某些属性，我们就可以再这些配置文件中指定这些属性的值。

  5）xxxxAutoConfiguration自动配置类给容器中添加组件。xxxProperties封装配置文件中相关的属性。

  ***

  ### 如何查看哪些配置类生效，哪些不生效

  在配置文件中加入debug=true。这样在启动springBoot项目时，控制台会输出配置类的信息。

  ***

  