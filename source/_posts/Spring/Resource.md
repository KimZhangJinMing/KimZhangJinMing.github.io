---
title: Resource接口.md
date: 2020-09-15 22:19:35
categories: Spring
tags: Spring
---

### JDK中加载资源的URL有什么缺点？ 

JDK中的资源是通过java.net.URL类来加载的，但是它不够强大。但是它有缺点：

* DK中没有提供标准的URL实现类去加载classpath路径下的资源或者是相对于ServletContext的资源。虽然它能够注册新的处理器去处理特殊的前缀(比如http:// ,file//)，但一般说比较复杂。
* URL接口提供的功能不够全面，缺乏很多基本的功能。比如检查所指向的资源是否存在等。

### Spring中对资源的抽象

Spring 改进了 Java 资源访问的策略。Spring 为资源访问提供了一个 Resource 接口，该接口提供了更强的资源访问能力，Spring 框架本身大量使用了 Resource 接口来访问底层资源。

![Resource接口](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/1645290-20190908175249585-769163389.png)

getFile()和getURL()方法通常无须使用，仅在通过简单方式访问无法实现时，Resource 提供传统的资源访问的功能。

### 策略模式

策略模式用于封装系列的算法，这些算法通常被封装在一个被称为 Context 类中，客户端程序可以自由选择其中一种算法，或让 Context 为客户端选择一个最佳的算法——使用策略模式的优势是为了支持算法的自由切换。

Spring 提供两个标志性接口：

- ResourceLoader：该接口实现类的实例可以获得一个 Resource 实例。
- ResourceLoaderAware：该接口实现类的实例将获得一个 ResourceLoader 的引用。

在 ResourceLoader 接口里有如下方法：

Resource getResource(String location)：该接口仅包含这个方法，该方法用于返回一个 Resource 实例。ApplicationContext 的实现类都实现 ResourceLoader 接口，因此 ApplicationContext 可用于直接获取 Resource 实例。

此处 Spring 框架的 ApplicationContext 不仅是 Spring 容器，而且它还是资源访问策略的”决策者”，也就是策略模式中 Context 对象，它将为客户端代码”智能”地选择策略实现。

当 ApplicationContext 实例获取 Resource 实例时，系统将默认采用与 ApplicationContext 相同的资源访问策略。对于如下代码：

```java
Resource res = ctx.getResource("some/resource/path/myTemplate.txt);
```

从上面代码中无法确定 Spring 将哪个实现类来访问指定资源，Spring 将采用和 ApplicationContext 相同的策略来访问资源。也就是说：如果 ApplicationContext 是 FileSystemXmlApplicationContext，res 就是 FileSystemResource 实例；如果 ApplicationContext 是 ClassPathXmlApplicationContext，res 就是 ClassPathResource 实例；如果 ApplicationContext 是 XmlWebApplicationContext，res 是 ServletContextResource 实例。

```java
public static void main(String[] args) throws IOException {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/spring-config.xml");
		Resource resource = applicationContext.getResource("text.txt");
    	// class org.springframework.core.io.DefaultResourceLoader$ClassPathContextResource
		System.out.println(resource.getClass());
    	// filename:text.txt
		System.out.println("filename:"+resource.getFilename());
    	// description:class path resource [text.txt]
		System.out.println("description:"+resource.getDescription());
}
```

由于程序中使用了 ClassPathApplicationContext 来获取资源，所以 Spring 将会从类加载路径下来访问资源，也就是使用 ClassPathResource 实现类.

```java
public static void main(String[] args) throws IOException {
    ApplicationContext applicationContext = new FileSystemXmlApplicationContext("D:\\IDEA Pro\\spring-framework-5.2.0.RELEASE\\spring-framework-5.2.0.RELEASE\\spring-demo\\src\\main\\resources\\spring\\spring-config.xml");
		Resource resource = applicationContext.getResource("text.txt");
    	// class org.springframework.core.io.FileSystemResource
		System.out.println(resource.getClass());
    	// filename:text.txt
		System.out.println("filename:"+resource.getFilename());
    	// description:file [D:\IDEA Pro\spring-framework-5.2.0.RELEASE\spring-framework-5.2.0.RELEASE\text.txt]
		System.out.println("description:"+resource.getDescription());
}
```

从上面的执行结果可以看出，程序的 Resource 实现类发了改变，变为 FileSystemResource 实现类。

另一方面使用 ApplicationContext 来访问资源时，也可不理会 ApplicationContext 的实现类，强制使用指定的 ClassPathResource、FileSystemResource 等实现类，这可通过不同前缀来指定，如下代码所示：

```java
public static void main(String[] args) throws IOException {
    ApplicationContext applicationContext = new FileSystemXmlApplicationContext("D:\\IDEA Pro\\spring-framework-5.2.0.RELEASE\\spring-framework-5.2.0.RELEASE\\spring-demo\\src\\main\\resources\\spring\\spring-config.xml");
    // class org.springframework.core.io.ClassPathResource
    Resource resource = applicationContext.getResource("classpath:text.txt");
}
```

类似地，还可以使用标准的 `java.net.URL` 前缀来强制使用 `UrlResource` ，如下所示：

```java
// class org.springframework.core.io.FileUrlResource
Resource resource = applicationContext.getResource("file:text.txt");
// class org.springframework.core.io.UrlResource
```

以下是常见前缀及对应的访问策略：

- classpath:以 ClassPathResource 实例来访问类路径里的资源。
- file:以 UrlResource 实例访问本地文件系统的资源。
- http:以 UrlResource 实例访问基于 HTTP 协议的网络资源。
- 无前缀:由于 ApplicationContext 的实现类来决定访问策略。

```java
public static void main(String[] args) throws IOException {
    // 使用指定前缀来加载资源，只对当次有效
    ApplicationContext applicationContext = new FileSystemXmlApplicationContext("classpath:spring/spring-config.xml");
    Resource resource = applicationContext.getResource("test.txt");
    // 	class org.springframework.core.io.FileSystemResource
    System.out.println(resource.getClass());
}
```

创建 Spring 容器时，系统将从类加载路径来搜索 spring-config.xml；但使用 ApplicationContext 来访问资源时，依然采用的是 FileSystemResource 实现类，这与 FileSystemXmlApplicationContext 的访问策略是一致的。这表明：通过 classpath: 前缀指定资源访问策略仅仅对当次访问有效，程序后面进行资源访问时，还是会根据 AppliactionContext 的实现类来选择对应的资源访问策略。

因此如果程序需要使用 ApplicationContext 访问资源，建议显式采用对应的实现类来加载配置文件，而不是通过前缀来指定资源访问策略。当然，我们也可在每次进行资源访问时都指定前缀，让程序根据前缀来选择资源访问策略。

```java
public static void main(String[] args) throws IOException {
    // 使用指定前缀来加载资源，只对当次有效
    ApplicationContext applicationContext = new FileSystemXmlApplicationContext("classpath:spring/spring-config.xml");
    Resource resource = applicationContext.getResource("classpath:test.txt");
    // 	class org.springframework.core.io.ClassPathResource
    System.out.println(resource.getClass());
}
```

由此可见，如果每次进行资源访问时指定了前缀，则系统会采用前缀相应的资源访问策略。



### 策略模式的应用

Resource 接口本身没有提供访问任何底层资源的实现逻辑，针对不同的底层资源，Spring 将会提供不同的 Resource 实现类，不同的实现类负责不同的资源访问逻辑。

Resource 接口就是策略模式的典型应用，Resource 接口就代表资源访问策略，但具体采用哪种策略实现，Resource 接口并不理会。**客户端程序只和 Resource 接口耦合**，并不知道底层采用何种资源访问策略，这样应用可以在不同的资源访问策略之间自由切换。比如，使用SystemFileResource替换ClassPathResource，只需要切换Resource接口的实现类即可。

```java
Resource resource = new ClassPathResource("classpath:db.properties");
// 相关的resource操作方法....
```

```java
Resource resource = new SystemFileResource("D://db.properties");
// 相关的resource操作方法....
```

Resource 不仅可在 Spring 的项目中使用，也可直接作为资源访问的工具类使用。意思是说：即使不使用 Spring 框架，也可以使用 Resource 作为工具类，用来代替 URL。当然，使用 Resource 接口会让代码与 Spring 的接口耦合在一起，但这种耦合只是部分工具集的耦合，不会造成太大的代码污染。

### Resource的实现类

Spring将常见的资源按照资源类型和路径分为了7大组，全部都实现了AbstractResource抽象类，分别如下：

* FileSystemResource     代表文件系统资源，以操作系统文件路径的方式访问

* PathResource    代表文件系统资源，以Path对象访问

* AbstractFileResolvingResource  代表需要解析的路径资源，如类资源Class  、URL资源等等  是个抽象类，有三个具体的实现

* ByteArrayResource  代表字节数组资源

* VfsResource  代表JBoss的虚拟文件系统VFS

* InputStreamResource  代表输入二进制流的资源

* DescriptiveResource 代表资源描述的资源，可以理解为资源的元数据(元资源) 不指向任何的实际资源对象

#### UrlResource的使用

虽然 UrlResource 是为访问网络资源而设计的，但通过使用 file 前缀也可访问本地磁盘资源。如果需要访问网络资源，可以使用如下两个常用前缀：

- http:－该前缀用于访问基于 HTTP 协议的网络资源。
- ftp:－该前缀用于访问基于 FTP 协议的网络资源。

由于 UrlResource 是对 java.net.URL 的封装，所以 UrlResource 支持的前缀与 URL 类所支持的前缀完全相同。

```java
public static void main(String[] args) throws IOException {
		// test.txt放在当前目录下
		Resource resource = new UrlResource("file:test.txt");
		// class org.springframework.core.io.UrlResource
		System.out.println(resource.getClass());
    	// filename:test.txt
		System.out.println("filename:"+resource.getFilename());
    	// description:URL [file:test.txt]
		System.out.println("description:"+resource.getDescription());
}
```

#### ClassPathResource的使用

ClassPathResource 用来访问类加载路径下的资源，相对于其他的 Resource 实现类，其主要优势是方便访问类加载路径里的资源，尤其对于 Web 应用，ClassPathResource 可自动搜索位于 WEB-INF/classes 下的资源文件，无须使用绝对路径访问。

```java
public static void main(String[] args) throws IOException {
		// test.txt放在resources文件夹下
		Resource resource = new ClassPathResource("classpath:test.txt");
		// class org.springframework.core.io.ClassPathResource
		System.out.println(resource.getClass());
    	// filename:classpath:test.txt
		System.out.println("filename:"+resource.getFilename());
    	// description:class path resource [classpath:test.txt]
		System.out.println("description:"+resource.getDescription());
}
```

ClassPathResource 实例可使用 ClassPathResource 构造器显式地创建，但更多的时候它都是隐式创建的，当执行 Spring 的某个方法时，该方法接受一个代表资源路径的字符串参数，当 Spring 识别该字符串参数中包含 classpath: 前缀后，系统将会自动创建 ClassPathResource 对象。

#### FileSystemResource的使用

与前两种 Resource 作资源访问的区别在于：资源字符串确定的资源，位于本地文件系统内 ，而且无须使用任何前缀。

```java
public static void main(String[] args) throws IOException {
		Resource resource = new FileSystemResource("D:\\IDEA Pro\\spring-framework-5.2.0.RELEASE\\spring-framework-5.2.0.RELEASE\\spring-demo\\src\\main\\resources\\test.txt");
		// class org.springframework.core.io.FileSystemResource	
		System.out.println(resource.getClass());
    	// filename:test.txt
		System.out.println("filename:"+resource.getFilename());
    	// description:file [D:\IDEA Pro\spring-framework-5.2.0.RELEASE\spring-framework-5.2.0.RELEASE\spring-demo\src\main\resources\test.txt]
		System.out.println("description:"+resource.getDescription());
}
```

通过上面代码不难发现，程序使用 UrlResource、FileSystemResource、ClassPathResource 三个实现类来访问资源的代码差异并不大，**唯一的缺点在于客户端代码需要与 Resource 接口的实现类耦合，这依然无法实现高层次解耦。**

这对于策略模式来说将没有任何问题，策略模式要解决的就是这个问题，策略模式将会提供一个 Context 类来为客户端代码”智能”地选择策略实现类。



### Spring资源访问相关类

所有资源高度抽象为二进制流，也就是不管你资源文件是什么格式，也不管你资源在哪里，Spring底层访问的都是文件的二进制流，这样就可以统一访问了。

![资源访问相关接口](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/687300-20190301175101710-1284958427.png)

* EncodedResource类：

  EncodedResource类是辅助类，从名字上可以看出，它是一个编码类。资源加载的时候，是采用操作系统默认的编码方式，为解决编码不统一的问题，Spring的IOC获取资源后，需要把资源重新编码一下。例如，在Spring应用程序上下文的 XmlBeanDefinitionReader 类中，获取了 资源后，需要对资源进一步解析，在解析之前，调用 new EncodedResource();

  

* WritableResource接口：

  Resource接口定义了资源的可访问等一系列的操作，但有些资源需要可写入的，如文件对象，因此还定义了WritableResource接口，继承了Resource接口，表示资源具有可写入的能力。

  FileSystemResource既实现了Resource接口，又实现了Writable接口，实现了可写可读的操作。

  ```java
  public interface WritableResource extends Resource {
      //  判断资源是否可写入
      default boolean isWritable() {
  		return true;
  	}
      // 获取资源写入的二进制流
      OutputStream getOutputStream() throws IOException;
      // 获取资源可写入的字节管道对象
      default WritableByteChannel writableChannel() throws IOException {
  		return Channels.newChannel(getOutputStream());
  	}
  }
  ```

  

* ContextResource接口：

  ContextResource 接口也继承了Resource接口，表示可以从关闭的上下文Context中获取资源的路径，这样应用程序上下文也就有了返回上下文路径的能力。

  ```java
  public interface ContextResource extends Resource {
      // 从关闭的上下文Context中获取资源的路径
      String getPathWithinContext();
  }
  ```

* AbstractResource类：

  AbstractResource 是个抽象类，Resource接口的大部分方法的默认实现。

  ```java
  public abstract class AbstractResource implements Resource {
      @Override
  	public boolean exists() {
  		// 先尝试判断文件是否存在，如果资源是文件形式，判断文件是否存在
  		if (isFile()) {
  			try {
  				return getFile().exists();
  			}
  			catch (IOException ex) {
                  return false;
              }
  		}
  		// 如果资源不是文件，回溯到二进制流，看二进制流是否能打开，如果可以，则就存在了
  		try {
  			getInputStream().close();
  			return true;
  		}
  		catch (Throwable ex) {
  			return false;
  		}
  	}
  
      /**
      * 判断资源是否可读，如果资源存在，抽象类认为资源总是可读的
      */
      @Override
  	public boolean isReadable() {
  		return exists();
  	}
  	
       /**
      * 判断资源是否打开，抽象类认为资源总是关闭的
      */
      @Override
  	public boolean isOpen() {
  		return false;
  	}
      
      /**
      * 判断是否是文件，抽象类认为资源不是文件
      */
      @Override
  	public boolean isFile() {
  		return false;
  	}
      
     /**
      * 该资源解析为URL，需要具体的子类去实现，这里抽象类假设都不能解析URL
      */
      @Override
  	public URL getURL() throws IOException {
  		throw new FileNotFoundException(getDescription() + " cannot be resolved to URL");
  	}
      
      /**
      * 该资源解析为URI
      */
      @Override
  	public URI getURI() throws IOException {
          // 获取子类实现的URL,通过URL解析URI
  		URL url = getURL();
  		try {
  			return ResourceUtils.toURI(url);
  		}
  		catch (URISyntaxException ex) {
  			throw new NestedIOException("Invalid URI [" + url + "]", ex);
  		}
  	}
      
     /**
      * 该资源解析为File,抽象类认为都不解析为File
      */
      @Override
  	public File getFile() throws IOException {
  		throw new FileNotFoundException(getDescription() + " cannot be resolved to absolute file path");
  	}
      
      /**
      * 获取资源的字节管道
      */
      @Override
  	public ReadableByteChannel readableChannel() throws IOException {
  		return Channels.newChannel(getInputStream());
  	}
      
      /**
      * 获取资源的长度，通过获取资源二进制流，遍历计算资源的长度  字节为单位
      */
      @Override
  	public long contentLength() throws IOException {
  		InputStream is = getInputStream();
  		try {
  			long size = 0;
  			byte[] buf = new byte[256];
  			int read;
  			while ((read = is.read(buf)) != -1) {
  				size += read;
  			}
  			return size;
  		}
  		finally {
  			try {
  				is.close();
  			}
  			catch (IOException ex) {
  			}
  		}
  	}
  	// 更多方法查看源码.....
  }
  ```

  小结：

  1. 没有实现资源的根接口 InputStreamSource ，方法getInputStream() 留给具体的子类去实现
2. 没有实现Resource接口的 getDescription() 方法，留给子类去实现，资源文件默认的equals()、hashCode() 都通过这个来判断



### ResourceLoader接口

ResourceLoader接口使用策略模式来提供Resource：

![ResourceLoader](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20200915225242681.png)

DefaultResourceLoader是ResourceLoader的实现类，实现了获取一个resource的方法。

ResourcePatternResolver实现了获取多个resource的方法。ApplicationContext接口是ResourcePatternResolver接口的子接口，也就说明ApplicationContext能获取多个资源。

AbstractApplicationContext既实现了ResourcePatternResolver接口，又继承了DefaultResourceLoader类，所以容器具有获取一个resource和获取多个resource的能力。

我们来看一下DefaultResourceLoader的getResource方法是怎么通过策略模式返回不同的Resource实例的：

```java
public Resource getResource(String location) {
    Assert.notNull(location, "Location must not be null");
    /**
	* 协议解析器解析，若指定了前缀file:或者http:等协议前缀，直接返回协议对应的Resource
	* 这也就是为什么可以通过指定前缀获取当次资源的Resource
	*/
    for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
        Resource resource = protocolResolver.resolve(location, this);
        if (resource != null) {
            return resource;
        }
    }

    /**
	* 如果是/开头，返回ClassPathResource
	*/
    if (location.startsWith("/")) {
        return getResourceByPath(location);
    }
    /**
	* 如果是classpath:开头，返回ClassPathResource
	*/
    else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
        return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
    }
    else {
        try {
            /**
			* 尝试解析成UrlResource,如果是文件，返回FileUrlResource,如果不是返回UrlResource
 			*/
            URL url = new URL(location);
            return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
        }
        catch (MalformedURLException ex) {
            // 如果不能解析成UrlResource,使用ClassPathResource
            return getResourceByPath(location);
        }
    }
}
```







### ResourceLoaderAware接口

ResourceLoaderAware 接口则用于指定该接口的实现类必须持有一个 ResourceLoader 实例。

类似于 Spring 提供的 BeanFactoryAware、BeanNameAware 接口，ResourceLoaderAware 接口也提供了一个 setResourceLoader() 方法，该方法将由 Spring 容器负责调用，Spring 容器会将一个 ResourceLoader 对象作为该方法的参数传入。

当我们把将 ResourceLoaderAware 实例部署在 Spring 容器中后，Spring 容器会将自身当成 ResourceLoader 作为 setResourceLoader() 方法的参数传入，由于 ApplicationContext 的实现类都实现了 ResourceLoader 接口，Spring 容器自身完全可作为 ResourceLoader 使用。



### 使用Resource作为属性

前面介绍了 Spring 提供的资源访问策略，但这些依赖访问策略要么需要使用 Resource 实现类，要么需要使用 ApplicationContext 来获取资源。实际上，当应用程序中的 Bean 实例需要访问资源时，Spring 有更好的解决方法：直接利用依赖注入。

从这个意义上来看，Spring 框架不仅充分利用了策略模式来简化资源访问，而且还将策略模式和 IoC 进行充分地结合，最大程度地简化了 Spring 资源访问。

归纳起来，如果 Bean 实例需要访问资源，有如下两种解决方案：

- 代码中获取 Resource 实例。
- 使用依赖注入。

对于第一种方式的资源访问，当程序获取 Resource 实例时，总需要提供 Resource 所在的位置，不管通过 FileSystemResource 创建实例，还是通过 ClassPathResource 创建实例，或者通过 ApplicationContext 的 getResource() 方法获取实例，都需要提供资源位置。这意味着：资源所在的物理位置将被耦合到代码中，如果资源位置发生改变，则必须改写程序。因此，通常建议采用第二种方法，让 Spring 为 Bean 实例依赖注入资源。



### 加载多份配置文件

classpath* : 前缀提供了装载多个 XML 配置文件的能力，当使用 classpath*: 前缀来指定 XML 配置文件时，系统将搜索类加载路径，找出所有与文件名的文件，分别装载文件中的配置定义，最后合并成一个 ApplicationContext。看如下代码：

```java
ApplicationContext applicationContext = new FileSystemXmlApplicationContext("classpath*:spring/spring-config.xml");
```

在类路径下放置两份相同文件名的配置文件：

![两份文件名相同的配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20200915214633261.png)

```java
Loading XML bean definitions from URL [file:/G:/publish/codes/
 08/8.3/ApplicationContext/classes/spring-config.xml]
 Loading XML bean definitions from URL [file:/G:/publish/codes/
 08/8.3/ApplicationContext/classes/spring/spring-config.xml]
```

当使用 classpath*: 前缀时，Spring 将会搜索类加载路径下所有满足该规则的配置文件。

如果不是采用 classpath*: 前缀，而是改为使用 classpath: 前缀，Spring 只加载第一份符合条件的 XML 文件，例如如下代码：

```java
ApplicationContext applicationContext = new FileSystemXmlApplicationContext("classpath:spring/spring-config.xml");
```

执行上面代码将只看到如下输出：

```java
Loading XML bean definitions from class path resource [spring-config.xml]
```

当使用 classpath: 前缀时，系统通过类加载路径搜索 spring-config.xml文件，如果找到文件名匹配的文件，系统立即停止搜索，装载该文件，即使有多份文件名匹配的文件，系统只装载第一份文件。资源文件的搜索顺序则取决于类加载路径的顺序，排在前面的配置文件将优先被加载。

另外，还有一种可以一次性装载多份配置文件的方式：指定配置文件时指定使用通配符，例如如下代码：

```java
ApplicationContext applicationContext = new FileSystemXmlApplicationContext("classpath:bean*.xml");
```

位于类加载路径下所有以 bean 开头的 XML 配置文件都将被加载。

除此之外，Spring 甚至允许将 classpath*: 前缀和通配符结合使用，如下语句也是合法的：

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath*:bean*.xml");
```

上面语句创建 ApplicationContext 实例时，系统将搜索所有的类加载路径下，所有以 bean.xml 开头的 XML 配置文件

###  file:前缀的用法

```java
public class SpringTest {
    public static void main(String[] args) throws Exception {
        // 通过文件通配符来一次性装载多份配置文件
        //ApplicationContext ctx = newFileSystemXmlApplicationContext("bean.xml");
        ApplicationContext ctx = newFileSystemXmlApplicationContext("/bean.xml");
        System.out.println(ctx);
        // 使用 ApplicationContext 的默认策略加载资源，没有指定前缀
        Resource r = ctx.getResource("book.xml");
        // 输出 Resource 描述
        System.out.println(r.getDescription());
    }
}
```

程序有两行粗体字代码用于创建 ApplicationContext，第一行粗体字代码指定资源文件时采用了相对路径的写法,第二行粗体字代码指定资源文件时采用了绝对路径的写法。

任意注释两条语句的其中之一，程序正常执行，没有任何区别，两句代码读取了相同的配置资源文件。问题是：如果程序中明明采用的一个是绝对路径、一个相对路径，为什么执行效果没有任何区别？

产生问题的原因：当 FileSystemXmlApplicationContext 作为 ResourceLoader 使用时，它会发生变化，FileSystemApplicationContext 会简单地让所有绑定的 FileSystemResource 实例把绝对路径都当成相对路径处理，而不管是否以斜杠开头，所以上面两行代码的效果是完全一样的。

如果程序中需要访问绝对路径，则不要直接使用 FileSystemResource 或 FileSystemXmlApplicationContext 来指定绝对路径。建议强制使用 file: 前缀来区分相对路径和绝对路径，例如如下两行代码：

```java
// 访问相对路径下的 bean.xml
ApplicationContext ctx = new FileSystemXmlApplicationContext("file:bean.xml");
// 访问绝对路径下 bean.xml
ApplicationContext ctx = new FileSystemXmlApplicationContext("file:/bean.xml");
```

相对路径以当前工作路径为路径起点，而绝对路径以文件系统根路径为路径起点。