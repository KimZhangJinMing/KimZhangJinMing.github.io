---
title: Spring容器.md
date: 2020-09-15 22:19:35
categories: Spring
tags: IOC
---
### 简单容器BeanFactory

BeanFactory接口是Spring中的简单容器。它定义了一个常量和多个bean操作相关的方法。

<img src="https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20200920122506382.png" alt="BeanFactory简单容器" style="zoom: 67%;" />

其中，FACTORY_BEAN_PREFIX定义了访问FactoryBean的前缀。由于通过FactoryBean的beanName获取的对象是FactoryBean生产的对象，而通过&beanName能获取到FactoryBean本身。

![BeanFactory继承关系](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20200920131516053.png)

* AutowireCapableBeanFactory：可以用来加载第三方的bean，由于无法在第三方提供的bean源代码中将其加入到Spring容器，所以需要这个接口。其中定义了装配bean的策略，比如byName、byType。
* HierarchicalBeanFactory：定义了层级关系， 接口中getParentBeanFactory方法能获取父级容器BeanFactory.
* ListableBeanFactory：能以列表的形式返回bean的信息。该接口的方法多数返回值为数组类型
* ConfigurableListableBeanFactory：定义了能配置容器的方法，比如ignoreDependencyType能忽略某些依赖的注入
* DefaultListableBeanFactory：抽象类，主要实现了容器中一些通用的方法。此外，它还实现了BeanDefinitionRegistry接口，是一个可独立运行的容器。其中定义了一个ConcurrentHashMap的成员变量beanDefinitionMap，这个map就是存放bean的容器。

> 在整个Spring框架中，只有2个地方定义了beanDefinitionMap，一个是DefaultListableBeanFactory类，一个是SimpleBeanDefinitionRegistry类，Spring默认是使用DefaultListableBeanFactory来存放bean



### 高级容器ApplicationContext

高级容器 ApplicationContext在BeanFactory的基础上实现了更多的接口，提供了更多的功能。相比于BeanFactory，ApplicationContext是面向Spring框架用户使用的，而BeanFactory更多的是Spring内部使用的容器。

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
```

* EnvironmentCapable：该接口中的getEnvironment能获取web.xml文件中的信息，比如在使用Spring MVC时，需要获取配置文件存放路径
* ResourcePatternResolver：该接口能解析多个Resource资源文件，支持classpath*的前缀

ApplictionContext接口提供的方法都是"可读"的方法，而ConfigurableApplicationContext接口提供了可以配置ApplicationContext的方法，比如refresh、close等方法。

![ConfigurableApplicationContext与ApplicationContext的关系](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20200920133717208.png)

AbstractApplicationContext类实现了ConfigurableApplicationContext中的通用方法，包括refresh方法：

refresh方法使用了模板方法的设计模式，模板方法设计模式主要包含以下四种方法：

* 模板方法：组合一些方法的执行流程
* 具体方法：具体实现的一些通用方法
* 钩子方法：视情况决定是否执行的方法，子类可以实现，也可以不实现
* 抽象方法：带有abstract关键字，子类必须实现的方法

refresh方法就是模板方法，定义了一系列容器初始化，配置解析的方法。

prepareRefresh方法是具体方法，AbstractApplicationContext提供了该方法的实现。

obtainFreshBeanFactory是抽象方法，AbstractApplicationContext并不做具体实现，该方法由子类实现。

postProcessBeanFactory是钩子方法，子类根据情况，决定实现与否。

```java
public void refresh() throws BeansException, IllegalStateException {
    // 加锁保证线程安全，避免在refresh的时候，容器再次被初始化或者destroy
    synchronized (this.startupShutdownMonitor) {
        // 容器初始化准备工作，主要是设置容器启动时间，检验必须的属性等
        prepareRefresh();

        // 告诉子类刷新内部容器
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 初始化容器配置，比如设置resourceLoader等
        prepareBeanFactory(beanFactory);

        try {
            // 钩子方法，默认无实现，将由子类视情况实现.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

### refresh方法详解

prepareRefresh方法，容器刷新前的准备工作：

主要是记录容器启动的时间，设置容器的状态，存储事件监听器，创建事件集合等。

this指的是ApplicationContext

```java
rotected void prepareRefresh() {
    // 记录容器启动时间
    this.startupDate = System.currentTimeMillis();
    // 设置容器状态，closed、active是AtomicBoolean类型的
    this.closed.set(false);
    this.active.set(true);

    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // 初始化propertySources属性，<context:property-placeholder location=""/>
    // 该方法由子类实现
    initPropertySources();

    // 验证所有标记为必需的属性都是可解析的，检查是否有必需的属性还没加载进来
    getEnvironment().validateRequiredProperties();

    // 存储刷新前注册的监听器
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // 创建事件集合
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

obtainFreshBeanFactory：

对于xml的方式，在此方法中创建DefaultListableBeanFactory工厂，将BeanDefinition注册到容器中，返回DefaultListableBeanFactory容器。

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   refreshBeanFactory();
   return getBeanFactory();
}
```

对于注解的方式，真正调用的是GenericApplicationContext方法。

注解方式在创建容器的构造方法中，就已经创建了DefaultListableBeanFactory容器，这里置需要将refreshed标志从false改为true(refreshed是AtomicBoolean类型)，设置SerializationId后返回DefaultListableBeanFactory容器即可。

```java
protected final void refreshBeanFactory() throws IllegalStateException {
		if (!this.refreshed.compareAndSet(false, true)) {
			throw new IllegalStateException(
					"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
		}
		this.beanFactory.setSerializationId(getId());
	}
```

prepareBeanFactory(beanFactory);

为DefaultListableBeanFactory设置一些属性，注册一些默认的Bean

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// 为DefaultListableBeanFactory工厂设置容器的类加载器，这样就可以去加载外部容器的资源了
		beanFactory.setBeanClassLoader(getClassLoader());
		// 设置表达式解析器，比如spel表达式解析器
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		// 设置一个默认的属性编辑器
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		/*
		 设置后置处理器，该处理器的作用是当应用程序定义的bean实现ApplicationContextAware接口时，
		 注入ApplicationContext对象
		  */
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		/*
		 如果某个bean依赖于以下接口的实现类，在自动装配的时候忽略它们
		 Spring会通过其他方式来处理这些依赖
		  */
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		/**
		 * 设置自动装配的特殊规则，比如是BeanFactory接口的实现类，在运行时修订为当前的BeanFactory
		 *  this指的是AbstractApplicationContext,因为AbstractApplicationContext实现了BeanFactory,ResourceLoader,ApplicationEventPublisher,ApplicationContext接口
		 */
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// 注册一个后置处理器，用于检查内部的bean是不是ApplicationListener,如果是则加入到事件监听队列
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```

postProcessBeanFactory(beanFactory);

这是一个钩子方法，允许容器的子类去注册后置处理器。

invokeBeanFactoryPostProcessors(beanFactory);

调用容器注册的容器级别的后置处理器。

registerBeanPostProcessors(beanFactory);

处理完BeanFactoryPostProcessors,接着注册BeanPostProcessors

initMessageSource();

国际化消息处理。

initApplicationEventMulticaster()

初始化ApplicationMulitCaster做为事件的发布者，可以存储所有事件监听者的信息，并根据不同的事件，通知不同的事件监听者

```java
protected void initApplicationEventMulticaster() {
    	// 获取DefaultListableBeanFactory
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    	// 如果容器中包含了ApplicationMulitCaster，取出赋值
		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
			this.applicationEventMulticaster =
					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
			if (logger.isTraceEnabled()) {
				logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
			}
		}
		else {
			// 容器中没有，使用默认的SimpleApplicationEventMulticaster
			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
			if (logger.isTraceEnabled()) {
				logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
						"[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
			}
		}
	}
```

onRefresh();

预留给ApplicationContext子类用于初始化其他特殊的bean，该方法需要在所有单例bean初始化之前调用.

finishBeanFactoryInitialization()

初始化ConvertService，实例化bean

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// 初始化ConvertService类型转换器，转换器的作用是为bean属性赋值的时候进行类型转换工作
		// 只有Spring容器本身不支持的类型，才需要类型转换器
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// 如果没有注册过bean的后置处理器，则注册默认的解析器
		// 解析器主要用于解析properties的PropertiesPlaceholderConfigurer
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// 停止使用临时的类加载器
		beanFactory.setTempClassLoader(null);

		// 允许缓存所有bean定义的元数据，不希望有进一步的修改
		beanFactory.freezeConfiguration();

		// 实例化所有非延时加载的单例
		beanFactory.preInstantiateSingletons();
	}
```

preInstantiateSingletons()

```java
public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		for (String beanName : beanNames) {
			// 兼容各种BeanDefinition，GenericBeanDefinition可以转换成RootBeanDefinition
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			// 非抽象类 ，单例且不是延迟加载的
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					// 处理FactoryBean的实例
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						final FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
											((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					// 处理普通bean的实例
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
	}

```

finishRefresh

清除缓存，发布容器刷新完成的事件等

```java
protected void finishRefresh() {
		// 清除一些上下文级别的资源缓存
		clearResourceCaches();

		// 初始化生命周期处理器
		initLifecycleProcessor();

		// 首先将刷新传播到生命周期处理器
		getLifecycleProcessor().onRefresh();

		// 发布容器刷新完成的事件给事件监听者
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		LiveBeansView.registerApplicationContext(this);
	}

```



### xml配置文件注册BeanDefinition流程

configLocation是配置文件的路径。

```java
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
    this(new String[] {configLocation}, true, null);
}
```

this(new String[] {configLocation}, true, null);

refresh默认值是true，也就是容器启动的时候，会调用refresh方法。refresh方法在AbstractApplicationContext类中实现。

```java
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}
```

```JAVA
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 容器初始化准备工作，主要是设置容器启动时间，检验必须的属性等
        prepareRefresh();

        // 告诉子类刷新内部容器
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 有关postProcessBeanFactory的相关操作...
 }
```

obtainFreshBeanFactory():

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    return getBeanFactory();
}
```

refreshBeanFactory():

```java
protected final void refreshBeanFactory() throws BeansException {
    // 关闭之前的beanFactory
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        // 创建新的beanFactory，使用DefaultListableBeanFactory来加载beanDefinition
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);
        // 加载bean定义信息
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}

```

AbstractXmlApplicationContext : loadBeanDefinitions(beanFactory);

创建XmlBeanDefinitionReader来解析xml配置文件。

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's
    // resource loading environment.
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    // this -> ClassPathApplicationContext,所有的applicationContext都实现了ClassLoader接口
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    // 调用重载的方法加载beanDefinition
    loadBeanDefinitions(beanDefinitionReader);
}
```

AbstractXmlApplicationContext : loadBeanDefinitions(XmlBeanDefinitionReader reader);

根据reader读取的配置文件，去加载beanDefinition。

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    Resource[] configResources = getConfigResources();
    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        reader.loadBeanDefinitions(configLocations);
    }
}
```

AbstractBeanDefinitionReader:loadBeanDefinitions(String... locations);

该方法可以加载多个配置文件，实际上就通过for循环调用加载一个配置文件的方法。

```java
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
    Assert.notNull(locations, "Location array must not be null");
    int count = 0;
    for (String location : locations) {
        count += loadBeanDefinitions(location);
    }
    return count;
}
```

AbstractBeanDefinitionReader:loadBeanDefinitions(String location);

加载一个配置文件的方法

```java
public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(location, null);
}
```

AbstractBeanDefinitionReader:loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources)

调用本类的其他重载方法。在这个方法中，获取了resourceLoader。通过resourceLoader的实例类型，判断是加载单个资源还是多个资源，最终调用子类的loadBeanDefinitions(Resource... resources)或者loadBeanDefinitions(Resource resources)方法来加载一个或多个资源。此时已经将配置文件的location转换成了Spring抽象的Resource接口。

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
    ResourceLoader resourceLoader = getResourceLoader();
    if (resourceLoader == null) {
        throw new BeanDefinitionStoreException(
            "Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
    }
    // 加载多个资源
    if (resourceLoader instanceof ResourcePatternResolver) {
        // Resource pattern matching available.
        try {
            // location转换成了Resource
            Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
            int count = loadBeanDefinitions(resources);
            if (actualResources != null) {
                Collections.addAll(actualResources, resources);
            }
            if (logger.isTraceEnabled()) {
                logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
            }
            return count;
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException(
                "Could not resolve bean definition resource pattern [" + location + "]", ex);
        }
    }
    else {
        // 加载单个资源
        // Can only load single resources by absolute URL.
        Resource resource = resourceLoader.getResource(location);
        int count = loadBeanDefinitions(resource);
        if (actualResources != null) {
            actualResources.add(resource);
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
        }
        return count;
    }
}
```

XmlBeanDefinitionReader:loadBeanDefinitions(Resource resource)

如果设置了编码，使用EncodeResource进行编码

```java
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}
```

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    Assert.notNull(encodedResource, "EncodedResource must not be null");
    if (logger.isTraceEnabled()) {
        logger.trace("Loading XML bean definitions from " + encodedResource);
    }
    // ThreadLocal,当前资源
    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
    if (currentResources == null) {
        currentResources = new HashSet<>(4);
        this.resourcesCurrentlyBeingLoaded.set(currentResources);
    }

    if (!currentResources.add(encodedResource)) {
        throw new BeanDefinitionStoreException(
            "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }
    try {
        // 获取资源流
        InputStream inputStream = encodedResource.getResource().getInputStream();
        try {
            // 转换成InputSource
            InputSource inputSource = new InputSource(inputStream);
            // 如果设置了编码，则进行编码操作
            if (encodedResource.getEncoding() != null) {
                inputSource.setEncoding(encodedResource.getEncoding());
            }
            // 真正加载Bean
            return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        }
        finally {
            inputStream.close();
        }
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
            "IOException parsing XML document from " + encodedResource.getResource(), ex);
    }
    finally {
        currentResources.remove(encodedResource);
        if (currentResources.isEmpty()) {
            this.resourcesCurrentlyBeingLoaded.remove();
        }
    }
}

```

XmlBeanDefinitionReader:ldoLoadBeanDefinitions(InputSource inputSource, Resource resource)

解析xml，获取document对象。根据document对象和配置文件的resource来注册beanDefinition

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
    throws BeanDefinitionStoreException {

    try {
        // 解析xml，获取document
        Document doc = doLoadDocument(inputSource, resource);
        // 注册bean
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }
    catch (BeanDefinitionStoreException ex) {
        throw ex;
    }
}

```

XmlBeanDefinitionReader:registerBeanDefinitions(Document doc, Resource resource)

创建document的解析对象，调用documentReader.registerBeanDefinitions来注册beanDefinition

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    // 创建DefaultBeanDefinitionDocumentReader来解析document
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    // 注册前bean的数量，registry=DefaultListableRegistry
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    // 注册后bean的数量
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

registerBeanDefinitions由子类实现，因为创建的是DefaultBeanDefinitionDocumentReader对象来解析document，所以使用DefaultBeanDefinitionDocumentReader实现的registerBeanDefinitions方法

```java
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    doRegisterBeanDefinitions(doc.getDocumentElement());
}
```

DefaultBeanDefinitionDocumentReader:doRegisterBeanDefinitions()

```java
protected void doRegisterBeanDefinitions(Element root) {
   
    BeanDefinitionParserDelegate parent = this.delegate;
    this.delegate = createDelegate(getReaderContext(), root, parent);
	// 验证profile，BeanDefinitionParseDelegate
    if (this.delegate.isDefaultNamespace(root)) {
        String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
        if (StringUtils.hasText(profileSpec)) {
            String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
                profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
            if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
                                 "] not matching: " + getReaderContext().getResource());
                }
                return;
            }
        }
    }
    // 钩子函数，处理xml前做的事情，默认什么都不执行
    preProcessXml(root);
    // 解析beanDefinition
    parseBeanDefinitions(root, this.delegate);
    // 处理xml后做的事情
    postProcessXml(root);

    this.delegate = parent;
}
```

DefaultBeanDefinitionDocumentReader:parseBeanDefinitions()

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
                          // 解析spring默认的配置文件
						parseDefaultElement(ele, delegate);
					}
					else {
                          // 解析自定义的配置文件
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```

DefaultBeanDefinitionDocumentReader:parseDefaultElement

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    	// 配置文件中import了其他配置文件，将其他配置文件的bean加入到容器中
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
    	// 配置文件中指定了alias别名标签
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
    	// 处理配置文件中的bean标签
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
    	// 处理配置文件中的beans标签，递归处理
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```

DefaultBeanDefinitionDocumentReader:processBeanDefinition

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    	// bdHolder封装了BeanDefinition和beanName
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// 发生注册完成事件
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```

调用BeanDefinitionReaderUtils.registerBeanDefinition的方法将bean加入到容器当中：

参数BeanDefinitionRegistry是默认的DefaultListableBeanFactory。

```java
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// beanName为key,beanDefinition为value，注册到beanDefinitionMap中
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// 将bean的别名注册到容器当中，这里处理的是bean标签上的name别名，区别于外层处理的alias
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}

```

DefaultListableBeanFactory.registerBeanDefinition

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
                  // 验证是否有方法重写
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}
		// 容器中是否已经注册过了该beanName
		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
            
             // 如果beanName已经存在，检查是否允许重写，是否有权限等，
             // 检查通过就覆盖原来容器中的bean
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
            // bean已经开始创建，说明beanDefinition已经注册到容器当中了，是一个增量操作
			if (hasBeanCreationStarted()) {
				// 锁住原来的容器，往里面添加
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                      // 先添加原来容器的beanDefinition
					updatedDefinitions.addAll(this.beanDefinitionNames);
                     // 再添加新增的beanDefinition
					updatedDefinitions.add(beanName);
                     // 覆盖原来的容器
					this.beanDefinitionNames = updatedDefinitions;
                     // 更新原来的单例，因为原来是单例模式的，现在可能不再是单例了
					removeManualSingletonName(beanName);
				}
			}
			else {
				// 使用原来的容器，添加新的beanDefinition
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}
		// 检查是否有同名的beanDefinition已经在容器中注册
		if (existingDefinition != null || containsSingleton(beanName)) {
            // 重置所有已经注册过的beanDefinition的缓存
			resetBeanDefinition(beanName);
		}
	}
```

DefaultListableBeanFactory.updateManualSingletonNames

removeManualSingletonName实际上是调用 updateManualSingletonNames方法。

该方法会先判断是否已经创建bean了，如果已经创建了bean，对原来的容器加锁，重新创建新的单例。因为更新之后可能原来是单例的bean，现在已经不是单例了。

```java
private void updateManualSingletonNames(Consumer<Set<String>> action, Predicate<Set<String>> condition) {
		if (hasBeanCreationStarted()) {
            // 更新原先创建的单例
			synchronized (this.beanDefinitionMap) {
				if (condition.test(this.manualSingletonNames)) {
					Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
					action.accept(updatedSingletons);
					this.manualSingletonNames = updatedSingletons;
				}
			}
		}
		else {
			// 先前没有创建过，直接将新增的单例对象添加进去
			if (condition.test(this.manualSingletonNames)) {
				action.accept(this.manualSingletonNames);
			}
		}
	}
```

DefaultListableBeanFactory.resetBeanDefinition

```java
protected void resetBeanDefinition(String beanName) {
		// 清除合并出来的beanDefinition
		clearMergedBeanDefinition(beanName);

		// 将相关的单例删除掉
		destroySingleton(beanName);

		// 做一些后置的清除操作
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			if (processor instanceof MergedBeanDefinitionPostProcessor) {
				((MergedBeanDefinitionPostProcessor) processor).resetBeanDefinition(beanName);
			}
		}

		// 递归，如果一个bean的parent已经重新设置了，它也需要重新设置
		for (String bdName : this.beanDefinitionNames) {
			if (!beanName.equals(bdName)) {
				BeanDefinition bd = this.beanDefinitionMap.get(bdName);
				if (bd != null && beanName.equals(bd.getParentName())) {
					resetBeanDefinition(bdName);
				}
			}
		}
	}
```

**分析到这里，Spring就将在xml配置中的bean标签加入到了容器beanDefinitionMap中，整个过程调用的方法非常多，这是因为Spring为了扩展性，将代码逻辑分成了多个方法。**

小结：

* 加载配置文件
* 创建XmlBeanDefinitionReader读取配置文件，获取document对象
* 创建DefaultBeanDefinitionDocumentReader解析document
  * import标签：importBeanDefinitionResource
  * alias标签：processAliasRegistration
  * bean标签：processBeanDefinition
  * beans标签：递归处理，重复前3步的处理
* 拿到BeanDefinitionHolder，BeanDefinitionHolder封装了BeanDefinition和beanName。调用BeanDefinitionReaderUtils.registerBeanDefinition方法注册BeanDefinition
* BeanDefinitionReaderUtils调用BeanDefinitionRegistry的registerBeanDefinition方法
  * validate(),
  * 判断容器中是否已经存在beanName对应的BeanDefinition，如果有已存在的beanName,检查是否有权限，有角色覆盖，有的话就覆盖
  * 判断是否已经创建bean，如果已经创建了bean，说明了容器已经在使用了，这次操作是一个增加操作。需要将原来的容器加锁，并更新原来的单例对象(因为现在的对象可能是多例，而不再是单例)



### 注解方式注册BeanDefinition流程

xml配置文件的BeanDefinition注册都是在refresh方法中的obtainFreshBeanFactory刷新容器时注册的。

```java
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

而注解方式的BeanDefinition注册有3中情况：

1. 内置BeanDefinition：在容器的构造函数中被调用的时候就注册到容器
2. 用户自定义的标记@Configuration的BeanDefinition：在容器构造函数中调用registry方法注册到容器
3. 常规的BeanDefinition(不标记@Configuration注解)：在refresh方法中的后置处理器中注册到容器

```java
invokeBeanFactoryPostProcessors(beanFactory);
```



```java
public AnnotationConfigApplicationContext(String... basePackages) {
    this();
    scan(basePackages);
    refresh();
}
```

this();

初始化AnnotatedBeanDefinitionReader，ClassPathBeanDefinitionScanner路径扫描器。

```java
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

这里的参数this是指AnnotationConfigApplicationContext本身，它的成员变量中还有一个属性beanFactory，默认是DefaultListableBeanFactory.

与xml的方式对比，基于注解的容器创建beanFactory的时机比较早，这是因为有一些内置BeanDefinition需要先注册到容器中。

在创建AnnotatedBeanDefiniftionReader的时候，就已经将内置BeanDefinition注册到容器当中了。

将AnnotationConfigApplicationContext作为BeanDefinitionRegistry传递给AnnotatedBeanDefinitionReader的构造函数。

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}
```

在AnnotatedBeanDefinitionReader的构造函数中就已经调用registerAnnotationConfigProcessors来注册内部BeanDefiniton了。

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    Assert.notNull(environment, "Environment must not be null");
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

```java
public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
    registerAnnotationConfigProcessors(registry, null);
}
```

AnnotationConfigUtils.registerAnnotationConfigProcessors:

主要是使用DefaultListableBeanFactory，将内置BeanDefinition加入到容器当中。

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    // 将registry转换成DefaultListableBeanFactory
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
        
        // ...如果不存在内置的BeanDefinition,就加入到容器当中
    }
```

在创建AnnotedBeanDefinitionReader时将内置BeanDefiniton加入到容器中后，创建路径扫描器ClassPathBeanDefinitionScanner。

```java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
    this(registry, true);
}
```

useDefaultFilters = true

```java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
    this(registry, useDefaultFilters, getOrCreateEnvironment(registry));
}
```

判断BeanDefinitionRegistry是不是ResourceLoader的实例，如果是的话，就将它当成ResourceLoader

```java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
                                      Environment environment) {

    this(registry, useDefaultFilters, environment,
         (registry instanceof ResourceLoader ? (ResourceLoader) registry : null));
}
```

设置ClassPathBeanDefinitionScanner的一些属性。

```java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
                                      Environment environment, @Nullable ResourceLoader resourceLoader) {

    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    this.registry = registry;

    if (useDefaultFilters) {
        registerDefaultFilters();
    }
    setEnvironment(environment);
    setResourceLoader(resourceLoader);
}
```

在创建好AnnotedBeanDefinitionReader和ClassPathBeanDefinitonScanner之后，调用register方法开始注册标记有@Configuration的BeanDefiniton

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    register(componentClasses);
    refresh();
}
```

```java
public void register(Class<?>... componentClasses) {
    Assert.notEmpty(componentClasses, "At least one component class must be specified");
    // this.reader = AnnotedBeadnDefinitionReader
    this.reader.register(componentClasses);
}
```

进入AnnotedBeanDefinitionReader类中执行register方法：

```java
public void register(Class<?>... componentClasses) {
    for (Class<?> componentClass : componentClasses) {
        registerBean(componentClass);
    }
}
```

for循环扫描所有标记@Configuration的类，准备注册到容器中：

```java
public void registerBean(Class<?> beanClass) {
    doRegisterBean(beanClass, null, null, null, null);
}
```

调用AnnotedBeanDefinitionReader类中的doRegisterBean方法，将Class包装成BeanDefinition，处理类上标记的注解，再调用BeanDefinitionReaderUtils.registerBeanDefinition将BeanDefinitonHolder包装类注册到容器当中，后面的流程与xml的方式是相同的。

```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {

		// 将Class包装成BeanDefinition
		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}
		// 处理@Scope注解
		abd.setInstanceSupplier(supplier);
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		// 如果没有指定beanName,使用默认的生成机制
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		// 处理@Primary,@Lazy等注解，设置相应的属性
		if (qualifiers != null) {
			for (Class<? extends Annotation> qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}
		// 获取BeanDefinition的包装类
		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		// 是否依据Spring的Scope生成动态代理对象
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		// 注册BeanDefinition
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}
```

在调用完register方法将标注@Confiuration的BeanDefinition加入到容器后，会调用refresh方法将普通的BeanDefinition（没有标记@Configuration）注册到容器当中。

当使用xml方式注册BeanDefiniton时，调用的是refresh方法中的obtainFreshBeanFactory将BeanDefinition注册到容器中。

```java
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

而当使用注解方式注册普通的BeanDefiiton时，调用的是refresh方法中的invokeBeanFactoryPostProcessors来将BeanDefinition注册到容器中。

```java
invokeBeanFactoryPostProcessors(beanFactory);
```

invokeBeanFactoryPostProcessors方法中调用PostProcessorRegistrationDelegate代理来调用invokeBeanFactoryPostProcessors方法。

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    	// 使用PostProcessorRegistrationDelegate来执行后置处理器，
    	// beanFactory = DefaultListableBeanFactory
		PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

		// Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
		// (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
	}

```



```java

```





如何注册委托给你AnnotationBeanDefinitionRead类的regist方法来实现

doRegisterBean(beanClass, null, null, null, null);

```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {

		// 将Class包装成BeanDefinition
		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}
		// 处理@Scope注解
		abd.setInstanceSupplier(supplier);
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		// 如果没有指定beanName,使用默认的生成机制
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		// 处理@Primary,@Lazy等注解，设置相应的属性
		if (qualifiers != null) {
			for (Class<? extends Annotation> qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}
		// 获取BeanDefinition的包装类
		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		// 是否依据Spring的Scope生成动态代理对象
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		// 注册BeanDefinition
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}

```

如果使用@Component，@Controller，@Service，@Repository注解标记的类，是在refresh方法中才注册到容器当中。

但是与XML的方式不同，XML是在下面一行代码的时候进行注册到容器中：

```java
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

注解的方式是在容器调用其后置处理器的时候，触发对BeanDefinition的注册。

```java
invokeBeanFactoryPostProcessors(beanFactory);
```



### doGetBean

```
Object sharedInstance = getSingleton(beanName);
```

**从三级缓存中获取单例对象，**

二级缓存：在一级缓存中获取不到bean实例，且实例正在创建的时候会去二级缓存查找

三级缓存：在二级缓存中获取不到bean实例，且允许循环依赖，会去三级缓存中查找

```java
// allowEarlyReference：是否允许循环依赖，传参进来是true
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// 尝试从一级缓存中获取bean实例，一级缓存中保存的都是最终形态的bean
		// 一级缓存是线程安全的ConcurrencyHashMap
		Object singletonObject = this.singletonObjects.get(beanName);
		// 如果从一级缓存中获取不到bean实例，并且单例的bean当前正在创建
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			// 给一级缓存对象加锁，因为要操作缓存了
			synchronized (this.singletonObjects) {
				// 尝试从二级缓存中获取bean实例，二级缓存中保存的是还没给属性赋值的bean
				// 二级缓存是普通的HashMap,因为有了synchronized关键字,能保证是线程安全的了
				singletonObject = this.earlySingletonObjects.get(beanName);
				// 如果从二级缓存中获取不到bean实例，并且允许循环引用
				if (singletonObject == null && allowEarlyReference) {
					// 尝试从三级缓存中获取bean实例,三级缓存中存储的是ObjectFactory
					// 三级缓存也是普通的HashMap
					// ObjectFactory是Spring内部使用的,通过getObject方法可以获取Factory创建的bean实例
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						// 此时bean实例的属性可能还未被注入,将bean实例放进二级缓存中
						this.earlySingletonObjects.put(beanName, singletonObject);
						// 从三级缓存中移除，因为bean实例只能在三级缓存的任一个缓存中保存
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

 如果获取到单例对象，且参数为空，执行if逻辑
 这里判断参数是否为空，如果参数不为空，需要赋值操作，就不是直接返回bean了

```java
if (sharedInstance != null && args == null) {
			if (logger.isTraceEnabled()) {
				// bean还在创建，说明循环依赖了
				if (isSingletonCurrentlyInCreation(beanName)) {
					logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
							"' that is not fully initialized yet - a consequence of a circular reference");
				}
				else {
					logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
				}
			}
			// 无论是否循环依赖，都会执行；
			// 如果是普通的bean,则直接返回。如果是BeanFactory,则调用它的getObject()方法
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}
```

bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);

判断bean是不是FactoryBean，一系列检查判断后，是的话调用其getObject方法创建bean实例。

```java
protected Object getObjectForBeanInstance(
			Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

		// 判断beanName是否以&开头
		if (BeanFactoryUtils.isFactoryDereference(name)) {
			if (beanInstance instanceof NullBean) {
				return beanInstance;
			}
			// 如果beanName以&开头，但又不是FactoryBean,抛出异常
			if (!(beanInstance instanceof FactoryBean)) {
				throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
			}
			if (mbd != null) {
				mbd.isFactoryBean = true;
			}
			return beanInstance;
		}

		// 不是以&开头，是普通的bean，直接返回
		if (!(beanInstance instanceof FactoryBean)) {
			return beanInstance;
		}
		// 以下逻辑为FactoryBean创建bean实例返回
		Object object = null;
		if (mbd != null) {
			mbd.isFactoryBean = true;
		}
		else {
			// 单例模式下，FactoryBean仅会创建出一个bean实例
			// 因此需要优先从缓存获取
			// 这里的缓存不是三级缓存
			object = getCachedObjectForFactoryBean(beanName);
		}
		if (object == null) {
			// Return bean instance from factory.
			FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
			// 传参进来的BeanDefinition为空，尝试从容器中获取该beanName的BeanDefinition
			if (mbd == null && containsBeanDefinition(beanName)) {
				mbd = getMergedLocalBeanDefinition(beanName);
			}
			// synthetic标识某个类是不是Spring容器内部生成的bean
			boolean synthetic = (mbd != null && mbd.isSynthetic());
             // 调用getObject方法创建出bean实例，并放在factorybeanObject缓存中，下次直接从缓存中获取
			object = getObjectFromFactoryBean(factory, beanName, !synthetic);
		}
		return object;
	}
```

object = getObjectFromFactoryBean(factory, beanName, !synthetic);

调用getObject创建bean对象，考虑是否需要单例...并执行后置处理器方法后，将创建后的bean加入factorybeanObject缓存，下次直接从缓存中获取单例对象。

这里保证单例模式，使用的是双重检查机制。

```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
		// 如果需要在工厂模式下维持单例
		if (factory.isSingleton() && containsSingleton(beanName)) {
			// 双重检查机制保证单例，对一级缓存加锁，防止多线程下,有别的线程已完成对单例bean的实例化
			synchronized (getSingletonMutex()) {
				Object object = this.factoryBeanObjectCache.get(beanName);
				if (object == null) {
					// 调用工厂的getObject方法，创建bean实例
					object = doGetObjectFromFactoryBean(factory, beanName);
					// 看看此时是否有别的线程先创建好了bean实例，如果是，使用最先创建出来的bean，以保证单例
					Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
					if (alreadyThere != null) {
						object = alreadyThere;
					}
					else {
						// 该bean是否需要后置处理
						if (shouldPostProcess) {
							// 如果别的线程正在创建该bean，但是还没有进行后置处理，直接返回
							if (isSingletonCurrentlyInCreation(beanName)) {
								// Temporarily return non-post-processed object, not storing it yet..
								return object;
							}
							// 后置处理完成前，先加入缓存里锁定起来
							beforeSingletonCreation(beanName);
							try {
								// 触发BeanPostProcessor,第三方框架可以在此用AOP来包装bean实例
								object = postProcessObjectFromFactoryBean(object, beanName);
							}
							catch (Throwable ex) {
								throw new BeanCreationException(beanName,
										"Post-processing of FactoryBean's singleton object failed", ex);
							}
							finally {
								// 后置处理完成后，从缓存锁定的名字里清除
								afterSingletonCreation(beanName);
							}
						}
						if (containsSingleton(beanName)) {
							// 将其放入factorybeanObject的缓存，证明单例已经创建完成了
							this.factoryBeanObjectCache.put(beanName, object);
						}
					}
				}
				return object;
			}
		}
		else {
			// 如果不是单例，则直接创建返回bean实例
			Object object = doGetObjectFromFactoryBean(factory, beanName);
			if (shouldPostProcess) {
				try {
					object = postProcessObjectFromFactoryBean(object, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
				}
			}
			return object;
		}
	}
```

​	若scope为prototype,或者是单例模式但是缓存中还不存在，就执行else分支

**首先是对循环依赖的判断**，如果存在循环依赖，Spring无法处理，直接抛出异常

**如果不是循环依赖，且容器存在父容器，则递归去父容器中获取bean实例**，递归的方式分几种

* 如果父容器是AbstractBeanFactory的实例，调用其doGetBean方法递归
* 如果父容器不是AbstractBeanFactory的实例
  * 参数不为空，委托父容器根据beanName和args进行查找
  * 如果参数为空，但getBean传入的Class不为空，委托父容器根据beanName和Class去查找bean实例
  * 如果参数和Class都为空，委托父容器根据beanName查找

```java
else {
    // 如果scope为prototype并且正在创建中，基本上是循环依赖的情况
    // 比如A依赖于B,B又依赖于A(需要创建A),此时A还未创建完成，就形成了循环依赖
    // 针对prototype的循环依赖，spring无解，直接抛出异常
    if (isPrototypeCurrentlyInCreation(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
    
    BeanFactory parentBeanFactory = getParentBeanFactory();
    // 如果存在父容器且当前容器中找不到beanName,递归查找父容器
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
        // Not found -> check parent.
        // 主要是针对FactoryBean,将&重新加上
        String nameToLookup = originalBeanName(name);
        // 如果父容器依旧是AbstractBeanFactory的实例
        if (parentBeanFactory instanceof AbstractBeanFactory) {
            // 递归调用方法来查找
            return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                nameToLookup, requiredType, args, typeCheckOnly);
        }
        else if (args != null) {
            // Delegation to parent with explicit args.
            // 如果有参数，则委派父级容器根据指定名称和显示的参数查找
            return (T) parentBeanFactory.getBean(nameToLookup, args);
        }
        else if (requiredType != null) {
            // No args -> delegate to standard getBean method.
            // 委派父级容器根据指定名称和类型查找
            return parentBeanFactory.getBean(nameToLookup, requiredType);
        }
        else {
            // 委派父级容器根据指定名称查找，递归
            return (T) parentBeanFactory.getBean(nameToLookup);
        }
    }
    
}
```

如果能从父容器中递归查找出bean实例，就返回bean实例了。如果查找不到，就要开始创建bean实例了

typeCheckOnly是通过传参进来的，typeCheckOnly=true代表只是进行类型检查， 而不创建bean实例

```java
// typeCheckOnly是为了检查getBean是否仅仅是为了类型检查获取bean,而不是创建bean
if (!typeCheckOnly) {
    // 如果不是类型检查，则需要重新合并BeanDefinition,并标记已经创建或即将创建的beanName
    // 这一步只是设置了标记位，并未真正的进行操作
    markBeanAsCreated(beanName);
}
```

创建实例前，需要重新合并BeanDefinition，防止原来的数据改动，并且将已经创建或即将创建的beanName加入到alreadyCreated这个set集合中。而在markBeanAsCreated方法中，先设置好标记位

```java
protected void markBeanAsCreated(String beanName) {
   // 双重检查机制,
   if (!this.alreadyCreated.contains(beanName)) {
      // 即将操作mergedBeanDefinitions，先上锁保证线程安全
      synchronized (this.mergedBeanDefinitions) {
         if (!this.alreadyCreated.contains(beanName)) {
            // 将原先合并之后的RootBeanDefinition的需要重新合并的状态设置为true
            // 表示需要重新合并一遍,以防原数据的改动
            clearMergedBeanDefinition(beanName);
            // 将已经创建好的或正在创建的Bean的名称加到alreadyCreated这个缓存中
            this.alreadyCreated.add(beanName);
         }
      }
   }
}
```

接下来，**从当前容器中获取BeanDefinition实例，如果设置了depend-on属性，递归实例化显示依赖的depend-on实例**

```java
// 将父类的BeanDefinition与子类的BeanDefinition进行合并覆盖
final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
// 对合并好的BeanDefinition做验证,主要看是否为abstract的
// 如果BeanDefinition是abstract的，会抛出BeanIsAbstractException异常
checkMergedBeanDefinition(mbd, beanName, args);

// Guarantee initialization of beans that the current bean depends on.
// 如果当前bean设置了depend-on属性
// depend-on属性用来指定bean初始化以及销毁的顺序
String[] dependsOn = mbd.getDependsOn();
if (dependsOn != null) {
    for (String dep : dependsOn) {
        // 校验是否存在循环依赖,如果有,直接抛出异常
        // 注意这里的key是
        if (isDependent(beanName, dep)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                            "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
        }
        // 缓存依赖调用
        registerDependentBean(dep, beanName);
        try {
            // 递归调用getBean方法，注册bean之间的依赖
            getBean(dep);
        }
        catch (NoSuchBeanDefinitionException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                            "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
        }
    }
}
```

dependentBeanMap是一个ConcurrencyHashMap,它的key为beanName,values为dependentBeanName

```java
private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap<>(64);
```

```java
protected boolean isDependent(String beanName, String dependentBeanName) {
    synchronized (this.dependentBeanMap) {
        return isDependent(beanName, dependentBeanName, null);
    }
}
```

注册依赖关系，我依赖于谁，谁又依赖于我

```java
public void registerDependentBean(String beanName, String dependentBeanName) {
    String canonicalName = canonicalName(beanName);

    // key是beanName,value是beanName所需要的依赖bean实例的beanName
    synchronized (this.dependentBeanMap) {
        Set<String> dependentBeans =
            this.dependentBeanMap.computeIfAbsent(canonicalName, k -> new LinkedHashSet<>(8));
        if (!dependentBeans.add(dependentBeanName)) {
            return;
        }
    }

    // key是依赖，values是哪个beanName需要我这个依赖
    synchronized (this.dependenciesForBeanMap) {
        Set<String> dependenciesForBean =
            this.dependenciesForBeanMap.computeIfAbsent(dependentBeanName, k -> new LinkedHashSet<>(8));
        dependenciesForBean.add(canonicalName);
    }
}

```

在递归处理完显示指定的depend-on依赖后，开始创建bean实例，但是bean有不同的scope，有不同的逻辑处理。

我们来看下单例模式的bean实例创建：

调用子类实现的createBean方法创建bean实例

这里又遇到了老朋友getObjectForBeanInstance，它可以根据beanName是否已&开头，来决定使用哪种方法创建bean

* 如果是FactoryBean，则调用FactoryBean的getObject方法创建实例
* 如果是普通的bean，则直接返回

```java
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> {
        try {
            return createBean(beanName, mbd, args);
        }
        catch (BeansException ex) {
            // Explicitly remove instance from singleton cache: It might have been put there
            // eagerly by the creation process, to allow for circular reference resolution.
            // Also remove any beans that received a temporary reference to the bean.
            destroySingleton(beanName);
            throw ex;
        }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

在创建出bean实例后，对bean进行类型检查后返回。

小结：

1. 尝试从缓存中获取bean
2. 循环依赖的判断
3. 递归去父容器获取bean实例
4. 如果父容器中获取不到，从当前容器中获取BeanDefinition实例
5. 递归实例化显示依赖的depend-on
6. 根据不同的Scope采用不同的策略来创建bean实例
7. 对bean进行类型检查

