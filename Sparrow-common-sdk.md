# Sparrow-common-sdk

sparrow-common-sdk做什么？如何做？



##概况

| module                             |     父module      | 功能                                                         |
| ---------------------------------- | :---------------: | ------------------------------------------------------------ |
| **plugin-repository**              |         -         | **sparrow提供的API服务接口（Peer+Cathaya）**                 |
| sparrow-adaptor-plugin             | plugin-repository | 数据适配接口                                                 |
| sparrow-block-plugin               | plugin-repository | 区块信息查询接口（单channel/多channel）                      |
| sparrow-common-plugin              | plugin-repository | 区块链基础功能接口：chain read/write，二级节点间通信，文本零知识证明 |
| sparrow-direct-plugin              | plugin-repository |                                                              |
| sparrow-message-interactive-plugin | plugin-repository |                                                              |
| sparrow-mobile-permission-plugin   | plugin-repository |                                                              |
| sparrow-plugin-citrus              | plugin-repository |                                                              |
| sparrow-plugin-permission          | plugin-repository |                                                              |
| sparrow-supervise-plugin           | plugin-repository |                                                              |
| **service-repository**             |         -         | **为plugin提供service**                                      |
| **sparrow-middle**                 |         -         | **与Peer交互前，整合Cathaya数据加解密，对数据做处理**        |
|                                    |                   |                                                              |
| **sparrow-core**                   |         -         | **Http调用Peer API底层依赖，即调用tx server**                |
| **sparrow-direct-plugin**          |         -         | **明文上链相关API**                                          |
| **sparrow-remoting**               |         -         | **调用区块链Peer节点API**                                    |
| sparrow-remoting-api               | sparrow-remoting  |                                                              |
| sparrow-remoting-multi-chain       | sparrow-remoting  |                                                              |
| sparrow-remoting-peer              | sparrow-remoting  |                                                              |
| sparrow-remoting-txsvr             | sparrow-remoting  |                                                              |



###调用流程

客户端 ---> plugin ---> service ---> remoting



sparrow-middle：定义了一些数据和加密相关的数据库sql



####sparrow-block-service









#### 后记

1. Spring为何使用接口注入？如何使用？

   使用接口注入可以**解耦**，可以通过注入不同的实现类来提供不同的功能，比如项目开发初期，注入一个mock的实现类，然后真正的类写好了再注入真正的类。

   如果使用```@Autowired```来自动注入，需要被自动注入的实现类需要用```@Component```注解标识，如果一个接口只有一个实现类，那么让Spring可以扫描到相关类就可以了，但是如果一个接口的实现类有多个，就需要@Autowired配合使用```@Qualifier("xxx")```，加上```@Component("xxx")```来唯一标志一个实现类。

2. Java SecurityRandom？



## sparrow-block-plugin



BlockController调用流程

```java
BlockController
  ↓
interface BlockService (BlockOperation)
  ↓
BlockTemplate
  ↓
interface ChainCRUDService (DefaultChainCRUDService), BlockSupport
(ChainCRUDService其实就是区块链上数据的读和写)
  ↓
ChainSupport, interface ChainCodeInvoker (PeerChainCodeInvoker, TxSvrChainCodeInvoker), Executor, interface ChainSourceStrategy (DefaultChainSourceStrategy, MultiChainSourceStrategy), ChainInvoker
  ↓
interface GlobalConfigurationManager, ChainSourceHolder
```







## sparrow-common-plugin



###单条数据上链调用流程

```java
ChainWriteController
  ↓
interface OperateChainService (DefaultOperateChainService, PartitionOperateChainService)
(链上数据操作的服务接口，包含数据上链、查询等功能，有两个实现类，用哪个类需要在配置文件中指定，但是因为在PartitionAutoConfiguration 中配置了优先使用 PartitionOperateChainService，因此如果引入了 sparrow-partition-service依赖，并且配置文件中制定了相关配置值，则优先使用 PartitionOperateChainService 这个实现类)
  ↓
ChainWriteOperation
（）
  ↓
ChainDataService
  ↓
interface ChainCRUDService (DefaultChainCRUDService)
  ↓
ChainInvoker
  ↓
interface ChainCodeInvoker (PeerChainCodeInvoker, TxSvrChainCodeInvoker)
  ↓
GrpcProcessor
  ↓
ClusterService
  ↓
interface ClusterStrategy (AbstractClusterStrategy, FailfastStrategy, FailoverStrategy, FailsafeStrategy)
  |
PeerServerRequester.request(), FailfastStrategy.doCall() / FailoverStrategy.doCall() / FailsafeStrategy.doCall()
```



| Class / Interface             | Function                  |
| ----------------------------- | ------------------------- |
| interface OperateChainService | 读写链上数据接口          |
| OperateSupport                | 链上数据读写辅助工具类    |
| ChainReadOperation            | 链上数据**读**操作service |
| ChainWriteOperation           | 链上数据**写**操作service |
|                               |                           |





###Spring解析配置文件？

不同场景下需要使用不同的配置，如何 <span style="color:red">使用配置文件</span> 灵活切换项目功能：

```@Configuration``` + ```@EnableConfigurationProperties``` + ```@ConditionalOnProperty```

```java
// 指明当前类是bean的配置类，包含bean的定义，功能类似于以前的xml配置bean的功能
@Configuration
// 启用 @ConfigurationProperties 注解的类
@EnableConfigurationProperties(ApplicationProperties.class)
// 当配置信息有指定值的时候，才启用此配置类，spring才会管理此配置类配置的bean
@ConditionalOnProperty(prefix = "project.config", value = "enable", havingValue = "true")
public class ApplicationConfiguration {
  // 写这个配置环境下各种需要Spring管理的 @Bean
  @Bean
  // 
  @Primary
  public MyBean() {...}
}
```



**如何变成一个可插拔的可替换组件？**

要实现的功能：一个功能可能有多种实现方式，如何动态地使用不同实现组件，而不影响到调用组件的上层，首先各个不同实现的组件各为一个module，组件module中包含一个配置文件，XXXConfiguration，然后使用上面的几个注解 ```@Configuration```，```@EnableConfigurationProperties```，```@ConditionalOnProperty``` 在使用的时候，

1. 引入要使用的组件依赖；
2. 在配置文件中启用该组件启用条件；

还有一种可插拔的方式是只要引用了某个组件，这个组件就能够使用，但这属于一个功能只有一个实现的组件情况，上面👆这个是一个功能我想在组件之间切换的情况。







### Spring是如何启动的

SpringApplication.run()运作流程？

```java
/**
	 * Static helper that can be used to run a {@link SpringApplication} from the
	 * specified sources using default settings and user supplied arguments.
	 * @param primarySources the primary sources to load
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return the running {@link ApplicationContext}
	 */
	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
```

**先 new -> 再 run**

####new的时候，做了什么事情❓

```java
/**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param resourceLoader the resource loader to use
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #setSources(Set)
	 */
	@SuppressWarnings({ "unchecked", "rawtypes" })
  // 传进来的 resourceLoader == null
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```



1. 设置resourceLoader（这里就是null）；
2. 设置primarySources（就是启动类）；
3. 设置当前web应用的类型（检测classpath下是否有指定的类，有则属于特定类型，共有三种类型：**NONE**, **SERVLET**, **REACTIVE**）；
4. 设置initializers（设置spring.factories当中指定的initializer（实现```ApplicationContextInitializer```接口）到spring中）；
5. 设置listeners（设置spring.factories当中指定的listener（实现```ApplicationListener```接口）到spring中）；
6. 设置含有main方法的主类；



##### 判断当前web应用类型

```java
//REACTIVE_WEB_ENVIRONMENT_CLASS=org.springframework.web.reactive.DispatcherHandler
//MVC_WEB_ENVIRONMENT_CLASS=org.springframework.web.servlet.DispatcherServlet
//MVC_WEB_ENVIRONMENT_CLASS={"javax.servlet.Servlet","org.springframework.web.context.ConfigurableWebApplicationContext"}
//这里，默认就是WebApplicationType.SERVLET
private WebApplicationType deduceWebApplicationType() {
  // isPresent的判断方法很简单，就是尝试使用Class.forName加载指定的类，如果没有加载到，捕捉到相应的Exception，那么这个类就没有，否则返回相应的Class实例
	if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
		&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
		return WebApplicationType.REACTIVE;
	}
	for (String className : WEB_ENVIRONMENT_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	return WebApplicationType.SERVLET;
}
```



**setInitializers 和 setListeners**

注意到 ```setInitializers```和```setListeners```方法都用到了```getSpringFactoriesInstance```方法

```java
// 依据spring.factories，加载指定类型的类
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

// SpringFactoriesLoader.loadFactoryNames
// 实际调用下面的loadSpringFactories方法
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
          // FACTORIES_RESOURCE_LOCATION: META-INF/spring.factories
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```



如果需要读取spring.factories中自己配置的类，可以使用 ```SpringFactoriesLoader``` 类的这个方法

```java
public static <T> List<T> loadFactories(Class<T> factoryClass, @Nullable ClassLoader classLoader)
```





**spring.factories是用来干嘛的？**

用来实现自动化配置的，也就是引入了一个依赖之后，这个依赖可以被自动配置。



Spring核心就是一个IoC容器，





####run的时候，做了什么事情❓



```java
/**
	 * Run the Spring application, creating and refreshing a new
	 * {@link ApplicationContext}.
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return a running {@link ApplicationContext}
	 */
	public ConfigurableApplicationContext run(String... args) {
    // 用来统计spring启动时间
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
    // Spring IoC 容器
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
    // 见下面👇解释
		SpringApplicationRunListeners listeners = getRunListeners(args);
    // 通知所有 run listeners，SpringApplication正在启动
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
      // 打印 Springboot banner
			Banner printedBanner = printBanner(environment);
      // 创建核心的ApplicationContext
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			// 
      prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```



##### SpringApplicationRunListener

这个listener是用来监听Spring启动步骤的，包含这些方法，可以看出来，对应启动过程的各个时间点：

```java
public interface SpringApplicationRunListener {
  // SpringApplication.run 刚刚开始执行
  default void starting(){}
  // 
  default void environmentPrepared(ConfigurableEnvironment environment){}
  default void contextPrepared(ConfigurableApplicationContext context){}
  default void contextLoaded(ConfigurableApplicationContext context){}
  default void started(ConfigurableApplicationContext context){}
  default void running(ConfigurableApplicationContext context){}
  default void failed(ConfigurableApplicationContext context, Throwable exception){}
}
```



下面这段代码是获取所有 ```SpringApplicationRunListener```

```java
SpringApplicationRunListeners listeners = getRunListeners(args);

// getRunListeners
private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
	// 还是解析classpath下面的所有spring.factories文件，找到SpringApplicationRunListener指定的类
  return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}
```



##### prepareContext



```java
	private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
			SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		postProcessApplicationContext(context);
    // 循环调用实现ApplicationContextInitializer接口的listenr的initialize方法
		applyInitializers(context);
    // 通知监听SpringApplicationRunListeners context prepared事件
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
    // 通知监听SpringApplicationRunListeners context loaded事件
		listeners.contextLoaded(context);
	}
```

