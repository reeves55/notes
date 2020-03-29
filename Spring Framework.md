# Spring Framework

```SpringApplication``` 这个类相当于一个执行器，负责把 ```ApplicationContext```创建并初始化好。



## Spring



### Bean容器

Spring最重要的其实就是Bean容器，这是Spring的核心，开始探索SpringBean容器的构建过程

一切的开始都从这段代码开始

```java
public class StartApplication {
    public static void main(String[] args) {
      	// 关键代码, 新建一个ApplicationContext
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(StartApplication.class);
}
```



###AnnotationConfigApplicationContext

从构造方法来看，创建一个 AnnotationConfigApplicationContext 对象分为3个步骤

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
  // 步骤1：调用无参构造方法
	this();
  // 步骤2：
	register(componentClasses);
  // 步骤3：
	refresh();
}
```



#### Step1: 调用无参构造方法

无参构造方法如下：

```java
// AnnotationConfigApplicationContext
public AnnotationConfigApplicationContext() {
	this.reader = new AnnotatedBeanDefinitionReader(this);
	this.scanner = new ClassPathBeanDefinitionScanner(this);
}

// 1. AnnotatedBeanDefinitionReader 构造方法
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
	Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
	Assert.notNull(environment, "Environment must not be null");
	this.registry = registry;
	this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
  // 最重要的一步，给 AnnotationConfigApplicationContext 加入 Processors
	AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}

// 2. ClassPathBeanDefinitionScanner 构造方法
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



#####AnnotatedBeanDefinitionReader 

实例化过程中，会向 AnnotationConfigApplicationContext 中加入一些Processors，这些Processors包括：

* org.springframework.context.annotation.internalConfigurationAnnotationProcessor -> ```ConfigurationClassPostProcessor```

* org.springframework.context.annotation.internalAutowiredAnnotationProcessor -> ```AutowiredAnnotationBeanPostProcessor```

* org.springframework.context.annotation.internalCommonAnnotationProcessor -> ```CommonAnnotationBeanPostProcessor```

* org.springframework.context.annotation.internalPersistenceAnnotationProcessor -> ```org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor```

  ⚠️注意：这个Processor加入到容器当中是有条件的，

* org.springframework.context.event.internalEventListenerProcessor ->

   ```EventListenerMethodProcessor```

* org.springframework.context.event.internalEventListenerFactory -> 

  ```DefaultEventListenerFactory```



##### ClassPathBeanDefinitionScanner

有一个属性叫做 ```includeFilters``` ，实例化过程中，会注册默认的过滤器，默认会添加一下过滤器：

* AnnotationTypeFilter( ```Component.class``` )

* new AnnotationTypeFilter( ```javax.annotation.ManagedBean.class``` )

  ⚠️注意：相应Class存在时，才会添加这个过滤器

* new AnnotationTypeFilter( ```javax.inject.Named``` )

  ⚠️注意：相应Class存在时，才会添加这个过滤器





#### Step2: 注册

主要是把 ```new AnnotationConfigApplicationContext(StartApplication.class)``` 这个构造方法传递的参数 ```StartApplication.class``` 注入到 AnnotationConfigApplicationContext 中去





#### Step3: 刷新

这个才是核心操作，

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// Prepare this context for refreshing.
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
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



一点一点来

##### prepareRefresh



##### obtainFreshBeanFactory

获取到当前ApplicationContext的初始化时的BeanFactory，要知道的是，不同类型的ApplicationContext所需要的初始化的BeanFactory实例是不一样的，所以obtainFreshBeanFactory基于不同的ApplicationContext可能返回不同的实例

```java
// AbstractApplicationContext

protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
	refreshBeanFactory();
	return getBeanFactory();
}

// 拓展点，模板类中的抽象方法
protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;
```



具体子类有哪些实现的方法呢❓

看一下 ```ClasspathXmlApplicationContext``` 类的实现

```java
@Override
protected final void refreshBeanFactory() throws BeansException {
	if (hasBeanFactory()) {
		destroyBeans();
		closeBeanFactory();
	}
	try {
		DefaultListableBeanFactory beanFactory = createBeanFactory();
		beanFactory.setSerializationId(getId());
		customizeBeanFactory(beanFactory);
    // 在这个步骤会解析xml配置文件，构建bean map
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



再看 ```AnnotationConfigApplicationContext``` 类的实现

```java
@Override
protected final void refreshBeanFactory() throws IllegalStateException {
	if (!this.refreshed.compareAndSet(false, true)) {
		throw new IllegalStateException(
				"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
	}
	this.beanFactory.setSerializationId(getId());
}
```



##### prepareBeanFactory



```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	// Tell the internal bean factory to use the context's class loader etc.
	beanFactory.setBeanClassLoader(getClassLoader());
	beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
	beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

	// Configure the bean factory with context callbacks.
	beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
	beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
	beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
	beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
	beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

	// BeanFactory interface not registered as resolvable type in a plain factory.
	// MessageSource registered (and found for autowiring) as a bean.
  // Map<Class<?>, Object> resolvableDependencies = new ConcurrentHashMap<>(16);
  // 把这些bean放到DefaultListableBeanFactory中的 resolvableDependencies 中去
	beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
	beanFactory.registerResolvableDependency(ResourceLoader.class, this);
	beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
	beanFactory.registerResolvableDependency(ApplicationContext.class, this);

	// Register early post-processor for detecting inner beans as ApplicationListeners.
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



##### postProcessBeanFactory

这是 AbstractApplicationContext 类提供给子类重写的方法，默认什么都不做，子类可重写，AnnotationConfigApplicationContext 并没有重写这个方法，所以啥也不做



##### invokeBeanFactoryPostProcessors

调用所有的 BeanFactoryPostProcesssor，BeanFactoryPostProcessors 是要对 BeanFactory 做一些操作的处理器，BeanFactory是一个Bean容器，那对它做操作无非就是注册Bean或者删除Bean之类的，在执行所有BeanFactoryPostProcessor的过程中，实际上是先执行所有的 BeanDefinitionRegistryPostProcessor，它是 BeanFactoryPostProcessor 的子类接口，由于它先于所有BeanFactoryPostProcessor，所以，可以在 BeanDefinitionRegistryPostProcessor 里面 注册新的 BeanFactoryPostProcessor 到容器中。



```java
// PostProcessorRegistrationDelegate

public static void invokeBeanFactoryPostProcessors(
		ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

  // 最先调用 BeanDefinitionRegistryPostProcessors
	Set<String> processedBeans = new HashSet<>();

	if (beanFactory instanceof BeanDefinitionRegistry) {
		BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
		List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
		List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

		for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
			if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
				BeanDefinitionRegistryPostProcessor registryProcessor =
						(BeanDefinitionRegistryPostProcessor) postProcessor;
				registryProcessor.postProcessBeanDefinitionRegistry(registry);
				registryProcessors.add(registryProcessor);
			}
			else {
				regularPostProcessors.add(postProcessor);
			}
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		// Separate between BeanDefinitionRegistryPostProcessors that implement
		// PriorityOrdered, Ordered, and the rest.
		List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();


    // 第一步、找到实现了 PriorityOrdered 接口的那些 BeanDefinitionRegistryPostProcessors
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
				processedBeans.add(ppName);
			}
		}
    // 根据 PriorityOrdered 来对这些 BeanDefinitionRegistryPostProcessors 进行排序
    // 然后按照顺序调用这些 Processor的 postProcessBeanDefinitionRegistry 方法
		sortPostProcessors(currentRegistryProcessors, beanFactory);
		registryProcessors.addAll(currentRegistryProcessors);
		invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
		currentRegistryProcessors.clear();

    // 第二步、找到实现了 Ordered 接口的那些 BeanDefinitionRegistryPostProcessors
		postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
		for (String ppName : postProcessorNames) {
			if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
				currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
				processedBeans.add(ppName);
			}
		}
    // 根据 Ordered 来对这些 BeanDefinitionRegistryPostProcessors 进行排序
    // 然后按照顺序调用这些 Processor的 postProcessBeanDefinitionRegistry 方法
		sortPostProcessors(currentRegistryProcessors, beanFactory);
		registryProcessors.addAll(currentRegistryProcessors);
		invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
		currentRegistryProcessors.clear();

		// 循环添加剩余的那些 BeanDefinitionRegistryPostProcessor，由于在执行过程中，可能又会
    // 添加新的 BeanDefinitionRegistryPostProcessor，所以有个 while 循环
		boolean reiterate = true;
		while (reiterate) {
			reiterate = false;
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
					reiterate = true;
				}
			}
      // 调用这些 Processor的 postProcessBeanDefinitionRegistry 方法
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();
		}

		// 调用上面找到的所有 BeanDefinitionRegistryPostProcessor 的 postProcessBeanFactory 方法
    // ⚠️ 注意，这和上面的 postProcessBeanDefinitionRegistry 不是一个方法
		invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
	}

	else {
		// Invoke factory processors registered with the context instance.
		invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
	}

  // 下面开始执行 BeanFactoryPostProcessor
	// Do not initialize FactoryBeans here: We need to leave all regular beans
	// uninitialized to let the bean factory post-processors apply to them!
	String[] postProcessorNames =
			beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

	// 把 BeanFactoryPostProcessor，分为 实现了 PriorityOrderd、实现了 Ordered、其他 共3类
	List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
	List<String> orderedPostProcessorNames = new ArrayList<>();
	List<String> nonOrderedPostProcessorNames = new ArrayList<>();
	for (String ppName : postProcessorNames) {
		if (processedBeans.contains(ppName)) {
			// skip - already processed in first phase above
		}
		else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
			priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
		}
		else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
			orderedPostProcessorNames.add(ppName);
		}
		else {
			nonOrderedPostProcessorNames.add(ppName);
		}
	}

	// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
	sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
	invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

	// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
	List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
	for (String postProcessorName : orderedPostProcessorNames) {
		orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
	}
	sortPostProcessors(orderedPostProcessors, beanFactory);
	invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

	// Finally, invoke all other BeanFactoryPostProcessors.
	List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
	for (String postProcessorName : nonOrderedPostProcessorNames) {
		nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
	}
	invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

	// Clear cached merged bean definitions since the post-processors might have
	// modified the original metadata, e.g. replacing placeholders in values...
	beanFactory.clearMetadataCache();
}
```







### Spring容器拓展点





#### 使用BeanPostProcessor自定义Bean





#### 使用BeanFactoryPostProcessor修改容器配置

BeanFactoryProcessor接口定义如下，需要注意的是，当 postProcessBeanFactory 方法被调用的时候，容器已经加载完毕了所有的beans，但是还没有实例化这些bean

```java
public interface BeanFactoryPostProcessor {

/**
 * Modify the application context's internal bean factory after its standard
 * initialization. All bean definitions will have been loaded, but no beans
 * will have been instantiated yet. This allows for overriding or adding
 * properties even to eager-initializing beans.
 * @param beanFactory the bean factory used by the application context
 * @throws org.springframework.beans.BeansException in case of errors
 */
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```



##### ConfigurationClassPostProcessor

ConfigurationClassPostProcessor 是一个 BeanDefinitionRegistryPostProcessor，这也意味着在运行 BeanFactoryPostProcessor之前，ConfigurationClassPostProcessor的 ```postProcessBeanDefinitionRegistry``` 方法会被执行，也就是在这里，完成了bean注册进容器

![ConfigurationClassPostProcessor](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/ConfigurationClassPostProcessor.png)



```java
// postProcessBeanDefinitionRegistry

static {
	candidateIndicators.add(Component.class.getName());
	candidateIndicators.add(ComponentScan.class.getName());
	candidateIndicators.add(Import.class.getName());
	candidateIndicators.add(ImportResource.class.getName());
}

public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
		List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
		String[] candidateNames = registry.getBeanDefinitionNames();

		for (String beanName : candidateNames) {
			BeanDefinition beanDef = registry.getBeanDefinition(beanName);
			if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
				if (logger.isDebugEnabled()) {
					logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
				}
			}
      // 哪些BeanDefinition可以作为配置解析的BeanDefinition呢，有下面4种
      // Component
      // ComponentScan
      // Import
      // ImportResource
			else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
				configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
			}
		}

		// Return immediately if no @Configuration classes were found
		if (configCandidates.isEmpty()) {
			return;
		}

		// Sort by previously determined @Order value, if applicable
		configCandidates.sort((bd1, bd2) -> {
			int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
			int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
			return Integer.compare(i1, i2);
		});

		// Detect any custom bean name generation strategy supplied through the enclosing application context
		SingletonBeanRegistry sbr = null;
		if (registry instanceof SingletonBeanRegistry) {
			sbr = (SingletonBeanRegistry) registry;
			if (!this.localBeanNameGeneratorSet) {
				BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
						AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
				if (generator != null) {
					this.componentScanBeanNameGenerator = generator;
					this.importBeanNameGenerator = generator;
				}
			}
		}

		if (this.environment == null) {
			this.environment = new StandardEnvironment();
		}

		// Parse each @Configuration class
		ConfigurationClassParser parser = new ConfigurationClassParser(
				this.metadataReaderFactory, this.problemReporter, this.environment,
				this.resourceLoader, this.componentScanBeanNameGenerator, registry);

		Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
		Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
		// 开始从含有配置信息的 BeanDefinition 解析配置
  	do {
			parser.parse(candidates);
			parser.validate();

			Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
			configClasses.removeAll(alreadyParsed);

			// Read the model and create bean definitions based on its content
			if (this.reader == null) {
				this.reader = new ConfigurationClassBeanDefinitionReader(
						registry, this.sourceExtractor, this.resourceLoader, this.environment,
						this.importBeanNameGenerator, parser.getImportRegistry());
			}
			this.reader.loadBeanDefinitions(configClasses);
			alreadyParsed.addAll(configClasses);

			candidates.clear();
			if (registry.getBeanDefinitionCount() > candidateNames.length) {
				String[] newCandidateNames = registry.getBeanDefinitionNames();
				Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
				Set<String> alreadyParsedClasses = new HashSet<>();
				for (ConfigurationClass configurationClass : alreadyParsed) {
					alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
				}
				for (String candidateName : newCandidateNames) {
					if (!oldCandidateNames.contains(candidateName)) {
						BeanDefinition bd = registry.getBeanDefinition(candidateName);
						if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
								!alreadyParsedClasses.contains(bd.getBeanClassName())) {
							candidates.add(new BeanDefinitionHolder(bd, candidateName));
						}
					}
				}
				candidateNames = newCandidateNames;
			}
		}
		while (!candidates.isEmpty());

		// Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
		if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
			sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
		}

		if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
			// Clear cache in externally provided MetadataReaderFactory; this is a no-op
			// for a shared cache since it'll be cleared by the ApplicationContext.
			((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
		}
	}
```





#### 使用FactoryBean自定义Bean的实例化逻辑











#### DefaultListableBeanFactory



关键属性：

* Comparator<Object> ```dependencyComparator```
* AutowireCandidateResolver ```autowireCandidateResolver```
* Map<String, BeanDefinition> ```beanDefinitionMap```





## SpringMVC



### HandlerMapping

```HandlerMapping``` 当中记录了请求和处理该请求的Bean和相应方法的映射。



###HandlerMethodArgumentResolver

```HandlerMethodArgumentResolver``` 是用来解析参数的，关键函数

```java
// 判断是否解析该参数
boolean supportsParameter(MethodParameter parameter);

// 解析参数
Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
```

Spring框架解析Controller方法中的参数是一个一个解析的，不同类型的参数可能使用的参数解析器是不同的，你可能这个方法有3个参数，一个是String类型，一个是@RequestBody注解参数，一个是普通类参数，那针对这三个参数，实际上解析使用的 ```HandlerMethodArgumentResolver``` 是不同的。

⚠️ 注意：这个HandlerMethodArgumentResolver具体什么类型和我们访问接口传递参数的方式，也就是请求的Content-type 无关，我们的api接口函数的参数类型可以是@RequestBody，同时我们方法用@GetMapping，是可以的，问题是不能成功解析出来，不能解析出来问题出现在@RequestBody这种方式指定了参数获取途径，如果请求的body没有内容，它是无法解析出参数值的。



那参数有几种类型呢❓

1. 普通类型，如String
2. 没有注解的POJO类型
3. 有注解的POJO类型



| 参数类型                                   | 解析类                             | 解析方式                                                     |
| ------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| ```@RequestBody POJO```                    | RequestResponseBodyMethodProcessor | 从请求体中解析                                               |
| 非简单类型POJO，```@ModelAttribute POJO``` | ModelAttributeMethodProcessor      | 从encoded-url中获取参数，支持GET和POST（x-www-form-urlencoded） |
| 普通类型（String, Integer等等）            |                                    |                                                              |



ModelAttributeMethodProcessor属性和值绑定的方法是WebDataBinder，其他地方应该也是用了这个方法，具体获取请求的参数值是通过 ```ServletRequest```， 调用```ServletRequest.getParameterValues(String paramName)``` 来获取其参数的值。



**如果参数类型是POJO对象**，那么就要把请求中的参数值设置为对象的属性的值，究竟是怎么设置参数对象属性的值呢，跟踪调用堆栈，可以找到对象是在 jackson-databind 库当中被反序列化的，```com/fasterxml/jackson/databind/deser/BeanDeserializer.class``` 类当中

```java
// ...
do {
    p.nextToken();
    SettableBeanProperty prop = _beanProperties.find(propName);
    if (prop != null) { // normal case
        try {
            // 关键操作
            prop.deserializeAndSet(p, ctxt, bean);
        } catch (Exception e) {
            wrapAndThrow(e, bean, propName, ctxt);
        }
        continue;
    }
    handleUnknownVanilla(p, ctxt, bean, propName);
} while ((propName = p.nextFieldName()) != null);
// ...



```



在jackson.databind库当中，<span style="color:red">设置一个对象属性的值的方式不只一种，</span>

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200323221239873.png" alt="image-20200323221239873" style="zoom:60%;float:left" />



```java
// MethodProperty使用setter方法来设置属性的值

@Override
public void deserializeAndSet(JsonParser p, DeserializationContext ctxt,
        Object instance) throws IOException
{
    Object value;
    if (p.hasToken(JsonToken.VALUE_NULL)) {
        if (_skipNulls) {
            return;
        }
        value = _nullProvider.getNullValue(ctxt);
    } else if (_valueTypeDeserializer == null) {
        value = _valueDeserializer.deserialize(p, ctxt);
        if (value == null) {
            if (_skipNulls) {
                return;
            }
            value = _nullProvider.getNullValue(ctxt);
        }
    } else {
        value = _valueDeserializer.deserializeWithType(p, ctxt, _valueTypeDeserializer);
    }
    try {
      	// 关键操作，实际上是调用属性的setter方法来设置对象属性的值
        _setter.invoke(instance, value);
    } catch (Exception e) {
        _throwAsIOE(p, e, value);
    }
}           
```



```java
// FieldProperty直接设置对象属性的值（对象属性得是public类型才行）

@Override
public void deserializeAndSet(JsonParser p,
        DeserializationContext ctxt, Object instance) throws IOException
{
    Object value;
    if (p.hasToken(JsonToken.VALUE_NULL)) {
        if (_skipNulls) {
            return;
        }
        value = _nullProvider.getNullValue(ctxt);
    } else if (_valueTypeDeserializer == null) {
        value = _valueDeserializer.deserialize(p, ctxt);
        // 04-May-2018, tatu: [databind#2023] Coercion from String (mostly) can give null
        if (value == null) {
            if (_skipNulls) {
                return;
            }
            value = _nullProvider.getNullValue(ctxt);
        }
    } else {
        value = _valueDeserializer.deserializeWithType(p, ctxt, _valueTypeDeserializer);
    }
    try {
        // 关键操作
        _field.set(instance, value);
    } catch (Exception e) {
        _throwAsIOE(p, e, value);
    }
}   
```







用来判断某种解析器是否支持解析某个参数，那不同的解析器判断自己能否解析的依据是不一定的，常用的解析器的解析条件如下：

* ```RequestParamMethodArgumentResolver``` 

  支持带有@RequestParam注解的参数或带有MultipartFile类型的参数

* ```RequestParamMapMethodArgumentResolver```

  支持带有@RequestParam注解的参数 && @RequestParam注解的属性value存在 && 参数类型是实现Map接口的属性

* ```PathVariableMethodArgumentResolver```

  支持带有@PathVariable注解的参数 且如果参数实现了Map接口，@PathVariable注解需带有value属性

* ```MatrixVariableMethodArgumentResolver```

  支持带有@MatrixVariable注解的参数 且如果参数实现了Map接口，@MatrixVariable注解需带有value属性

* ```RequestResponseBodyMethodProcessor```

  支持带有@RequestBody注解的参数

* ```ServletRequestMethodArgumentResolver```

  参数类型是实现或继承或是WebRequest、ServletRequest、MultipartRequest、HttpSession、Principal、Locale、TimeZone、InputStream、Reader、HttpMethod这些类。（这就是为何我们在Controller中的方法里添加一个HttpServletRequest参数，Spring会为我们自动获得HttpServletRequest对象的原因）

* ```ServletResponseMethodArgumentResolver```

  参数类型是实现或继承或是ServletResponse、OutputStream、Writer这些类

* ```RedirectAttributesMethodArgumentResolver```

   参数是实现了RedirectAttributes接口的类

* ```HttpEntityMethodProcessor```

  参数类型是HttpEntity



参数解析器 ```HandlerMethodArgumentResolver``` 的25种类型：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200323162002129.png" alt="image-20200323162002129" style="zoom:50%;float:left" />



### MessageConverter



参数解析器 ```HttpMessageConverter``` 的10种类型：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200323144702260.png" alt="image-20200323144702260" style="zoom:50%;float:left" />



### HandlerMethodReturnValueHandler

```HandlerMethodReturnValueHandler``` 是用来解析controller方法的返回值的，根据返回值的类型不同，也应该使用不同的解析器，常用的解析器如下：

* ```ModelAndViewMethodReturnValueHandler```

  返回值类型是ModelAndView或其子类

* ```ModelMethodProcessor```

  返回值类型是Model或其子类

* ```ViewMethodReturnValueHandler```

  返回值类型是View或其子类

* ```HttpHeadersReturnValueHandler```

  返回值类型是HttpHeaders或其子类 

* ```ModelAttributeMethodProcessor```

  返回值有@ModelAttribute注解

* ```ViewNameMethodReturnValueHandler```

  返回值是void或String





返回值解析器 ```HandlerMethodReturnValueHandler``` 的15种类型：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200323153046764.png" alt="image-20200323153046764" style="zoom:50%;float:left" />





> 整体流程图

![SpringMVC请求处理流程](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/SpringMVC请求处理流程.png)























> Questions

ConfigurableEnvironment在SpringApplication当中是用来做什么的❓





## SpringBoot



```AnnotationConfigServletWebServerApplicationContext``` 这种类型的ApplicationContext比较特别的一点就是，它在onRefresh步骤当中，启动了一个tomcat服务器

![image-20200220223930545](/Users/xiao/Library/Application Support/typora-user-images/image-20200220223930545.png)

### createWebServer调用栈

```AnnotationConfigServletWebServerApplicationContext``` 继承自 ```ServletWebServerApplicationContext```



```java
// 1. 开始入口点
private void createWebServer() {
  	// webServer : null
		WebServer webServer = this.webServer;
  	// servletContext : null
		ServletContext servletContext = getServletContext();
		if (webServer == null && servletContext == null) {
			ServletWebServerFactory factory = getWebServerFactory();
			this.webServer = factory.getWebServer(getSelfInitializer());
		}
		else if (servletContext != null) {
			try {
				getSelfInitializer().onStartup(servletContext);
			}
			catch (ServletException ex) {
				throw new ApplicationContextException("Cannot initialize servlet context", ex);
			}
		}
		initPropertySources();
	}

// 2. getWebServerFactory
protected ServletWebServerFactory getWebServerFactory() {
		// Use bean names so that we don't consider the hierarchy
		String[] beanNames = getBeanFactory().getBeanNamesForType(ServletWebServerFactory.class);
		if (beanNames.length == 0) {
			throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to missing "
					+ "ServletWebServerFactory bean.");
		}
		if (beanNames.length > 1) {
			throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to multiple "
					+ "ServletWebServerFactory beans : " + StringUtils.arrayToCommaDelimitedString(beanNames));
		}
  
  	// 实际执行过程中，发现ServletWebServerFactory接口的bean只有一个
    // 就是 TomcatServletWebServerFactory
		return getBeanFactory().getBean(beanNames[0], ServletWebServerFactory.class);
	}

// 3. TomcatServletWebServerFactory.getWebServer

// org.apache.coyote.http11.Http11NioProtocol
private String protocol = DEFAULT_PROTOCOL; 

public WebServer getWebServer(ServletContextInitializer... initializers) {
		Tomcat tomcat = new Tomcat();
  	// baseDir : null
		File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
		tomcat.setBaseDir(baseDir.getAbsolutePath());
  	// DEFAULT_PROTOCOL
		Connector connector = new Connector(this.protocol);
		tomcat.getService().addConnector(connector);
		customizeConnector(connector);
		tomcat.setConnector(connector);
		tomcat.getHost().setAutoDeploy(false);
		configureEngine(tomcat.getEngine());
		for (Connector additionalConnector : this.additionalTomcatConnectors) {
			tomcat.getService().addConnector(additionalConnector);
		}
		prepareContext(tomcat.getHost(), initializers);
		return getTomcatWebServer(tomcat);
	}
```



