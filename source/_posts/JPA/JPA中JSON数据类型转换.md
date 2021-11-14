---
title: JPA中JSON数据类型转换.md
date: 2020-08-10 22:46:35
categories: JPA
tags: 序列化，工具类
---

当我们在数据表中存储的是json字符串的时候，映射到实体类的时候，Java中只有String类型能和json字符串对应。

那么，如果我们想把json字符串反序列化成对象,有什么办法嘛？

> 利用Getter、Setter方法

在序列化、反序列化的时候，都要调用实体类的利用Getter、Setter方法，我们可以利用这个特点，在Getter方法中进行反序列化，在Setter方法中进行序列化。当然，序列化的方法可以使用Jackson或者FastJson。

缺点：

* 如果某个类中有需要序列化、反序列化的属性，那就得修改这个类的Getter、Setter方法，无法做到通用。
* 有一种观点认为不应该在实体类中写业务逻辑

>利用JPA的Converter

我们利用JPA的converter，并在实体类中指定要使用的converter，达到序列化和反序列化的效果。

可以将实体的属性反序列化为Map<String,Object>和List<Object>这种数据结构，也可以反序列化为一个具体的实体类。我们以HashMap来举一个栗子。

* 编写converter需要实现AttributeConverter<T,K>接口，T是Java实体中的属性类型，K是数据表中字段的数据类型
* 需要加上@Converter注解
* Jackson序列化主要调用writeValueAsString方法，反序列化主要调用readValue方法
* 使用SpringBoot内置的Jackson进行序列化，可以通过依赖注入的方式注入ObjectMapper

```java
/**
 * @Author Ming
 * @Date 2020/06/13 19:02
 * @Description 单体JSON对象映射工具类
 */
@Converter
public class MapAndJson implements AttributeConverter<Map<String, Object>, String> {

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public String convertToDatabaseColumn(Map<String, Object> attribute) {
        if(CollectionUtils.isEmpty(attribute)){
            return "";
        }
        try {
            return objectMapper.writeValueAsString(attribute);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(99999);
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public Map<String, Object> convertToEntityAttribute(String dbData) {
        if(StringUtils.isBlank(dbData)){
            return Collections.EMPTY_MAP;
        }
        try {
            return objectMapper.readValue(dbData, HashMap.class);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(99999);
        }
    }
}
```

实体类中使用@convert注解标明要使用的converter。

```java
 @Convert(converter = MapAndJson.class)
 private Map<String,Object> specs;
```

缺点：

* 使用Map和List这种结构，无法调用业务类的方法。例如List<Spec>可以调用Spec这个类的业务方法。而使用List<Object>无法调用类的业务方法。
* 如果有多个实体类需要序列化、反序列化，需要编写多个Converter

我们追求一种通用的写法，可不可以利用泛型呢？

上面的Converter方法中可以使用泛型来达到一种通用的效果，但是Java中泛型是有缺点的。主要体现在反序列化的第二个参数，如果直接使用T.class是不行的。所以我们需要一种机制可以将Class传入到Converter中，但是JPA的Converter无法做到这一点。

```java
return objectMapper.readValue(dbData, HashMap.class);
```

我们尝试自己编写一个工具类来实现序列化和反序列化的功能。这样就可以通过传参的方式传入Class。

```java
/**
 * @Author Ming
 * @Date 2020/06/13 21:58
 * @Description 序列化与反序列化工具类,支持泛型
 */
@Component
public class GenericAndJson<T> {

    private static ObjectMapper objectMapper;

    @Autowired
    public void setObjectMapper(ObjectMapper objectMapper) {
        GenericAndJson.objectMapper = objectMapper;
    }

    /**
     * 序列化
     *
     * @param o   需要转换成json字符串的对象
     * @param <T> 目标对象类型
     * @return json字符串,如果是空,返回""
     */
    public static <T> String objectToJson(T o) {
        if (o == null) {
            return "";
        }
        try {
            return GenericAndJson.objectMapper.writeValueAsString(o);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(99999);
        }
    }

    /**
     * 反序列化
     *
     * @param s    json字符串
     * @param type typeReference对象
     * @param <T>  目标对象类型
     * @return 转换后的目标对象,如果是空,数组返回[],对象返回{}
     */
    @SuppressWarnings("unchecked")
    public static <T> T jsonToObject(String s, TypeReference<T> type) {
        if (StringUtils.isBlank(s)) {
            return type.getType().getTypeName().contains("java.util.List")
                    ? (T) Collections.EMPTY_LIST : (T) Collections.EMPTY_MAP;
        }
        try {
            return GenericAndJson.objectMapper.readValue(s, type);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(99999);
        }
    }

    /**
     * json字符串转换成List,将List<T>中的T当成泛型,但这种方式还是把T转换成了LinkedHashMap
     *
     * @param s   json字符串
     * @param <T> 目标对象类型
     * @return 转换后的目标对象
     */
   /* public static <T> List<T> jsonToList(String s) {
        if (StringUtils.isBlank(s)) {
            return Collections.emptyList();
        }
        try {
            List<T> list = GenericAndJson.objectMapper.readValue(s, new TypeReference<List<T>>() {
            });
            return list;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(99999);
        }
    }*/
}

```

这里因为注入了ObjectMapper对象，所以需要加上@Component注解。同时，这里还巧妙的利用了setter方法注入static的对象。

同时，在实体类中需要使用Getter、Setter方法来处理。

```java
	// ...省略
	private String specs;
	
	/**
     * 数据库中获取的字符串对象反序列化
     *
     * @return 反序列化后的对象
     */
    public List<Spec> getSpecs() {
        if (StringUtils.isBlank(this.specs)) {
            return Collections.emptyList();
        }
        return GenericAndJson.jsonToObject(this.specs, new TypeReference<List<Spec>>() {
        });
    }

    /**
     * 序列化对象保存到数据库
     *
     * @param specs 对象
     * @return json字符串
     */
    public String setSpecs(List<Spec> specs) {
        if (specs == null || specs.isEmpty()) {
            return "";
        }
        return GenericAndJson.objectToJson(specs);
    }
```



上面的工具类是比较通用的工具类，但是还不够好。

缺点：

* 需要使用TypeReference来传入参数

  

注释掉的方法是对不需要传入TypeReference所做的尝试。

一种思路是把List<T>当成一个泛型T

一种思路是把List<T>中的T当成一个泛型T。这种方法看似可行，但是DEBUG发现，传入的实体类并没有生效，而是使用了LinkedHashMap来实现的，这就达不到可以调用实体类的业务方法的期望。