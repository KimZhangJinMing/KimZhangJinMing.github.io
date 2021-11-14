







### 3.整合视图层

***

#### 3.1 整合Freemarker

SpringBoot提供了Freemarker的自动配置类。只需要将freemark依赖导入，就可以使用freemarker.

查看`FreeMarkerProperties`,只需要把模板文件放在templates目录下，且文件以.ftlh结尾就可以。

```java
@ConfigurationProperties(prefix = "spring.freemarker")
public class FreeMarkerProperties extends AbstractTemplateViewResolverProperties {

	public static final String DEFAULT_TEMPLATE_LOADER_PATH = "classpath:/templates/";

	public static final String DEFAULT_PREFIX = "";

	public static final String DEFAULT_SUFFIX = ".ftlh";
}
```

当然，也可以通过配置文件修改默认的配置，如以下将文件后缀名修改为ftl：

```yml
spring:
  freemarker:
    suffix: .ftl
```

#### 3.2 整合Thymeleaf



### 4.JSON

***

所有的JSON生成都离不开相关的HttpMessageConverter.

HttpMessageConverter主要有两大功能：

* 将服务端返回的对象序列化成JSON字符串
* 将前端传来的JSON字符串反序列化成Java对象

SpringBoot中自动配置了Jackson和Gson的HttpMessageConverter.

所以，如果用户使用Jackson和Gson的话，如果没有其他额外的配置，则只需要添加依赖即可。

#### 4.1 Jackson

如果引入了spring-boot-starter-web模块的话，SpringBoot默认会引入Jackson依赖。

![Jackson依赖](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210107214534564.png)

Jackson的自动配置类：`JacksonHttpMessageConvertersConfiguration`

```java
@Configuration(proxyBeanMethods = false)
class JacksonHttpMessageConvertersConfiguration {

	@Configuration(proxyBeanMethods = false)
    // 如果导入了jsckson的依赖，条件成立，SpringBoot会自动配置一个MaapingJackson2HttpMessageConverter
	@ConditionalOnClass(ObjectMapper.class) 
	@ConditionalOnBean(ObjectMapper.class)
	@ConditionalOnProperty(name = HttpMessageConvertersAutoConfiguration.PREFERRED_MAPPER_PROPERTY,
			havingValue = "jackson", matchIfMissing = true)
	static class MappingJackson2HttpMessageConverterConfiguration {
		
        // 如果没有提供MappingJackson2HttpMessageConverter的类,SpringBoot默认为我们提供MappingJackson2HttpMessageConverter，它注入一个ObjectMapper,而ObjectMapper是由JacksonAutoConfiguration类提供的
		@Bean
		@ConditionalOnMissingBean(value = MappingJackson2HttpMessageConverter.class,
				ignoredType = {			"org.springframework.hateoas.server.mvc.TypeConstrainedMappingJackson2HttpMessageConverter",
						"org.springframework.data.rest.webmvc.alps.AlpsJsonHttpMessageConverter" })
		MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
			return new MappingJackson2HttpMessageConverter(objectMapper);
		}

	}
}

```

测试小栗子：只需要引入jackson依赖，如果没有其他额外的配置，则可直接使用。

```java
@RestController
public class JsonController {

    @GetMapping("/json")
    public List<Map<String, Object>> json() {
        ArrayList<Map<String,Object>> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            Map<String, Object> map = new HashMap<>();
            map.put("id",i);
            map.put("name","xiaoming"+i);
            map.put("date",new Date());
            list.add(map);
        }
        return list;
    }
}
```

测试结果：

```json
[
  {
    "date": "2021-01-07T14:00:00.094+0000",
    "name": "xiaoming0",
    "id": 0
  },
  {
    "date": "2021-01-07T14:00:00.094+0000",
    "name": "xiaoming1",
    "id": 1
  },
  {
    "date": "2021-01-07T14:00:00.094+0000",
    "name": "xiaoming2",
    "id": 2
  },
  {
    "date": "2021-01-07T14:00:00.094+0000",
    "name": "xiaoming3",
    "id": 3
  },
  {
    "date": "2021-01-07T14:00:00.094+0000",
    "name": "xiaoming4",
    "id": 4
  }
]
```

> 日期格式化

1.如果是在实体类中，可以使用@JsonFormat(pattern="yyyy-MM-dd")标记在实体属性上。这种方式有一个弊端，如果有多个实体类，需要在每一个实体类的属性上都添加注解

2.自动提供一个MappingJackson2HttpMessageConverter,提供日期格式化的功能

3.自己提供一个ObjectMapper,提供日期格式化的功能

```java
// 方式一:自己提供一个mappingJackson2HttpMessageConverter，本质上也就是设置了ObjectMapper
@Configuration
public class JsonConfiguration {

    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
        return new MappingJackson2HttpMessageConverter(objectMapper);
    }

  
}
```

```java
// 方式二: 既然mappingJackson2HttpMessageConverter也是设置ObjectMapper,那我提供一个自己的ObjectMapper即可
@Configuration
public class JsonConfiguration {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return objectMapper;
    }
}
```

```java
// 方式三： 结合方式一和方式二
@Configuration
public class JsonConfiguration {
	
    // 使用注入的objectMapper来构建MappingJackson2HttpMessageConverter
    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
        return new MappingJackson2HttpMessageConverter(objectMapper);
    }
	
    // 自己构建ObjectMapper
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return objectMapper;
    }
}

```

测试结果：

```json
[
  {
    "date": "2021-01-07 22:16:57",
    "name": "xiaoming0",
    "id": 0
  },
  {
    "date": "2021-01-07 22:16:57",
    "name": "xiaoming1",
    "id": 1
  },
  {
    "date": "2021-01-07 22:16:57",
    "name": "xiaoming2",
    "id": 2
  },
  {
    "date": "2021-01-07 22:16:57",
    "name": "xiaoming3",
    "id": 3
  },
  {
    "date": "2021-01-07 22:16:57",
    "name": "xiaoming4",
    "id": 4
  }
]
```



#### 4.2 Gson

SpringBoot也提供了Gson的`GsonHttpMessageConverter`

如果无需其他的配置,只要排除默认的Jackson依赖，引入Gson依赖即可使用。

Gson版本无需指定，由SpringBoot控制。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
</dependency>
```

`GsonHttpMessageConverter`

```java
@Configuration(proxyBeanMethods = false)
// 如果不存在Gson的类话，自动配置生效
@ConditionalOnClass(Gson.class)
class GsonHttpMessageConvertersConfiguration {

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnBean(Gson.class)
    @Conditional(PreferGsonOrJacksonAndJsonbUnavailableCondition.class)
    static class GsonHttpMessageConverterConfiguration {

        // 提供GsonHttpMessageConverter
        @Bean
        @ConditionalOnMissingBean
        GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
            GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
            converter.setGson(gson);
            return converter;
        }
    }
}
```

> 日期格式化

```java
@Configuration
public class JsonConfiguration {

    @Bean
    public GsonHttpMessageConverter gsonHttpMessageConverter() {
        Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd HH:mm:ss").create();
        return new GsonHttpMessageConverter(gson);
    }
}

```



#### 4.3 FastJson

SpringBoot没有提供FastJsonHttpMessageConverter，如果需要使用FastJson,需要自己提供HttpMessageConverter.

排除默认的Jackson依赖，引入FastJson依赖。

FastJson版本不受SpringBoot控制，需要自己引入版本。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.73</version>
</dependency>
```

编写FastJson的配置类：

```java
@Configuration
public class JsonConfiguration {

    @Bean
    public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        return new FastJsonHttpMessageConverter();
    }
}
```

> 日期格式化

```java
@Configuration
public class JsonConfiguration {

    @Bean
    public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd HH:mm:ss");
        converter.setFastJsonConfig(config);
        return converter;
    }
}
```

**在测试3中JSON格式化的过程中，发现每一种JSON对日期的默认处理方式都不同。**



### 5.静态资源访问

***

#### 5.1 默认的静态资源访问规则

SpirngBoot中，和Web有关的配置类是`WebMvcAutoConfiguration`

其中，和静态资源访问有关的方法是addResourceHandlers

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    // 缓存相关
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    // 重点，这里返回的staticPathPattern是/**
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

this.resourceProperties.getStaticLocations()返回的是静态资源存放的位置：
在ResourceProperties类中定义的常量数组，也就是说，放在这4个目录下的静态资源，SpringBoot可以直接访问到。

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };
```

getResourceLocations()方法将默认的4个目录作为参数传递进去，加上SERVLET_LOCATIONS这么一个路径返回。

SERVLET_LOCATIONS是定义的一个常量/，代表着webapp的目录，也就是说，现在springboot默认能访问到的存放静态资源的路径有5个。

且它们的优先级是按照数组的顺序。

![image-静态资源优先级](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210107232317191.png)

```java
static String[] getResourceLocations(String[] staticLocations) {
    String[] locations = new String[staticLocations.length + SERVLET_LOCATIONS.length];
    System.arraycopy(staticLocations, 0, locations, 0, staticLocations.length);
    System.arraycopy(SERVLET_LOCATIONS, 0, locations, staticLocations.length, SERVLET_LOCATIONS.length);
    return locations;
}
```

#### 5.2 自定义资源位置

> 通过配置文件指定

一旦在配置文件中指定了static-locations，springboot默认的4个目录下的位置会被配置文件中指定的替换，也就是说CLASSPATH_RESOURCE_LOCATIONS数组下指定的静态资源文件存放路径不再生效。

```yml
spring:
  # 指定静态资源存放路径
  resources:
    static-locations: classpath:/ming/
```

> 通过配置类指定

需要实现WebMvcConfigurer接口，重写addResourceHandlers方法。

addResourceHandler对应着配置文件中的spring.mvc.static-pattern。

addResourceLocations对应着配置文件中的spring.resource.static-location。

```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
        .addResourceLocations("classpath:/ming/");
    }
}
```





### 6. 文件上传

***

Web文件上传通常有2种方案：

* CommonMultipartResolver(需要引入common-fileupload组件)

* StandardServletMultipartResolver(需要Servlet 3.0以上才支持)

SpringBoot使用的是StandardServletMultipartResolver。

> 配置文件上传相关属性

可以直接在配置文件中指定与文件上传相关的一些属性，以最大上传文件大小为例。

如果文件超出限制大小，后端报错The field file exceeds its maximum permitted size of 1024 bytes.

此时断点都进不去，由于文件超出限制大小，请求被拦截了。

```yml
spring:
  servlet:
    multipart:
      max-file-size: 1KB  # 大写
```

#### 4.1 单文件上传

文件上传后端处理需要注意的问题：

* 确保保存目录的存在，否则会报FileNotFoundException
* 文件名称相同的处理
* 静态资源访问的处理，如果自定了静态资源访问路径，要注意自定义的静态资源访问路径覆盖springboot默认的静态资源访问路径的问题

```java
@RestController
public class FileController {
	
    // 这里格式化的方式要和返回的路径和保存的路径拼接成最终的字符串，注意格式
    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("/yyyy/MM/dd/");
	
    // 文件上传必须是post请求，get请求没有请求体
    @PostMapping("/upload")
    public String upload(HttpServletRequest request, MultipartFile file) throws IOException {
        String formatDate = LocalDate.now().format(formatter);
        // 文件要保存的路径，实际上是webapp目录下，而webapp目录默认是静态资源可以访问的路径，可以通过浏览器直接访问文件
        String savePath = request.getServletContext().getRealPath("/img") + formatDate;
        // 如果目录不存在，需要创建目录，否则报错FileNotFoundException
        File dest = new File(savePath);
        if(!dest.exists()){
            dest.mkdirs();
        }
        // 处理文件名称 uuid+后缀名
        String uuid = UUID.randomUUID().toString().replace("-", "");
        String originalFilename = file.getOriginalFilename();
        String fileName = uuid + originalFilename.substring(originalFilename.lastIndexOf("."));
        // 保存文件到dest目录
        file.transferTo(new File(dest, fileName));
        // 返回文件路径
        return request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + "/img" + formatDate + fileName;
    }
}
```

从后端接收到的file来看，使用的是StandardServletMultipartResplver:

![后端接收到的file](C:\Users\zjm16\AppData\Roaming\Typora\typora-user-images\image-20210109125716984.png)



>  前端通过表单上传文件

需要注意：

* name属性的名称要和后端参数的名称一致
* 需要指定enctype

```html
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" id="file" name="file">
    <input type="submit" value="保存">
</form>
```

> 通过Ajax方式上传文件

$("#file")[0]是将jq对象转换成DOM对象，由于是单文件上传，files[0]取出第一个文件。

`processData、contentType`属性的设置是非常重要的。

```javascript
<div id="msg"></div>
<input type="file" id="file" name="file">
<input type="button" value="保存" onclick="uploadFile()">
<script>
    function uploadFile(){
        var file = $("#file")[0].files[0];
        var formData = new FormData();
        formData.append("file",file);
        $.ajax({
            url:"/upload",
            type:"post",
            data: formData,
            // 告诉JQuery不要对数据进行处理
            processData: false,
            // 告诉JQuery不要设置content-type
            contentType: false,
            success: function (result) {
                $("#msg").text(result)
            }
        })
    }

</script>
```



#### 4.2 多文件上传

多文件上传有2中情况，处理请求的方式也不同：

* 一个input框，上传多个文件(后端接收一个MultipartFile数组，循环保存)
* 多个input框(后端绑定多个MultipartFile参数，对应前端的多个input框的name属性)

以下以一个input框,上传多个文件为例子。

```java
@RestController
public class FileController {

    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("/yyyy/MM/dd/");

    @PostMapping("/uploads")
    public List<String> uploads(HttpServletRequest request, MultipartFile[] files) throws IOException {
        System.out.println(files.length);
        String formatDate = LocalDate.now().format(formatter);
        // 文件要保存的路径，实际上是webapp目录下，而webapp目录默认是静态资源可以访问的路径，可以通过浏览器直接访问文件
        String savePath = request.getServletContext().getRealPath("/img") + formatDate;
        // 如果目录不存在，需要创建目录，否则报错FileNotFoundException
        File dest = new File(savePath);
        if(!dest.exists()){
            dest.mkdirs();
        }
        List<String> filePaths = new ArrayList<>();
        for (MultipartFile file : files) {
            // 处理文件名称 uuid+后缀名
            String uuid = UUID.randomUUID().toString().replace("-", "");
            String originalFilename = file.getOriginalFilename();
            String fileName = uuid + originalFilename.substring(originalFilename.lastIndexOf("."));
            // 保存文件到dest目录
            file.transferTo(new File(dest, fileName));
            String filePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + "/img" + formatDate + fileName;
            filePaths.add(filePath);
        }

        // 返回文件路径
        return filePaths;
    }
}

```

> 使用form表单：

`在file组件下添加multiple表示上传多文件`

```html
<form action="/uploads" method="post" enctype="multipart/form-data">
    <input type="file" id="files" name="files" multiple>
    <input type="submit" value="保存">
</form>
```

> 使用ajax:

`在file组件下添加multiple表示上传多文件`

通过formData传递参数到后台的方式非常讲究，经过测试，以下两种方式后台接收不到参数：

```javascript
// 方式一
var files = $("#files")[0].files;
for (var i = 0; i < files.length; i++) {
    formData.append("files[" + i + "]", files[i]);
}

// 方式二
var files = $("#files")[0].files;
formData.append("files",files);

// 正确的传递方式
var files = $("#files")[0].files;
for (var i = 0; i < files.length; i++) {
    formData.append("files", files[i]);
}
```

这里还需要注意的一点是FormData是一个特殊的对象，通过console.log打印是{}，而实际上它是有值的。 

```html
<div id="msg"></div>
<input type="file" id="files" name="files" multiple>
<input type="button" value="保存" onclick="uploadFile()">
<script>
    function uploadFile() {
        var files = $("#files")[0].files;
        var formData = new FormData();
        for (var i = 0; i < files.length; i++) {
            formData.append("files", files[i]);
        }
        
        $.ajax({
            url: "/uploads",
            type: "post",
            processData: false,
            contentType: false,
            data: formData,
            success: function (result) {
                $("#msg").text(result)
            }
        })
    }

</script>
```



#### 4.3 使用FastJson报错Content-Type cannot contain wildcard type '*'

在新版本的fastJson,FastJsonHttpMessageConverter初始化时，设置了MediaType

```java
public FastJsonHttpMessageConverter() {
    // MediaType.ALL是 /
    super(MediaType.ALL);
}
```

而在SpringBoot的AbstractHttpMessageConverter类中,write的过程中，addDefaultHeaders方法中:

```java
headers.setContentType(contentTypeToUse);
```

setContentType方法中检查了Content-type不能含有通配符：

```java
public void setContentType(@Nullable MediaType mediaType) {
    if (mediaType != null) {
        Assert.isTrue(!mediaType.isWildcardType(), "Content-Type cannot contain wildcard type '*'");
        Assert.isTrue(!mediaType.isWildcardSubtype(), "Content-Type cannot contain wildcard subtype '*'");
        set(CONTENT_TYPE, mediaType.toString());
    }
    else {
        remove(CONTENT_TYPE);
    }
}
```

解决办法是在配置FastJsonHttpMessageConverter时候，设置Content-type:

```java
@Configuration
public class JsonConfiguration {

    @Bean
    public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        List<MediaType> supportMediaTypes = this.getSupportMediaTypes();
        converter.setSupportedMediaTypes(supportMediaTypes);
        config.setDateFormat("yyyy-MM-dd HH:mm:ss");
        converter.setFastJsonConfig(config);
        return converter;
    }

    private List<MediaType> getSupportMediaTypes() {
        List<MediaType> supportedMediaTypes = new ArrayList<>();
        supportedMediaTypes.add(MediaType.APPLICATION_JSON);
        supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        supportedMediaTypes.add(MediaType.APPLICATION_ATOM_XML);
        supportedMediaTypes.add(MediaType.APPLICATION_FORM_URLENCODED);
        supportedMediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
        supportedMediaTypes.add(MediaType.APPLICATION_PDF);
        supportedMediaTypes.add(MediaType.APPLICATION_RSS_XML);
        supportedMediaTypes.add(MediaType.APPLICATION_XHTML_XML);
        supportedMediaTypes.add(MediaType.APPLICATION_XML);
        supportedMediaTypes.add(MediaType.IMAGE_GIF);
        supportedMediaTypes.add(MediaType.IMAGE_JPEG);
        supportedMediaTypes.add(MediaType.IMAGE_PNG);
        supportedMediaTypes.add(MediaType.TEXT_EVENT_STREAM);
        supportedMediaTypes.add(MediaType.TEXT_HTML);
        supportedMediaTypes.add(MediaType.TEXT_MARKDOWN);
        supportedMediaTypes.add(MediaType.TEXT_PLAIN);
        supportedMediaTypes.add(MediaType.TEXT_XML);
        return supportedMediaTypes;
    }
}

```



### 7.@ControllerAdvice

***

#### 7.1 全局异常处理

可以定义一个类，专门用来处理异常。在类上添加@ControllerAdvice注解，在方法上添加@ExceptionHandler(xxxException.class)注解，当发生异常的时候，会被这个方法捕获。

异常的处理方式可以返回一个错误页面，也可以通过response直接返回异常信息。

```java
@ControllerAdvice
public class GlobalController {

    @ExceptionHandler(value = MaxUploadSizeExceededException.class)
    public ModelAndView handlerFileException(MaxUploadSizeExceededException e) {
        ModelAndView model = new ModelAndView("error");
        model.addObject("error",e.getMessage());
        return model;
    }
}
```

#### 7.2 预设全局数据

可以在@ControllerAdvice标记的类中，配合@ModelAttribute返回一些全局通用的数据，任何一个controller都能获取该数据。

```java
@ControllerAdvice
public class GlobalController {
	
    @ModelAttribute(value = "info")
    public Map<String,Object> globalInfo() {
        Map<String,Object> map = new HashMap<>(2);
        map.put("info","hello");
        map.put("version","1.0");
        return map;
    }
}

```

在任意一个controller中都可以获取@ControllerAdvice中返回的数据：

key为@ModelAttribute指定的value值，value为@ModelAttrrbute方法的返回值

```java
@GetMapping("/info")
public Map<String,Object> globalInfo(Model model) {
    // 方式一 ，key=info,value=hashmap
    Map<String, Object> attribute = (Map<String, Object>) model.getAttribute("info");
    // 方式二
    Map<String, Object> map = model.asMap();
    Map<String, Object> attribute1  = (Map<String, Object>) map.get("info");
    return attribute1;
}
```



### 8.自定义异常处理

***

#### 8.1 默认的异常处理机制

> 静态错误页面

在static目录下新增目录error,在error目录下创建`状态码.html`文件，springBoot会根据状态码的不同返回不同的错误页面。

比如在error目录下创建404.html，500.html的页面，当出现404错误的时候，会返回404.html页面，当出现500错误的时候，会返回500.html页面。

这种做法很繁琐，状态码有很多，就意味着一个状态码对应着一个错误页面。因此，SpringBoot提供了一种模糊的匹配。

比如在error目录下创建4xx.html，5xx.html的页面，当出现4xx错误的时候，会返回4x.html页面，当出现5xx错误的时候，会返回5xx.html页面。

如果404.html和4xx.html同时存在。404.html的优先级会高于4xx.html。`精确匹配的优先级高于模糊匹配`

> 动态错误页面

静态的错误页面，不能够提供更好的错误提示。因此，SpringBoot还提供了一种动态的错误页面，自动收集了错误信息，可以展示在页面上。

动态错误页面存放在template下的error目录。其文件名的命名规则跟静态页面的规则相同，比如400.html,4xx.html等。

唯一不同的是，它可以渲染错误信息,这些错误信息是SpringBoot为我们收集的：

```html
出错的url：<h1 th:text="${path}"></h1>
出错的时间：<h1 th:text="${timestamp}"></h1>
状态码：<h1 th:text="${status}"></h1>
错误类型:<h1 th:text="${error}"></h1>
错误信息:<h1 th:text="${message}"></h1>
```

当静态页面和动态页面同时存在时，`动态页面的优先级高于静态页面`

在SpringBoot2.4.1的版本中，如果没有在error目录下创建静态页面或动态页面，在template目录下创建的error.html也会被解析成动态的错误页面。

**优先级小结：精确高于模糊，动态高于静态。**



#### 8.2 异常处理源码分析

ErrotMvcAutoConfiguration自动配置类中， 如果我们没有提供ErrorViewResolver接口的实现类，SpringBoot默认为我们提供一个DefaultErrorViewResolver

```java
@Bean
@ConditionalOnBean(DispatcherServlet.class)
@ConditionalOnMissingBean(ErrorViewResolver.class)
DefaultErrorViewResolver conventionErrorViewResolver() {
    return new DefaultErrorViewResolver(this.applicationContext, this.resourceProperties);
}
```

在BasicErrorController类中errorHtml方法中查找错误页面，如果没有找到错误页面，返回一个error.html的动态页面。也就是说在template目录下创建的error页面会被当成错误页面。

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
    HttpStatus status = getStatus(request);
    Map<String, Object> model = Collections
        .unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
    response.setStatus(status.value());
    // 错误页面处理
    ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
}
```

其中，resolveErrorView是处理错误页面的。

```java
protected ModelAndView resolveErrorView(HttpServletRequest request, HttpServletResponse response, HttpStatus status,Map<String, Object> model) {
    // 错误页面视图解析器，如果不能获取到对应的错误页面，返回null
    for (ErrorViewResolver resolver : this.errorViewResolvers) {
        // 调用下面的resolveErrorView方法
        ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
        if (modelAndView != null) {
            return modelAndView;
        }
    }
    return null;
}

public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
    // 获取状态码对应的错误页面
    ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
    // 获取不到状态码对应的页面，尝试模糊匹配错误页面，这也就体现了精确匹配的优先级高于模糊匹配
    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
    }
    return modelAndView;
}

```



#### 8.3 自定义异常数据

ErrotMvcAutoConfiguration自动配置类中,如果我们没有提供ErrorAttributes接口的实现类，SpringBoot默认为我们提供一个DefaultErrorAttributes。

```java
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes() {
    return new DefaultErrorAttributes(this.serverProperties.getError().isIncludeException());
}
```

我们可以自己提供一个ErrorAttributes的实现类，继承自DefaultErrorAttributes，自己提供异常数据：

```java
@Configuration
public class ErrorAttributeConfiguration extends DefaultErrorAttributes{
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> originalMap = super.getErrorAttributes(webRequest, includeStackTrace);
        originalMap.put("info","my error");
        return originalMap;
    }
}
```

![自定义异常数据](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210110135117273.png)



#### 8.4 自定义异常视图

我们可以自己提供一个ErrorViewResolver的实现类，继承自DefaultErrorViewResolver，自己提供异常视图：

需要注意的一点是：`model的数据是保存在unmodifiableMap的，如果想修改，可以先复制再修改，或覆盖它原本的值。`

```java
@Configuration
public class ErrorViewResolverConfiguration extends DefaultErrorViewResolver {

    public ErrorViewResolverConfiguration(ApplicationContext applicationContext, ResourceProperties resourceProperties) {
        super(applicationContext, resourceProperties);
    }

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        // 对应着template/error/myerror.html
        ModelAndView modelAndView = new ModelAndView("/error/myerror",model);
        // 也可以添加上自定义的异常数据
        modelAndView.addObject("info","myerror info");
        return modelAndView;
    }
}
```

![model数据](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210110142819449.png)



### 9.CORS跨域

#### 9.1 同源问题

同源指的是`协议、域名、端口`要相同。

同源策略的目的是为了保证用户信息的安全，防止恶意的网站窃取用户的数据。

目前所有的浏览器都支持同源策略。

设想这样一种情况：A网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取A网站的 Cookie，会发生什么？

目前，如果非同源，共有三种行为收到限制：

* Cookie、LocalStorage和IndexDB无法读取
* DOM无法获得
* AJAX请求不能发送

#### 9.2 解决AJAX同源问题

* JSONP(只支持GET请求)
* WebSocket
* CORS

#### 9.3 CORS

[阮一峰博客](http://www.ruanyifeng.com/blog/2016/04/cors.html)

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

#### 9.4 SpringBoot中使用CORS

案例：使用8081端口的应该发送AJAX请求8080端口的服务，

```javascript
$.ajax({
    url:"http://localhost:8080/hello",
    type:"get",
    success: function (result) {
        $("#msg").text(result);
    }
})
```

浏览器控制台报错：

```txt
Access to XMLHttpRequest at 'http://localhost:8080/hello' from origin 'http://localhost:8081' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.text
```

我们修改一下服务端，让它可以接收8081的请求,只需要加上@CorssOrigin注解，指明接收哪个域的请求：

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    @CrossOrigin(value = "http://localhost:8081")
    public String hello() {
        return "hello";
    }
}
```

8081再次向8080发起AJAX请求，请求成功。我们来看下请求头信息：

`Origin`用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果`Origin`指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含`Access-Control-Allow-Origin`字段（详见下文），就知道出错了，从而抛出一个错误，被`XMLHttpRequest`的`onerror`回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

```txt
GET /hello HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
sec-ch-ua: "Google Chrome";v="87", " Not;A Brand";v="99", "Chromium";v="87"
Accept: */*
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Origin: http://localhost:8081  # 
Sec-Fetch-Site: same-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost:8081/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
```

响应信息：

如果`Origin`指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

上面的头信息之中，有三个与CORS请求相关的字段，都以`Access-Control-`开头。

**（1）Access-Control-Allow-Origin**

该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求。

**（2）Access-Control-Allow-Credentials**

该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为`true`，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为`true`，如果服务器不要浏览器发送Cookie，删除该字段即可。

**（3）Access-Control-Expose-Headers**

该字段可选。CORS请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。上面的例子指定，`getResponseHeader('FooBar')`可以返回`FooBar`字段的值。

```txt
HTTP/1.1 200
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
Access-Control-Allow-Origin: http://localhost:8081
Content-Type: text/plain;charset=UTF-8
Content-Length: 5
Date: Sun, 10 Jan 2021 12:06:58 GMT
```

#### 9.5 全局配置CROS

@CrossOrigin可以加在方法上，也可以加在类上。但是如果有多个方法或多个类,还是要写多次@CrossOrigin。

可以在全局配置，只需要写一次。

```java
@Configuration
public class GlobalConfiguration implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 所有接口
            .allowedOrigins("http://localhost:8081") // 可接收来自http://localhost:8081的请求
            .allowedHeaders("*")  // 可接收所有请求
            .allowedMethods("*"); // 可接收所有的请求方法
    }

}
```

#### 9.6 探测请求

当AJAX发送PUT/DELETE请求，或Content-type为application/json的请求时，由于不知道服务端支不支持PUT方法，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错。

服务端接收到探测请求后，会返回一些信息。

请求信息：

预检"请求用的请求方法是`OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是`Origin`，表示请求来自哪个源。

除了`Origin`字段，"预检"请求的头信息包括两个特殊字段。

**（1）Access-Control-Request-Method**

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是`PUT`。

**（2）Access-Control-Request-Headers**

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

```txt
OPTIONS /hello HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Accept: */*
Access-Control-Request-Method: PUT
Origin: http://localhost:8081
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-site
Sec-Fetch-Dest: empty
Referer: http://localhost:8081/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
```

服务器收到"预检"请求以后，检查了`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后，确认允许跨源请求，就可以做出回应。

下面的HTTP响应中，关键的是`Access-Control-Allow-Origin`字段，表示`http://localhost:8081`可以请求数据。该字段也可以设为星号，表示同意任意跨源请求。

如果服务器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被`XMLHttpRequest`对象的`onerror`回调函数捕获。

一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个`Origin`头信息字段。服务器的回应，也都会有一个`Access-Control-Allow-Origin`头信息字段。

服务器回应的其他CORS相关字段如下：

**（1）Access-Control-Allow-Methods**

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

**（2）Access-Control-Allow-Headers**

如果浏览器请求包括`Access-Control-Request-Headers`字段，则`Access-Control-Allow-Headers`字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

**（3）Access-Control-Allow-Credentials**

该字段与简单请求时的含义相同。

**（4）Access-Control-Max-Age**

该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期1800秒，即允许缓存该条回应1800秒，在此期间，不用发出另一条预检请求。

```txt
HTTP/1.1 200
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
Access-Control-Allow-Origin: http://localhost:8081
Access-Control-Allow-Methods: PUT
Access-Control-Max-Age: 1800
Allow: GET, HEAD, POST, PUT, DELETE, OPTIONS, PATCH
Content-Length: 0
Date: Sun, 10 Jan 2021 12:42:33 GMT
```

### 10.注册拦截器

***

编写拦截器，需要实现HandlerInterceptor接口

```java
public class HelloInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");

    }
}

```

配置类中创建自己编写的拦截器:

```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(helloInterceptor());
    }

    @Bean
    public HandlerInterceptor helloInterceptor() {
        return new HelloInterceptor();
    }
}
```

如果在拦截器中添加@Component注解将拦截器加入到IOC容器当中：

```java
@Component
public class HelloInterceptor implements HandlerInterceptor {...}
```

在配置类中使用@Autowired注入拦截器：

```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Autowired
    private HelloInterceptor helloInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(helloInterceptor);
    }
}
```

这种注入的方法，如果不使用@ConfigurationProperties注入配置等功能，也是可以的。不过，推荐一起写在配置类中，方便管理。



### 11.系统启动任务

***

#### 11.1 CommandLineRunner

实现接口CommandLineRunner,并将其加入容器,CommandLineRunner就生效了。

如果有多个Runner,可以使用@Order指定优先级。@Order指定的数字越小，优先级越大。

@Order默认的优先级是Integet.MAX_VALUE

```java
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        // 这里接收的参数是命令行启动时传递的参数..
        System.out.println(Arrays.toString(args));
    }
}
```

配置启动时命令行传递参数：

![配置启动参数](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210110210942742.png)

#### 11.2 ApplicationRunner

实现接口ApplicationRunner，并将其加入容器,ApplicationRunner就生效了。

设定命令行传参为hello world --param=hahaha

```java
@Component
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // [hello, world, --param=hahaha]
        String[] sourceArgs = args.getSourceArgs();
        System.out.println(Arrays.toString(sourceArgs));

        // [hello, world]
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println(nonOptionArgs);

        // key-value
        Set<String> optionNames = args.getOptionNames();
        // [hahaha]
        for (String optionName : optionNames) {
            System.out.println(args.getOptionValues(optionName));
        }
    }
}
```

ApplicationRunner与CommandLineRunner可以同时存在，ApplicationRunner提供了更多的方式获取参数：

| 方法               | 描述                                                     |
| ------------------ | -------------------------------------------------------- |
| getSourceArgs()    | 获取启动的所有参数,与CommandLineRunner获取参数的形式相同 |
| getNonOptionArgs() | 获取没有key的参数，比如上面的hello world                 |
| getOptionNames()   | 获取参数的键值对,key为--指定的，比如上面的param=hahaha   |

### 12.整合Web基础组件

***

编写好基本的基础组件(Servlet,Listener,Filter)后,启动类添加@ServletComponentScan(basePackages = "xxxx")扫描基础组件所在的包，即可加入IOC容器。

下面以创建一个监听器为例子：

```java
@WebListener
public class MyListener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("requestDestroyed");
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("requestInitialized");

    }
}
```

### 13.路径映射

***

有时候，我们有一个动态页面，不需要接收数据，但是又不能直接访问，因为动态页面不在静态资源的目录下。

这时候，我们通常需要写一个controller来完成页面的跳转，这种做法很繁琐。

我们可以使用直接配置路径映射，而无需编写controller.

```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/test1")
                .setViewName("test1");
        registry.addViewController("/test2")
                .setViewName("test2");
    }
}
```

此时，在template目录下添加test1.html,test2.html，浏览器访问/test1，/test2可以访问到这两个动态页面。

### 14.参数类型转换

***

如果后端参数是一个Date类型，前端传递的是字符串：

```java
@RestController
public class ConverterController {

    @GetMapping("/date/{today}")
    public void date(@PathVariable Date today) {
        System.out.println(today);
    }
}
```

访问[http://localhost:8081/date/2021-01-01](),报错如下：

```txt
org.springframework.web.method.annotation.MethodArgumentTypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'java.util.Date'
```

后端并不能自动把String类型的参数转换成Date类型。

这就需要我们自己编写转换器了，编写一个类，实现Converter<源数据类型，目标类型>接口，并且加入到IOC容器：

Convert的第一个泛型参数是数据原来的数据类型，这里是String。

Convert的第二个泛型参数是数据的目标类型，这里是Date类型。

```java
@Component
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try {
            return simpleDateFormat.parse(source);
        } catch (ParseException e) {
            throw new RuntimeException("error");
        }
    }
}
```

### 15.自定义欢迎页，图标

***

`欢迎页index.html`可以是静态页面，也可以是动态页面，可以放在静态目录(static等静态资源访问的目录)下，也可以放在动态目录(template).

当静态页面和动态页面同时存在，优先访问静态页面。

`图标favicon.ico`放在static目录下。



### 16.去除自定义配置

***

去除自定义配置有2种方式：

* @SpringBootApplication(exclude="")
* 在配置文件中指定spring.autoconfigure.exclude,类型为List<Class>

```yml
spring:
  autoconfigure:
    exclude: 
      - org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
      - org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration
```





