---
title: Spring AOP.md
date: 2020-09-05 14:43:35
categories: Spring
tags: AOP
---

### 通知的几种类型

* 前置通知（Before）
* 后置通知（AfterReturning）
* 异常通知（AfterThrowing）
* 最后通知（After）：无论方法是正常结束，还是发生异常，都会执行。相当于写在finally块里的代码
* 环绕通知（Around）



### Spring AOP使用步骤

1. 导入aop相关的包

   ```xml
   org.springframework.spring-aop
   org.aspectj.aspectjweaver
   aopalliance.aopalliance
   ```

2. 开启AOP自动代理、包扫描

   ```xml
   <!-- 扫描注解 -->
   <context:component-scan base-package="com.imooc"/>
   
   <!-- 开启自动代理 -->
   <aop:aspectj-autoproxy/>
   ```

3. 编写切面类，使用@Aspect标注，并加入IOC容器

   ```java
   @Component
   @Aspect
   public class LogUtils {
   
       @Before("execution(public Integer com.imooc.demo3.Calculator.*(..))")
       public void before(){
           System.out.println("方法执行前before...");
       }
   
       @After("execution(public Integer com.imooc.demo3.Calculator.*(..))")
       public void after(){
           System.out.println("方法执行结束after...");
       }
   
       @AfterReturning("execution(public Integer com.imooc.demo3.Calculator.*(..))")
       public void afterReturning(){
           System.out.println("法正常返回AfterReturning...");
       }
   
       @AfterThrowing("execution(public Integer com.imooc.demo3.Calculator.*(..))")
       public void afterThrowing(){
           System.out.println("方法发生异常AfterThrowing:"+e.getMessage());
       }
   
       @Around("execution(public int com.imooc.demo3.Calculator.*(..))")
       public void around(){
           Object[] args = joinPoint.getArgs();
   
           Object proceed = null;
           try {
               System.out.println("[环绕通知]前置...");
               // 控制主方法的执行
               proceed = joinPoint.proceed(args);
               System.out.println("[环绕通知]后置...");
           } catch (Exception e) {
               System.out.println("[环绕中异常通知]...");
               // 环绕通知内发生的异常，如果捕获，需要再次抛出，方便下一个AfterThrowing捕获
               throw new RuntimeException("环绕内发生了异常");
           } finally {
               System.out.println("[环绕结束通知]...");
           }
   
           return proceed;
       }
   }
   
   ```

### 没有环绕通知的执行顺序

```java
try{
    @Before
    // 方法执行
    @AfterReturning
}catch (Exception e){
    @AfterThrowing
}finally {
    @After
}
```

* 方法正常执行：@Before ->  正常方法执行 ->  @After -> @AfterReturning
* 方法发生异常：@Before ->  正常方法执行 ->  @After -> @AfterThrowing(@AfterReturning执行)

注意的是先执行@After通知之后，再执行@AfterReturning通知。



### 环绕通知

环绕通知相当于拥有其他四种类型的通知,因为它可以拦截目标方法执行。

不调用Object obj = proceedingJoinPoint.proceed();即可拦截原有方法执行。

```java
proceed = joinPoint.proceed(args);
```

这一行代码相当于JDK动态代理中的method.invoke(obj,args);在执行方法过后，要把方法返回的对象return。



### 环绕通知的执行顺序

环绕通知的执行顺序与其他通知的执行顺序有一点不同，下面代码的执行顺序是：

* 方法正常执行：[环绕通知]前置...  ->   正常方法执行 ->  [环绕通知]后置...  -> [环绕结束通知]...
* 方法发生异常：[环绕通知]前置...  ->   正常方法执行  -> 环绕内发生了异常  -> [环绕结束通知]...

```java
try {
    System.out.println("[环绕通知]前置...");
    // 控制主方法的执行
    proceed = joinPoint.proceed(args);
    System.out.println("[环绕通知]后置...");
} catch (Exception e) {
    System.out.println("[环绕中异常通知]...");
    // 环绕通知内发生的异常，如果捕获，需要再次抛出，方便下一个AfterThrowing捕获
    throw new RuntimeException("环绕内发生了异常");
} finally {
    System.out.println("[环绕结束通知]...");
}

```

需要注意的一点是，如果环绕通知外还有异常通知，需要将异常再次抛出，否则外边的异常通知会失效。



### 有环绕通知的执行顺序

以上面的栗子为说明，如果具有环绕通知和其他通知，执行结果如下：

对比没有环绕通知的执行顺序，其实就是把【正常方法执行】这一步骤换成【环绕通知方法执行】,并且环绕通知的优先级高于其他通知。

```java
=========方法正常结束===========
[环绕通知]前置...
方法执行前before...
[环绕通知]后置...
[环绕结束通知]...
方法执行结束after...
方法正常返回AfterReturning...
=========方法发生异常===========
[环绕通知]前置...
方法执行前before...
[环绕中异常通知]...
[环绕结束通知]...
方法执行结束after...
方法发生异常AfterThrowing:环绕内发生了异常

```



### 多个切面的通知执行顺序

如果有多个切面切一个方法，切面类是有执行顺序的，其就像一个同心圆，由外到里，再由里到外地执行。

一般来说，会按照切面类类命的首字母来决定执行顺序。但是，可以通过@Order注解来指定优先级。值越小的优先级越大。如果是xml的配置方式，可以通过在xml文件中配置切面类的顺序来控制切面类的执行顺序，当然也可以通过order属性来指定。



### JoinPoint获取目标方法的信息

* 除了环绕通知之外的其他通知，可以在切面方法中使用JoinPoint来获取目标方法的信息。环绕通知可以使用ProceedingJoinPoint来获取目标方法的信息。JointPoint是Spring内置的，无需在注解中指定。

* ```java
  public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
      Object[] args = joinPoint.getArgs();
  }
  ```

* 对于@AfterReturning，还可以在注解中使用returning指定一个变量，并将变量绑定到方法参数，即可接收目标方法返回值

* ```java
  @AfterReturning(value = "execution(public int com.imooc.demo3.Calculator.*(..))",returning = "result")
  public void afterReturning(JoinPoint joinPoint,Object result){
      // 获取方法名
      Signature signature = joinPoint.getSignature();
  }
  ```

* 对于@AfterThrowing，还可以在注解中使用throwing指定一个变量，并将变量绑定到方法参数，即可接收目标方法异常信息

* ```java
  @AfterThrowing(value = "execution(public int com.imooc.demo3.Calculator.*(..))",throwing = "e")
  public void afterThrowing(Exception e){
      System.out.println("方法发生异常AfterThrowing:"+e.getMessage());
  }
  ```

* 直接输出JoinPoint可以获得切入点表达式的信息



### PointCut抽取共同的连接点

随便定义一个空方法，使用@PointCut来定义切入点表达式

```java
@Pointcut(value = "execution(public int com.imooc.demo3.Calculator.*(..))")
private void pointCut(){};
```

使用这个共用的切入点表达式如下：

```java
@Before("pointCut()")
public void before(){
    System.out.println("方法执行前before...");
}
@AfterReturning(value = "pointCut()",returning = "result")
public void afterReturning(JoinPoint joinPoint,Object result){
    // 获取方法名
    Signature signature = joinPoint.getSignature();
    System.out.println("方法正常返回AfterReturning...");
}
```

**注意，如果想在其他类使用这个切入点表达式，需要写全限定类名。**

```java
@Before("com.imooc.demo3.Calculator.pointCut()")
public void before(){
    System.out.println("方法执行前before...");
}
```



### xml配置AOP

* 将被代理的对象和切面类加入到IOC容器中
* 配置AOP切面类，切入点等信息

**需要注意的是配置通知方法时，不需要加()。比如before而不是before()**

```xml
<!--被代理对象加入容器-->
<bean id="calculator" class="com.imooc.demo3.Calculator"/>

<!--切面类加入容器-->
<bean id="logUtils" class="com.imooc.demo3.LogUtils"/>

<aop:config>
    <!--配置在<aop:aspect>标签外，可以全局使用-->
    <aop:pointcut id="pointCut" expression="execution(public int com.imooc.demo3.Calculator.*(..))"/>
    <!--指定切面类-->
    <aop:aspect ref="logUtils">
        <aop:before method="before" pointcut-ref="pointCut"/>
        <aop:after method="after" pointcut-ref="pointCut"/>
        <aop:after-returning method="afterReturning" pointcut-ref="pointCut" returning="result"/>
        <aop:after-throwing method="afterThrowing" pointcut-ref="pointCut" throwing="e"/>
        <aop:around method="around" pointcut-ref="pointCut"/>
    </aop:aspect>
</aop:config>
```





### 踩坑1：AOP不生效？

```java
@ContextConfiguration("classpath:applicationContext.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class CalculatorTest {
    @Test
    public void calculatorTest(){
        // 踩坑1：必须从ioc容器中取出对象才可以
        Calculator calculator = new Calculator();
        System.out.println(calculator.add(1, 2));
        System.out.println(calculator.div(2, 0));
    }
}

```

执行以上单元测试代码，AOP好像不生效。原因在于第7行代码，Calculator不是从容器中获取的，自然不能使用Spring提供的AOP功能。

### 踩坑2：IOC容器中保存的是代理对象

从坑1爬出来之后，自然想到注入Calculator。

```java
@ContextConfiguration("classpath:applicationContext.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class CalculatorTest {

    @Autowired
    private Calculator calculator;

    @Test
    public void calculatorTest(){
        System.out.println(calculator.add(1, 2));
        System.out.println(calculator.div(2, 0));
    }
}
```

此时执行会报错：

```java
org.springframework.beans.factory.BeanNotOfRequiredTypeException: Bean named 'calculator' is expected to be of type 'com.imooc.demo3.Calculator' but was actually of type 'com.sun.proxy.$Proxy26'
```

可以看到，**AOP的底层是代理，Calculator经过AOP代理之后**，真正保存的是com.sun.proxy.$Proxy26代理对象。可以通过以下代码验证：

```java
System.out.println(calculator);// com.imooc.demo3.Calculator@69c79f09
System.out.println(calculator.getClass());// class com.sun.proxy.$Proxy26
```

而代理对象和被代理对象，唯一的共同点就是实现了同一个接口Calculete。我们使用接口接收注入的对象：

```java
@Autowired
private Calculate calculator;
```

### 踩坑3：基本类型引发的错误

在注入Calculate接口后，本以为会一帆风顺，但是结果往往很现实：

```java
Null return value from advice does not match primitive return type for: public abstract int com.imooc.demo3.Calculate.add(int,int)
```

发生这个错误的原因在于，Calculate接口规定的add方法返回值是int的基本类型，而切面在执行before方法时，返回值类型是void，切面会返回null类型。这就导致了基本类型和null不可兼容的错误。

```java
 @Before("execution(public Integer com.imooc.demo3.Calculator.*(..))")
    public void before(){
        System.out.println("方法开始");
    }
```

一种简单的解决方案是将Calculate接口的返回值类型修改为包装类，因为包装类与null是兼容的。**同时要注意，修改了接口的返回值，要检查切入点表达式是否满足切入条件。**

