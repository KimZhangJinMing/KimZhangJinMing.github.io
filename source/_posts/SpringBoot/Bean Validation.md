---
title: Bean Validation.md
date: 2020-10-03 14:31:35
categories: SpringBoot
tags: 校验
---

### JSR303 参数校验

参数校验提供了一些简单的注解，可以对前端传递过来的参数进行验证，并且自定义异常信息。常用的注解有：

* @Min(value="",message="")
* @Max
* @Range
* @Length



### 开启参数校验 

> 验证简单的参数

* 在类(Controller)上添加@Validated，开启参数校验
* 在方法的参数上使用@Min等验证注解

> 验证DTO中的参数

* 在方法参数上(一般是Controller中的方法)添加@Validated，开启参数校验
* 在DTO的成员变量上添加验证注解

> 级联校验

* 在方法参数上(一般是Controller中的方法)添加@Validated，开启参数校验
* 在验证的DTO中关联的DTO上加@Valid，开启级联校验
* 在关联的DTO中添加验证注解

**需要说明的一点是，DTO是用服务器用来接收前端传递过来的参数的，可以通过把相关联的DTO属性加入到同一个DTO，从而消除级联的校验。**



### @Validated与@Valid的区别：

* 它们具有一定的相似性，并不是互斥的关系，在一定情况下可以互换使用
* @Valid是Java标准，而@Validated是Spring对@Valid的扩展
* 一般情况下，@Validate用于开启参数校验，@Valid用于开启级联校验



### 自定义校验注解

1. 自定义一个注解，加上模板方法groups、payloads

   ```java
   Class<?>[] groups() default {};
   
   Class<? extends Payload>[] payload() default {};
   ```

2. 注解上添加@Constraint(validatedBy = xxxx.class)，关联一个类，用来编写校验逻辑

3. 定义一个类，实现ConstraintValidator<T,R>接口，**泛型T表示自定义注解的类型，*泛型R表示自定义注解修饰的目标的类型**。

4. 实现isValid的方法，返回boolean值标志是否通过校验

5. 如果需要获取注解中定义的参数，要实现initialize方法，将注解的数据赋值给成员变量，这样在isValid方法中就可以获取注解中的信息进行判断

#### 一个自定义校验注解的栗子

自定义一个注解，加上模板方法groups、payloads，注解上添加@Constraint(validatedBy = xxxx.class)，关联一个类，用来编写校验逻辑。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Constraint(validatedBy = PasswordEqualValidator.class)
public @interface PasswordEqual {

    int min();

    int max();

    String message() default "password is not equal";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
```

定义一个类编写校验的相关逻辑，T是自定义注解的类型，R是自定义注解修饰的目标的类型。

```java
public class PasswordEqualValidator implements ConstraintValidator<PasswordEqual, PersonDTO> {

    private int min;

    private int max;

    @Override
    public boolean isValid(PersonDTO personDTO, ConstraintValidatorContext constraintValidatorContext) {
        boolean flag = true;
        String password1 = personDTO.getPassword1();
        String password2 = personDTO.getPassword2();
        if(!password1.equals(password2)){
            flag = false;
        }
        if(password1.length() < min){
            flag = false;
        }
        if(password2.length() > max){
            flag = false;
        }
        return flag;
    }

    @Override
    public void initialize(PasswordEqual constraintAnnotation) {
        this.min = constraintAnnotation.min();
        this.max = constraintAnnotation.max();
    }
}
```

完成自定义注解的编写后，在PersonDTO中使用：

**记得开启参数校验，参考上面“开启参数校验”这一章节。**

```java
@Builder
@Getter
@ToString
@PasswordEqual(message = "密码错误",min = 2,max = 4)
public class PersonDTO {
    @Length(min = 2,max = 7)
    private String name;
    @Min(value = 7)
    private Integer age;

    private String password1;
    private String password2;
}
```



### 参数校验结合全局异常处理

参数校验分为两种情况：

* POST请求通过body传递的参数
* GET请求通过url传递的参数

> POST Body 传递的参数校验异常处理

1. 抛出MethodArgumentNotValidException
2. 存在校验有多个错误信息的可能，需要将所有的错误信息拼接成字符串返回
3. List<ObjectError>中getDefaultMessage可以获取到错误信息

```java
/**
* <p>处理参数校验异常</p>
* 1.参数校验异常使用统一的HTTP状态码 400
* 2.所有的参数校验异常都是抛出MethodArgumentNotValidException
* 3.如果校验有多个错误,将所有的错误信息用分号分割
*
* @param request the request of http
* @param e       the exception that method throw
* @return the unify response message
*/
@ExceptionHandler(MethodArgumentNotValidException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ResponseBody
public UnifyResponse handleMethodArgumentNotValidException(HttpServletRequest request, MethodArgumentNotValidException e) {
    Map<String, String> requestInfo = getRequestUrlAndMethod(request);
    System.out.println(e);
    // get exception message
    List<ObjectError> errors = e.getBindingResult().getAllErrors();
    String message = getBeanValidationMessage(errors);
    return new UnifyResponse(10001, message, requestInfo.get("url"), requestInfo.get("method"));
}

/**
* 将多个错误拼接成以分号分割的字符串返回
*
* @param errors the errors list that exception throws
* @return the message about the exception
*/
private String getBeanValidationMessage(List<ObjectError> errors) {
    return errors.stream()
        .map(DefaultMessageSourceResolvable::getDefaultMessage)
        .collect(Collectors.joining(";"));
}
```

> 通过url传递的参数校验异常处理

一是使用SpingBoot默认的message

```java
/**
     * <p>处理url参数传递校验异常</p>
     * 1.参数校验异常使用统一的HTTP状态码 400
     * 2.所有的参数校验异常都是抛出ConstraintViolationException
     * 3.有多种获取异常信息的方式，这里采用第二种
     *  1.遍历e.getConstraintViolations对每一个异常进行处理，可以获取更多异常信息
     *  2.利用e.getMessage已经处理好的异常信息
     * @param request the request of http
     * @param e       the exception that method throw
     * @return the unify response message
     */
@ExceptionHandler(ConstraintViolationException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ResponseBody
public UnifyResponse handleConstraintViolationException(HttpServletRequest request, ConstraintViolationException e) {
    Map<String, String> requestInfo = getRequestUrlAndMethod(request);
    System.out.println(e);
    return new UnifyResponse(99997, e.getMessage(), requestInfo.get("url"), requestInfo.get("method"));
}

```

二是自己处理的message：

```java
private String getConstraintViolationMessage(ConstraintViolationException e){
    String message = "";
    Set<ConstraintViolation<?>> constraintViolations = e.getConstraintViolations();
    for (ConstraintViolation<?> constraintViolation : constraintViolations) {
        String path = constraintViolation.getPropertyPath().toString();
        String[] names = path.split("[.]");
        String name = names[1];
        message =  name + ":" + constraintViolation.getMessage();
    }
    return message;
}
```





### 抽取出message配置模板

在resources目录下，创建一个ValidationMessages.properties文件(规定了只能是这个文件)。

**注意，在这里用{}来引用配置文件中的key。在配置文件中，也能使用{}来引入注解中的值。**

```java
id.positive=id不能为负数{max}{min}
name.notBlank=参数不能为空
```

```java
@RequestMapping(value = "name/{name}")
public Banner getByName(@PathVariable @NotBlank(message = "{name.notBlank}") String name) {
    Banner banner = bannerService.getByName(name);
    if (banner == null) {
        throw new NotFoundException(30005);
    }
    return banner;
}
```

