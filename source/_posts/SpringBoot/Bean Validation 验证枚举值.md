---
title: Bean Validation 验证枚举值.md
date: 2020-11-02 21:43:35
categories: SpringBoot
tags: 校验
---

### 自定义注解验证枚举类型的值

使用场景：在前后端分离的开发中，如果后端的一个参数对应的是一个枚举类型的值，前端是无法传递枚举值的。

只能通过与后端约定好的格式进行传递。比如后端定义枚举BooleanEumn：1 -> 是，0 -> 否。

但如果前端不按照约定传参，比如传值2，就会发生错误。

此时，可以通过自定义BeanValidation的注解来校验枚举值。需要指定枚举类和校验的方法即可。

### 自定义注解

1. 要求校验方法是静态方法(反射无法调用枚举的实例方法)。如果想使用实例方法也行，需要修改反射的invoke方法。

静态方法：Boolean result = (Boolean)method.invoke(null, value);

实例方法：Boolean result = (Boolean)method.invoke(enumClass.newInstance(), value

2. 原则上校验值的类型要和校验方法的类型一致，都为包装类型。但在Validator中做了兼容，如果存在普通类型形参的校验方法，也可以被调用。

枚举类：

```java
public enum BooleanEnum {
    //
    NO(0,"否"),
    YES(1,"是");

    private int value;

    public static boolean existed(int value){
        return Stream.of(BooleanEnum.values()).anyMatch(t -> t.value == value);
    }

    BooleanEnum(int value,String description){
        this.value = value;
    }
}
```

自定义校验注解：使用内部类的方式实现Validator

```java
/**
 * @Author Ming
 * @Date 2020/11/01 21:04
 * @Description 校验是否是枚举值
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = EnumValue.EnumValueValidator.class)
public @interface EnumValue {

    String message() default "参数有误";

    Class<? extends Enum<?>> enumClass();

    String enumMethod();

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    class EnumValueValidator implements ConstraintValidator<EnumValue,Object>{

        Class<? extends Enum<?>> enumClass;

        String enumMethod;

        @Override
        public void initialize(EnumValue constraintAnnotation) {
            this.enumClass = constraintAnnotation.enumClass();
            this.enumMethod = constraintAnnotation.enumMethod();
        }

        @Override
        public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
            // 标记注解的值为空,不进行校验
            if(value == null){
                return true;
            }
            // 没有指定注解的校验类不进行校验
            if(enumClass == null){
                return true;
            }


            // 获取校验值的class
            Class<?> valueClass = value.getClass();

            try {
                Method method = null;
                try {
                    // 反射获取方法不会进行自动拆装箱,如果不存在与valueClass相同类型的形参会报错
                    method = enumClass.getMethod(enumMethod, valueClass);
                } catch (NoSuchMethodException e) {
                    // 获取不到包装类型的方法,尝试获取基本类型参数的方法
                    method = enumClass.getMethod(enumMethod,transPrimitiveType(valueClass));
                }
                // 判断是不是静态方法
                if(!Modifier.isStatic(method.getModifiers())){
                    throw new HttpException(30006);
                }

                // 校验方法的返回值
                Class<?> returnType = method.getReturnType();

                // 如果校验方法的返回值不是boolean类型或Boolean,抛出异常
                if(!Boolean.TYPE.equals(returnType) || !boolean.class.equals(returnType)){
                    throw new HttpException(30001);
                }
                Boolean result = (Boolean)method.invoke(null, value);
                // 避免Boolean返回的是null
                return result == null ? false : result;
            } catch (Exception e) {
                throw new HttpException(30005);
            }

        }

        /**
         * 由于实现ConstraintValidator<EnumValue,Object>接口中指明注解的值是Object类型
         * 也就是参数值的Class一定是包装类型的,如果验证方法中存在基本类型的方法,通过反射获取方法会抛出异常
         * 此方法对包装类的Class进行转换基本类型,这样在验证方法中可以调用基本类型和包装类型的方法
         * @param valueClass 包装类型
         * @return 基本类型
         */
        private Class<?> transPrimitiveType(Class<?> valueClass){
            if(Integer.class.equals(valueClass)){
                return int.class;
            }else if(Boolean.class.equals(valueClass)){
                return boolean.class;
            }else if(Byte.class.equals(valueClass)){
                return byte.class;
            }else if(Short.class.equals(valueClass)){
                return Short.class;
            }else if(Long.class.equals(valueClass)){
                return long.class;
            }else if(Float.class.equals(valueClass)){
                return float.class;
            }else if(Double.class.equals(valueClass)){
                return double.class;
            }else if(Character.class.equals(valueClass)){
                return char.class;
            }else{
                return null;
            }

        }
    }

}

```

使用：

```java
@Positive
@NotNull
private Long spuId;
```



