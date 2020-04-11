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



###构建ApplicationContext实例

所有的 ApplicationContext 实例化过程都要经历过两个主要步骤：

1. 做好 ```refresh``` 调用之前的准备工作
2. 调用 ```refresh``` 模板方法



![ApplicationContext继承关系](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ApplicationContext继承关系.png)



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



####AnnotationConfigApplicationContext

##### 调用无参构造方法

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



######AnnotatedBeanDefinitionReader 

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



###### ClassPathBeanDefinitionScanner

有一个属性叫做 ```includeFilters``` ，实例化过程中，会注册默认的过滤器，默认会添加一下过滤器：

* AnnotationTypeFilter( ```Component.class``` )

* new AnnotationTypeFilter( ```javax.annotation.ManagedBean.class``` )

  ⚠️注意：相应Class存在时，才会添加这个过滤器

* new AnnotationTypeFilter( ```javax.inject.Named``` )

  ⚠️注意：相应Class存在时，才会添加这个过滤器



##### 注册配置类

主要是把 ```new AnnotationConfigApplicationContext(StartApplication.class)``` 这个构造方法传递的参数 ```StartApplication.class``` 注入到 AnnotationConfigApplicationContext 中去

```java
@Override
public void register(Class<?>... componentClasses) {
	Assert.notEmpty(componentClasses, "At least one component class must be specified");
	this.reader.register(componentClasses);
}
```





### AbstractApplicatoinContext.refresh

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

#### 1. prepareRefresh





#### 2. obtainFreshBeanFactory

获取到当前ApplicationContext的初始化时的BeanFactory，要知道的是，不同类型的ApplicationContext所需要的初始化的BeanFactory实例是不一样的，所以obtainFreshBeanFactory基于不同的ApplicationContext可能返回不同的实例

```java
// AbstractApplicationContext

protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
	refreshBeanFactory();
	return getBeanFactory();
}

// 模板类中的抽象方法，具体逻辑由子类实现
protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;
```



具体子类有哪些实现的方法呢❓看一下 ```ClasspathXmlApplicationContext``` 类的实现

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
    // 在这个步骤会解析xml配置文件，得到所有的BeanDefinition，并放入 beanDefinitionMap 中
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



#### 3. prepareBeanFactory

主要是对 ```DefaultListableBeanFactory``` 下面几个属性进行初始设置：

* ClassLoader                                                    ```beanClassLoader```
* BeanExpressionResolver                               ```beanExpressionResolver```
* Set<PropertyEditorRegistrar>      ``` propertyEditorRegistrars```
* List<BeanPostProcessor>                    ```beanPostProcessors```
* Set<Class<?>>                                                ```ignoredDependencyInterfaces```
* Map<Class<?>, Object>                                 ```resolvableDependencies```
* ClassLoader                                                   ```tempClassLoader```
* Map<String, Object>                                     ```singletonObjects```
* Map<String, ObjectFactory<?>>                   ```singletonFactories```
* Map<String, Object>                                     ```earlySingletonObjects```
* Set<String>                                              ```registeredSingletons```



```java
// beanFactory 实际上是一个 DefaultListableBeanFactory对象
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	// ClassLoader beanClassLoader
  beanFactory.setBeanClassLoader(getClassLoader());
  // BeanExpressionResolver beanExpressionResolver
	beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
  // Set<PropertyEditorRegistrar> propertyEditorRegistrars
	beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

	// List<BeanPostProcessor> beanPostProcessors
	beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  // Set<Class<?>> ignoredDependencyInterfaces
	beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
	beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
	beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
	beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
	beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

  // Map<Class<?>, Object> resolvableDependencies
  // 把这些bean放到 resolvableDependencies 中去
	beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
	beanFactory.registerResolvableDependency(ResourceLoader.class, this);
	beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
	beanFactory.registerResolvableDependency(ApplicationContext.class, this);

	// Register early post-processor for detecting inner beans as ApplicationListeners.
  // List<BeanPostProcessor> beanPostProcessors
	beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

	// Detect a LoadTimeWeaver and prepare for weaving, if found.
	if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
		beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
		// ClassLoader tempClassLoader
		beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
	}

	// Register default environment beans.
	if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
    // Map<String, Object> singletonObjects.put
    // Map<String, ObjectFactory<?>> singletonFactories.remove
    // Map<String, Object> earlySingletonObjects.remove
    // Set<String> registeredSingletons.add
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



#### 4. postProcessBeanFactory

暂时不明白这个方法究竟是来干什么的❓

AnnotationConfigApplicationContext 和 XmlClasspathApplicationContext 都没有重写这个方法，啥也没做



#### 5. invokeBeanFactoryPostProcessors

从方法名字上看，意思就是执行所有的 BeanFactoryPostProcessor ，实际上执行的有两种类型：

* ```BeanFactoryPostProcessor```
* ```BeanDefinitionRegistryPostProcessor```

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/BeanDefinitionRegistryPostProcessor2.png" alt="BeanDefinitionRegistryPostProcessor2" style="zoom:50%;float:left" />



执行过程是这样的：

> 第一阶段

1. 先获取 <span style="color:red">**ApplicationContext**</span> 的 ```beanFactoryPostProcessors``` 中所有的 BeanDefinitionRegistryPostProcessor 类型的bean，然后调用它们的 postProcessBeanDefinitionRegistry 方法，还没找到 ```beanFactoryPostProcessors``` 这个属性在哪里设置其值的❓；
2. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 ```beanDefinitionNames``` 属性和 ```manualSingletonNames``` 属性中找到 对应的类型为 BeanDefinitionRegistryPostProcessor 并且实现了 ```PriorityOrdered``` 接口的 bean，将这些bean排序之后，实例化bean，执行它们的 postProcessBeanDefinitionRegistry 方法；
3. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 beanDefinitionNames 属性和 manualSingletonNames 属性中找到 对应的类型为 BeanDefinitionRegistryPostProcessor 并且实现了 ```Ordered``` 接口的 bean，将这些bean排序之后，实例化bean，执行它们的 postProcessBeanDefinitionRegistry 方法；
4. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 beanDefinitionNames 属性和 manualSingletonNames 属性中找到 对应的类型为 BeanDefinitionRegistryPostProcessor 的所有剩余 bean，实例化bean，并执行它们的 postProcessBeanDefinitionRegistry 方法；
5. 执行上面👆所有 BeanDefinitionRegistryPostProcessor 的 postProcessBeanFactory 方法（因为BeanDefinitionRegistry继承自BeanFactoryPostProcessor，也有 postProcessBeanFactory 方法 ）；
6. 执行 <span style="color:red">**ApplicationContext**</span> 的 ```beanFactoryPostProcessors``` 中所有不是 BeanDefinitionRegistryPostProcessor 类型的其他 BeanFactoryPostProcessor 对象的 postProcessBeanFactory 方法；

> 第二阶段

7. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 ```beanDefinitionNames``` 属性和 ```manualSingletonNames``` 属性中找到 对应的类型为 BeanFactoryPostProcessor （去除第一阶段处理过程的 BeanDefinitionRegistryPostProcessor），并实现了 ```PriorityOrdered``` 接口的bean，实例化bean，并调用其 postProcessBeanFactory 方法；
8. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 beanDefinitionNames 属性和 manualSingletonNames 属性中找到 对应的类型为 BeanFactoryPostProcessor （去除第一阶段处理过程的 BeanDefinitionRegistryPostProcessor），并实现了 ```Ordered``` 接口的bean，实例化bean，并调用其 postProcessBeanFactory 方法；
9. 从 <span style="color:blue"> **DefaultListableBeanFactory**</span>  的 beanDefinitionNames 属性和 manualSingletonNames 属性中找到 对应的类型为 BeanFactoryPostProcessor （去除第一阶段处理过程的 BeanDefinitionRegistryPostProcessor）剩余的bean，实例化bean，并调用其 postProcessBeanFactory 方法；

> 第三阶段

10. 清理metadata cache，有待考究





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



##### BeanDefinitionRegistryPostProcessor

目前只有一个实现类，就是 ```ConfigurationClassPostProcessor```，ConfigurationClassPostProcessor 是一个 BeanDefinitionRegistryPostProcessor，这也意味着在执行所有的BeanFactoryPostProcessor之前，ConfigurationClassPostProcessor的 ```postProcessBeanDefinitionRegistry``` 方法会被执行，也就是通过这个入口，ConfigurationClassPostProcessor 找到并解析了配置类，将我们注解的bean注册进容器

![ConfigurationClassPostProcessor](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/ConfigurationClassPostProcessor.png)



```java
// ConfigurationClassPostProcessor

static {
	candidateIndicators.add(Component.class.getName());
	candidateIndicators.add(ComponentScan.class.getName());
	candidateIndicators.add(Import.class.getName());
	candidateIndicators.add(ImportResource.class.getName());
}

/**
 * Derive further bean definitions from the configuration classes in the registry.
 */
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
	int registryId = System.identityHashCode(registry);
	if (this.registriesPostProcessed.contains(registryId)) {
		throw new IllegalStateException("postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
	}
	if (this.factoriesPostProcessed.contains(registryId)) {
		throw new IllegalStateException("postProcessBeanFactory already called on this post-processor against " + registry);
	}
	this.registriesPostProcessed.add(registryId);

	processConfigBeanDefinitions(registry);
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



// ConfigurationClassParser
// 上面👆执行 parser.parse(candidates); 最终会调用这个方法
protected final SourceClass doProcessConfigurationClass(
		ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
		throws IOException {

	if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
		// Recursively process any member (nested) classes first
		processMemberClasses(configClass, sourceClass, filter);
	}

	// Process any @PropertySource annotations
	for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
			sourceClass.getMetadata(), PropertySources.class,
			org.springframework.context.annotation.PropertySource.class)) {
		if (this.environment instanceof ConfigurableEnvironment) {
			processPropertySource(propertySource);
		}
		else {
			logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
					"]. Reason: Environment must implement ConfigurableEnvironment");
		}
	}

	// Process any @ComponentScan annotations
	Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
			sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
	if (!componentScans.isEmpty() &&
			!this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
		for (AnnotationAttributes componentScan : componentScans) {
			// The config class is annotated with @ComponentScan -> perform the scan immediately
			Set<BeanDefinitionHolder> scannedBeanDefinitions =
					this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
			// Check the set of scanned definitions for any further config classes and parse recursively if needed
			for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
				BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
				if (bdCand == null) {
					bdCand = holder.getBeanDefinition();
				}
				if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
					parse(bdCand.getBeanClassName(), holder.getBeanName());
				}
			}
		}
	}

	// Process any @Import annotations
	processImports(configClass, sourceClass, getImports(sourceClass), filter, true);

	// Process any @ImportResource annotations
	AnnotationAttributes importResource =
			AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
	if (importResource != null) {
		String[] resources = importResource.getStringArray("locations");
		Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
		for (String resource : resources) {
			String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
			configClass.addImportedResource(resolvedResource, readerClass);
		}
	}

	// Process individual @Bean methods
	Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
	for (MethodMetadata methodMetadata : beanMethods) {
		configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
	}

	// Process default methods on interfaces
	processInterfaces(configClass, sourceClass);

	// Process superclass, if any
	if (sourceClass.getMetadata().hasSuperClass()) {
		String superclass = sourceClass.getMetadata().getSuperClassName();
		if (superclass != null && !superclass.startsWith("java") &&
				!this.knownSuperclasses.containsKey(superclass)) {
			this.knownSuperclasses.put(superclass, configClass);
			// Superclass found, return its annotation metadata and recurse
			return sourceClass.getSuperClass();
		}
	}

	// No superclass -> processing is complete
	return null;
}
```





##### BeanFactoryPostProcessor

举个🌰。。。





#### 6. registerBeanPostProcessors

把所有 BeanFactory 中的所有类型为BeanPostProcessors 的 BeanDefinition ```实例化之后```，按照顺序 添加到 BeanFactory 的 ```List<BeanPostProcessor> beanPostProcessors``` 属性中

 

```java
// PostProcessorRegistrationDelegate
// 调用链最后会调用这个方法
public static void registerBeanPostProcessors(
		ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

	String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

	// Register BeanPostProcessorChecker that logs an info message when
	// a bean is created during BeanPostProcessor instantiation, i.e. when
	// a bean is not eligible for getting processed by all BeanPostProcessors.
	int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
	beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

	// Separate between BeanPostProcessors that implement PriorityOrdered,
	// Ordered, and the rest.
	List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
	List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
	List<String> orderedPostProcessorNames = new ArrayList<>();
	List<String> nonOrderedPostProcessorNames = new ArrayList<>();
	for (String ppName : postProcessorNames) {
		if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
      // 注意 ⚠️：getBean 方法会实例化bean
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			priorityOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
			orderedPostProcessorNames.add(ppName);
		}
		else {
			nonOrderedPostProcessorNames.add(ppName);
		}
	}

	// First, register the BeanPostProcessors that implement PriorityOrdered.
	sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
	registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

	// Next, register the BeanPostProcessors that implement Ordered.
	List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
	for (String ppName : orderedPostProcessorNames) {
		BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
		orderedPostProcessors.add(pp);
		if (pp instanceof MergedBeanDefinitionPostProcessor) {
			internalPostProcessors.add(pp);
		}
	}
	sortPostProcessors(orderedPostProcessors, beanFactory);
	registerBeanPostProcessors(beanFactory, orderedPostProcessors);

	// Now, register all regular BeanPostProcessors.
	List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
	for (String ppName : nonOrderedPostProcessorNames) {
		BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
		nonOrderedPostProcessors.add(pp);
		if (pp instanceof MergedBeanDefinitionPostProcessor) {
			internalPostProcessors.add(pp);
		}
	}
	registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

	// Finally, re-register all internal BeanPostProcessors.
	sortPostProcessors(internalPostProcessors, beanFactory);
	registerBeanPostProcessors(beanFactory, internalPostProcessors);

	// Re-register post-processor for detecting inner beans as ApplicationListeners,
	// moving it to the end of the processor chain (for picking up proxies etc).
	beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}
```





#### 7. initMessageSource





#### 8. initApplicationEventMulticaster





#### 9. onRefresh





#### 10. registerListeners





#### 11. finishBeanFactoryInitialization

实例化所有的非懒加载的单例bean，





#### createBean 🌟

这是核心方法，但是这个方法也很复杂，所以具体又分为几个步骤

1. resolveBeanClass；
2. prepareMethodOverrides；
3. resolveBeforeInstantiation；
4. doCreateBean；



##### resolveBeanClass

**获取到bean的真实Class对象**



```java
/**
 * AbstractBeanFactory
 */
protected Class<?> resolveBeanClass(final RootBeanDefinition mbd, String beanName, final Class<?>... typesToMatch)
		throws CannotLoadBeanClassException {

	try {
    // BeanDefinition ->
    // return (this.beanClass instanceof Class);
		if (mbd.hasBeanClass()) {
			return mbd.getBeanClass();
		}
		if (System.getSecurityManager() != null) {
			return AccessController.doPrivileged((PrivilegedExceptionAction<Class<?>>) () ->
				doResolveBeanClass(mbd, typesToMatch), getAccessControlContext());
		}
		else {
			return doResolveBeanClass(mbd, typesToMatch);
		}
	}
	catch (PrivilegedActionException pae) {
		//...
}
  

private Class<?> doResolveBeanClass(RootBeanDefinition mbd, Class<?>... typesToMatch)
			throws ClassNotFoundException {

		ClassLoader beanClassLoader = getBeanClassLoader();
		ClassLoader dynamicLoader = beanClassLoader;
		boolean freshResolve = false;

		if (!ObjectUtils.isEmpty(typesToMatch)) {
			// When just doing type checks (i.e. not creating an actual instance yet),
			// use the specified temporary class loader (e.g. in a weaving scenario).
			ClassLoader tempClassLoader = getTempClassLoader();
			if (tempClassLoader != null) {
				dynamicLoader = tempClassLoader;
				freshResolve = true;
				if (tempClassLoader instanceof DecoratingClassLoader) {
					DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
					for (Class<?> typeToMatch : typesToMatch) {
						dcl.excludeClass(typeToMatch.getName());
					}
				}
			}
		}
		// 如果 BeanDefinition的 beanClass 属性是 Class类型，则返回 Class.name
  	// 如果 beanClass 属性是 String类型，则直接返回 beanClass
		String className = mbd.getBeanClassName();
		if (className != null) {
			Object evaluated = evaluateBeanDefinitionString(className, mbd);
			if (!className.equals(evaluated)) {
				// A dynamically resolved expression, supported as of 4.2...
				if (evaluated instanceof Class) {
					return (Class<?>) evaluated;
				}
				else if (evaluated instanceof String) {
					className = (String) evaluated;
					freshResolve = true;
				}
				else {
					throw new IllegalStateException("Invalid class name expression result: " + evaluated);
				}
			}
			if (freshResolve) {
				// When resolving against a temporary class loader, exit early in order
				// to avoid storing the resolved Class in the bean definition.
				if (dynamicLoader != null) {
					try {
						return dynamicLoader.loadClass(className);
					}
					catch (ClassNotFoundException ex) {
						if (logger.isTraceEnabled()) {
							logger.trace("Could not load class [" + className + "] from " + dynamicLoader + ": " + ex);
						}
					}
				}
				return ClassUtils.forName(className, dynamicLoader);
			}
		}

		// Resolve regularly, caching the result in the BeanDefinition...
		return mbd.resolveBeanClass(beanClassLoader);
	}
```



##### prepareMethodOverrides



```java
/**
 * BeanDefinition
 */
public void prepareMethodOverrides() throws BeanDefinitionValidationException {
	// Check that lookup methods exist and determine their overloaded status.
  // !this.methodOverrides.isEmpty()
	if (hasMethodOverrides()) {
		getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);
	}
}


protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {
	int count = ClassUtils.getMethodCountForName(getBeanClass(), mo.getMethodName());
	if (count == 0) {
		throw new BeanDefinitionValidationException(
				"Invalid method override: no method with name '" + mo.getMethodName() +
				"' on class [" + getBeanClassName() + "]");
	}
	else if (count == 1) {
		// Mark override as not overloaded, to avoid the overhead of arg type checking.
		mo.setOverloaded(false);
	}
}


```



##### resolveBeforeInstantiation

如果这一步有返回值，那就不往下面走了，直接返回bean。

```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
	Object bean = null;
	if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
		// Make sure bean class is actually resolved at this point.
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			Class<?> targetType = determineTargetType(beanName, mbd);
			if (targetType != null) {
				bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
				if (bean != null) {
					bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
				}
			}
		}
		mbd.beforeInstantiationResolved = (bean != null);
	}
	return bean;
}


protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
	for (BeanPostProcessor bp : getBeanPostProcessors()) {
		if (bp instanceof InstantiationAwareBeanPostProcessor) {
			InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
			Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
			if (result != null) {
				return result;
			}
		}
	}
	return null;
}

public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
	Object result = existingBean;
	for (BeanPostProcessor processor : getBeanPostProcessors()) {
		Object current = processor.postProcessAfterInitialization(result, beanName);
		if (current == null) {
			return result;
		}
		result = current;
	}
	return result;
}
```





##### doCreateBean





```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}
```





###### createBeanInstance

创建bean对象，但不设置其属性值，功能类似于new，这里只是创建了bean class指定的对象，还远不是一个bean，bean的属性都还没有注入，只是作为一个类的对象存在，而不是Spring当中一个可以使用的bean，创建一个对象主要有几种方式：

* 通过Bean定义的 instanceSupplier 实例化POJO对象；
* 通过Bean定义的 工厂方法 实例化POJO对象；
* 通过Bean的构造方法 实例化POJO对象；



```java
/**
 * AbstractAutowireCapableBeanFactory
 *
 * Create a new instance for the specified bean, using an appropriate instantiation strategy: factory method, constructor autowiring, or simple instantiation.
 */
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
	
  // ① 拿到bean的class，由于在定义bean的时候，class可以有多种表示方法，可以直接指定class全限定名，
  // 也可以使用 SpEL表达式来指定等等，所以要根据用户定义的bean class属性具体的值，解析出bean的 
  // Class 对象
  // ② 这个步骤还有一个目的，如果没有加载类，则加载类，Class.forName
	Class<?> beanClass = resolveBeanClass(mbd, beanName);

  // beanClass.getModifiers() 返回 int 值，是类的修饰符，包含 public,private,protected,
  // final,static,abstract,interface
  // 这里要判断类是否既不是 public 的，也不允许访问非 public 的构造方法
  // 如果是这样，那就没办法创建bean对象了，抛出异常 ⚠️
	if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
		throw new BeanCreationException(mbd.getResourceDescription(), beanName,
				"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
	}

  // Spring5.0新增的实例化策略,如果设置了该策略,将会覆盖构造方法和工厂方法实例化策略
  // 如果 BeanDefinition 定义了 instanceSupplier，则调用 instanceSupplier.get()
  // 拿到 bean对象，直接返回 🔚
	Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
	if (instanceSupplier != null) {
		return obtainFromSupplier(instanceSupplier, beanName);
	}
	
  // 如果 BeanDefinition 定义了 factoryMethodName，则调用
  // ConstructorResolver.instantiateUsingFactoryMethod 来创建 bean 对象，直接返回 🔚
	if (mbd.getFactoryMethodName() != null) {
		return instantiateUsingFactoryMethod(beanName, mbd, args);
	}

	// Shortcut when re-creating the same bean...
  // ③ 当创建一个相同的bean时,使用之间保存的快照
  // 这里可能会有一个疑问,什么时候会创建相同的bean呢?
  // 		③-->① 单例模式: Spring不会缓存该模式的实例,那么对于单例模式的bean,什么时候会用到该实例化
  //    策略呢? 我们知道对于IoC容器除了可以索取bean之外,还能销毁bean,当我们调用
  //    xmlBeanFactory.destroyBean(myBeanName,myBeanInstance),销毁bean时,
  //    容器是不会销毁已经解析的构造函数快照的,如果再次调用xmlBeanFactory.getBean(myBeanName)时,
  //    就会使用该策略了.
  // 		③-->② 原型模式: 对于该模式的理解就简单了,IoC容器不会缓存原型模式bean的实例,当我们第二次向容
  //    器索取同一个bean时,就会使用该策略了.
	boolean resolved = false;
	boolean autowireNecessary = false;
	if (args == null) {
		synchronized (mbd.constructorArgumentLock) {
      // resolvedConstructorOrFactoryMethod 实际上是上一次创建bean实例时使用的构造方法
      // 或者是工厂方法，如果此次是第一次创建bean实例，则会将这次调用的构造方法
      // 或者是工厂方法，保存在这个变量里
			if (mbd.resolvedConstructorOrFactoryMethod != null) {
				resolved = true;
        // autowireNecessary 标识 构造方法调用时的参数是不是也已经解析过了
				autowireNecessary = mbd.constructorArgumentsResolved;
			}
		}
	}
	if (resolved) {
		if (autowireNecessary) {
			return autowireConstructor(beanName, mbd, null, null);
		}
		else {
			return instantiateBean(beanName, mbd);
		}
	}

	// Candidate constructors for autowiring?
	Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
	if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
			mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
		return autowireConstructor(beanName, mbd, ctors, args);
	}

	// Preferred constructors for default construction?
	ctors = mbd.getPreferredConstructors();
	if (ctors != null) {
		return autowireConstructor(beanName, mbd, ctors, null);
	}

	// No special handling: simply use no-arg constructor.
	return instantiateBean(beanName, mbd);
}
```



> 最朴实的 ```instantiateBean``` - 默认构造方法实例化bean

```getInstantiationStrategy()``` 方法会返回BeanFactory中 ```instantiationStrategy``` 属性的值，这个属性默认值是 ```new CglibSubclassingInstantiationStrategy()```。



```java
/*
 * AbstractAutowireCapableBeanFactory
 */
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
	try {
		Object beanInstance;
		final BeanFactory parent = this;
		if (System.getSecurityManager() != null) {
			beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
					getInstantiationStrategy().instantiate(mbd, beanName, parent),
					getAccessControlContext());
		}
		else {
      // 实际上调用的是 CglibSubclassingInstantiationStrategy.instantiate(mbd, beanName, parent)
			beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
		}
		BeanWrapper bw = new BeanWrapperImpl(beanInstance);
		initBeanWrapper(bw);
		return bw;
	}
	catch (Throwable ex) {
		throw new BeanCreationException(
				mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
	}
}
```



```java
/*
 * CglibSubclassingInstantiationStrategy
 */
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
	// Don't override the class with CGLIB if no overrides.
  // 如果BeanDefinition不存在method overrides，则使用反射实例化对象
	if (!bd.hasMethodOverrides()) {
    // 如果是反射的话，首先要解析出来，用什么构造函数来实例化对象
		Constructor<?> constructorToUse;
		synchronized (bd.constructorArgumentLock) {
			constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
			if (constructorToUse == null) {
				final Class<?> clazz = bd.getBeanClass();
				if (clazz.isInterface()) {
					throw new BeanInstantiationException(clazz, "Specified class is an interface");
				}
				try {
					if (System.getSecurityManager() != null) {
						constructorToUse = AccessController.doPrivileged(
								(PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
					}
					else {
						constructorToUse = clazz.getDeclaredConstructor();
					}
					bd.resolvedConstructorOrFactoryMethod = constructorToUse;
				}
				catch (Throwable ex) {
					throw new BeanInstantiationException(clazz, "No default constructor found", ex);
				}
			}
		}
    // 最终调用 Constructor.newInstance(args)
		return BeanUtils.instantiateClass(constructorToUse);
	}
	else {
		// Must generate CGLIB subclass.
		return instantiateWithMethodInjection(bd, beanName, owner);
	}
}
```



重点看一下通过Cglib如何实例化bean对象的



```java
/*
 * CglibSubclassingInstantiationStrategy
 */
@Override
protected Object instantiateWithMethodInjection(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner,
		@Nullable Constructor<?> ctor, Object... args) {
	// ctor == null -> true
	// Must generate CGLIB subclass...
	return new CglibSubclassCreator(bd, owner).instantiate(ctor, args);
}


/*
 * CglibSubclassingInstantiationStrategy.CglibSubclassCreator
 */
public Object instantiate(@Nullable Constructor<?> ctor, Object... args) {
	Class<?> subclass = createEnhancedSubclass(this.beanDefinition);
	Object instance;
	if (ctor == null) {
		instance = BeanUtils.instantiateClass(subclass);
	}
	else {
		try {
			Constructor<?> enhancedSubclassConstructor = subclass.getConstructor(ctor.getParameterTypes());
			instance = enhancedSubclassConstructor.newInstance(args);
		}
		catch (Exception ex) {
			throw new BeanInstantiationException(this.beanDefinition.getBeanClass(),
					"Failed to invoke constructor for CGLIB enhanced subclass [" + subclass.getName() + "]", ex);
		}
	}
	// SPR-10785: set callbacks directly on the instance instead of in the
	// enhanced class (via the Enhancer) in order to avoid memory leaks.
	Factory factory = (Factory) instance;
	factory.setCallbacks(new Callback[] {NoOp.INSTANCE,
			new LookupOverrideMethodInterceptor(this.beanDefinition, this.owner),
			new ReplaceOverrideMethodInterceptor(this.beanDefinition, this.owner)});
	return instance;
}
```





> 用 ```instantiateUsingFactoryMethod``` - 工厂方法实例化bean

如果 BeanDefinition 对象当中的 ```factoryMethodName``` 属性值不为 null，则使用工厂方法实例化bean，工厂方法分为两种，一种是 **实例工厂方法**，一种是 **静态工厂方法**。实例工厂方法指的是这个bean需要调用某个bean实例的某个方法来获取，静态工厂方法就是直接调用某个类的静态方法来获取bean实例。



```xml
<!-- 静态工厂方法（指定静态工厂类和工厂方法） -->
<bean id="bmwCar" class="com.home.factoryMethod.CarStaticFactory" factory-method="getCar">
    <constructor-arg value="3"></constructor-arg>           
</bean>

<!--实例工厂方法（指定工厂bean和工厂方法）-->
<bean id="car4" factory-bean="carFactory" factory-method="getCar">
	<constructor-arg value="4"></constructor-arg>           
</bean>

<bean id="carFactory" class="com.home.factoryMethod.CarInstanceFactory" />
```



```java
// ConstructorResolver

public BeanWrapper instantiateUsingFactoryMethod(
			final String beanName, final RootBeanDefinition mbd, @Nullable final Object[] explicitArgs) {

    BeanWrapperImpl bw = new BeanWrapperImpl();
    this.beanFactory.initBeanWrapper(bw);

    Object factoryBean;
    Class<?> factoryClass;
    boolean isStatic;

    // 1、判断是实例工厂还是静态工厂方法
    // 获取factoryBeanName，即配置文件中的工厂方法
    // 注意：静态工厂方法是没有factoryBeanName的，所以如果factoryBeanName不为null，
    // 则一定是实例工厂方法，否则就是静态工厂方法
    String factoryBeanName = mbd.getFactoryBeanName();
    if (factoryBeanName != null) {
        if (factoryBeanName.equals(beanName)) {
            throw new BeanDefinitionStoreException(mbd.getResourceDescription(), beanName,
                    "factory-bean reference points back to the same bean definition");
        }
        // 获取factoryBeanName实例
        factoryBean = this.beanFactory.getBean(factoryBeanName);
        if (mbd.isSingleton() && this.beanFactory.containsSingleton(beanName)) {
            throw new ImplicitlyAppearedSingletonException();
        }
        factoryClass = factoryBean.getClass();
        isStatic = false;
    }
    else {
        // It's a static factory method on the bean class.
        if (!mbd.hasBeanClass()) {
            throw new BeanDefinitionStoreException(mbd.getResourceDescription(), beanName,
                    "bean definition declares neither a bean class nor a factory-bean reference");
        }
        factoryBean = null;
        factoryClass = mbd.getBeanClass();
        isStatic = true;
    }

    Method factoryMethodToUse = null;
    ArgumentsHolder argsHolderToUse = null;
    Object[] argsToUse = null;

    // 2、判断有无显式指定参数,如果有则优先使用,如xmlBeanFactory.getBean("cat", args);
    if (explicitArgs != null) {
        argsToUse = explicitArgs;
    }
    // 3、从缓存中加载工厂方法和构造函数参数
    // 如果第一次加载bean的工厂方法，则将成功解析出的工厂方法赋值给 BeanDefinition 的
    // resolvedConstructorOrFactoryMethod，并且如果解析出的候选工厂方法只有一个且没有可以传给
    // 工厂方法的参数，并且BeanDefinition也没有保存工厂方法参数，说明这就是唯一的一个可用工厂方法
  	// 了，就把这个方法设置为 factoryMethodToIntrospect 属性的值，
    else {
        Object[] argsToResolve = null;
        synchronized (mbd.constructorArgumentLock) {
            factoryMethodToUse = (Method) mbd.resolvedConstructorOrFactoryMethod;
            if (factoryMethodToUse != null && mbd.constructorArgumentsResolved) {
                // Found a cached factory method...
                argsToUse = mbd.resolvedConstructorArguments;
                if (argsToUse == null) {
                    argsToResolve = mbd.preparedConstructorArguments;
                }
            }
        }
        if (argsToResolve != null) {
            argsToUse = resolvePreparedArguments(beanName, mbd, bw, factoryMethodToUse, argsToResolve, true);
        }
    }

    // 4、未能从缓存中加载工厂方法和构造函数参数，
    // 则解析并确定应该使用哪一个工厂方法实例化，并解析构造函数参数
    if (factoryMethodToUse == null || argsToUse == null) {
        // Need to determine the factory method...
        // Try all methods with this name to see if they match the given arguments.
        factoryClass = ClassUtils.getUserClass(factoryClass);

        // 4.1、获取factoryClass中所有的方法
        Method[] rawCandidates = getCandidateMethods(factoryClass, mbd);
        List<Method> candidateSet = new ArrayList<>();
        // 4.2、从获取到的所有方法中筛选出可能符合条件的方法
        for (Method candidate : rawCandidates) {
            // isStatic-->是之前解析过的，如果当前工厂方法是静态工厂方法，那么isStatic-->true;
            // 如果当前工厂方法是实例工厂方法，那么isStatic-->false
            // 通过Modifier.isStatic(candidate.getModifiers()) == isStatic判断，过滤掉一部分不符合条件的方法
            // mbd.isFactoryMethod(candidate)-->判断是否工厂方法
            if (Modifier.isStatic(candidate.getModifiers()) == isStatic && mbd.isFactoryMethod(candidate)) {
                candidateSet.add(candidate);
            }
        }
        // 4.3、对候选工厂方法按照方法的参数个数进行倒序排序
        Method[] candidates = candidateSet.toArray(new Method[0]);
        AutowireUtils.sortFactoryMethods(candidates);

        ConstructorArgumentValues resolvedValues = null;
        boolean autowiring = (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR);
        int minTypeDiffWeight = Integer.MAX_VALUE;
        Set<Method> ambiguousFactoryMethods = null;

        // 4.4、定义最小工厂方法参数个数，以备循环解析候选方法使用
        int minNrOfArgs;
        if (explicitArgs != null) {
            // 如指定参数不为空，则使用指定参数个数作为最小方法参数个数
            minNrOfArgs = explicitArgs.length;
        }
        else {
            // 尝试从BeanDefinition中加载构造函数信息，以确定最小方法参数个数
            if (mbd.hasConstructorArgumentValues()) {
                ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
                resolvedValues = new ConstructorArgumentValues();
                minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
            }
            else {
                // 以上均未能获取，则将最小方法参数个数置为0
                minNrOfArgs = 0;
            }
        }

        // 5.循环候选工厂方法，并确定最终使用的工厂方法
        LinkedList<UnsatisfiedDependencyException> causes = null;
        for (Method candidate : candidates) {
            Class<?>[] paramTypes = candidate.getParameterTypes();

            // 如果候选方法的参数个数大于之前定义的最小方法参数个数，则继续循环
            // 如果候选方法的参数个数为1，而定义的最小方法参数个数为2，那么肯定不会使用该方法作为工厂方法
            if (paramTypes.length >= minNrOfArgs) {
                ArgumentsHolder argsHolder;

                // 5.1 、指定方法参数不为空，则优先使用指定方法参数
                if (explicitArgs != null){
                    // Explicit arguments given -> arguments length must match exactly.
                    if (paramTypes.length != explicitArgs.length) {
                        continue;
                    }
                    argsHolder = new ArgumentsHolder(explicitArgs);
                }
                // 5.2、否则，解析方法参数
                else {
                    // Resolved constructor arguments: type conversion and/or autowiring necessary.
                    try {
                        String[] paramNames = null;
                        ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
                        if (pnd != null) {
                            paramNames = pnd.getParameterNames(candidate);
                        }
                        argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw,
                                paramTypes, paramNames, candidate, autowiring, candidates.length == 1);
                    }
                    catch (UnsatisfiedDependencyException ex) {
                        // Swallow and try next overloaded factory method.
                        if (causes == null) {
                            causes = new LinkedList<>();
                        }
                        causes.add(ex);
                        continue;
                    }
                }
                // 5.3、 通过构造函数参数权重对比,得出最适合使用的构造函数
                // 先判断是返回是在宽松模式下解析构造函数还是在严格模式下解析构造函数。(默认是宽松模式)
                // 对于宽松模式:例如构造函数为(String name,int age),配置文件中定义(value="美美",value="3")
                // 	 那么对于age来讲,配置文件中的"3",可以被解析为int也可以被解析为String,
                //   这个时候就需要来判断参数的权重,使用ConstructorResolver的静态内部类ArgumentsHolder分别对字符型和数字型的参数做权重判断
                //   权重越小,则说明构造函数越匹配
                // 对于严格模式:严格返回权重值,不会根据分别比较而返回比对值
                // minTypeDiffWeight = Integer.MAX_VALUE;而权重比较返回结果都是在Integer.MAX_VALUE做减法,起返回最大值为Integer.MAX_VALUE
                int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
                        argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
                // Choose this factory method if it represents the closest match.
                if (typeDiffWeight < minTypeDiffWeight) {
                    factoryMethodToUse = candidate;
                    argsHolderToUse = argsHolder;
                    argsToUse = argsHolder.arguments;
                    minTypeDiffWeight = typeDiffWeight;
                    ambiguousFactoryMethods = null;
                }
                // 5.4 若果未能明确解析出需要使用的工厂方法
                // 对于具有相同数量参数的方法，如果具有相同类型的差异权值，则收集这些候选对象，并最终引发歧义异常。
                // 但是，只在非宽松的构造函数解析模式中执行该检查，并显式地忽略覆盖的方法(具有相同的参数签名)。
                // Find out about ambiguity: In case of the same type difference weight
                // for methods with the same number of parameters, collect such candidates
                // and eventually raise an ambiguity exception.
                // However, only perform that check in non-lenient constructor resolution mode,
                // and explicitly ignore overridden methods (with the same parameter signature).
                else if (factoryMethodToUse != null && typeDiffWeight == minTypeDiffWeight &&
                        !mbd.isLenientConstructorResolution() &&
                        paramTypes.length == factoryMethodToUse.getParameterCount() &&
                        !Arrays.equals(paramTypes, factoryMethodToUse.getParameterTypes())) {
                    if (ambiguousFactoryMethods == null) {
                        ambiguousFactoryMethods = new LinkedHashSet<>();
                        ambiguousFactoryMethods.add(factoryMethodToUse);
                    }
                    ambiguousFactoryMethods.add(candidate);
                }
            }
        }

        // 6、异常处理
        if (factoryMethodToUse == null) {
            if (causes != null) {
                UnsatisfiedDependencyException ex = causes.removeLast();
                for (Exception cause : causes) {
                    this.beanFactory.onSuppressedException(cause);
                }
                throw ex;
            }
            List<String> argTypes = new ArrayList<>(minNrOfArgs);
            if (explicitArgs != null) {
                for (Object arg : explicitArgs) {
                    argTypes.add(arg != null ? arg.getClass().getSimpleName() : "null");
                }
            }
            else if (resolvedValues != null){
                Set<ValueHolder> valueHolders = new LinkedHashSet<>(resolvedValues.getArgumentCount());
                valueHolders.addAll(resolvedValues.getIndexedArgumentValues().values());
                valueHolders.addAll(resolvedValues.getGenericArgumentValues());
                for (ValueHolder value : valueHolders) {
                    String argType = (value.getType() != null ? ClassUtils.getShortName(value.getType()) :
                            (value.getValue() != null ? value.getValue().getClass().getSimpleName() : "null"));
                    argTypes.add(argType);
                }
            }
            String argDesc = StringUtils.collectionToCommaDelimitedString(argTypes);
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "No matching factory method found: " +
                    (mbd.getFactoryBeanName() != null ?
                        "factory bean '" + mbd.getFactoryBeanName() + "'; " : "") +
                    "factory method '" + mbd.getFactoryMethodName() + "(" + argDesc + ")'. " +
                    "Check that a method with the specified name " +
                    (minNrOfArgs > 0 ? "and arguments " : "") +
                    "exists and that it is " +
                    (isStatic ? "static" : "non-static") + ".");
        }
        else if (void.class == factoryMethodToUse.getReturnType()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "Invalid factory method '" + mbd.getFactoryMethodName() +
                    "': needs to have a non-void return type!");
        }
        else if (ambiguousFactoryMethods != null) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "Ambiguous factory method matches found in bean '" + beanName + "' " +
                    "(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " +
                    ambiguousFactoryMethods);
        }

        if (explicitArgs == null && argsHolderToUse != null) {
            argsHolderToUse.storeCache(mbd, factoryMethodToUse);
        }
    }

    // 7、根据解析出来的工厂方法创建对应的bean的实例
    try {
        Object beanInstance;

        if (System.getSecurityManager() != null) {
            final Object fb = factoryBean;
            final Method factoryMethod = factoryMethodToUse;
            final Object[] args = argsToUse;
            beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
                    this.beanFactory.getInstantiationStrategy().instantiate(mbd, beanName, this.beanFactory, fb, factoryMethod, args),
                    this.beanFactory.getAccessControlContext());
        }
        else {
            beanInstance = this.beanFactory.getInstantiationStrategy().instantiate(
                    mbd, beanName, this.beanFactory, factoryBean, factoryMethodToUse, argsToUse);
        }

        bw.setBeanInstance(beanInstance);
        return bw;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Bean instantiation via factory method failed", ex);
    }
}

```



参考：

https://blog.csdn.net/lyc_liyanchao/article/details/83098579





###### applyMergedBeanDefinitionPostProcessors

BeanDefinition 有一个属性叫 ```postProcessed```，标识该BeanDefinition是否被 MergedBeanDefinitionPostProcessor处理过， 如果没有被处理过，则调用 applyMergedBeanDefinitionPostProcessors 方法，然后设置 postProcessed=true ，所以这个方法只会处理一次。



```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
	for (BeanPostProcessor bp : getBeanPostProcessors()) {
		if (bp instanceof MergedBeanDefinitionPostProcessor) {
			MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
			bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
		}
	}
}
```



###### populateBean



```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
	if (bw == null) {
		if (mbd.hasPropertyValues()) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
		}
		else {
			// Skip property population phase for null instance.
			return;
		}
	}

	// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
	// state of the bean before properties are set. This can be used, for example,
	// to support styles of field injection.
	if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof InstantiationAwareBeanPostProcessor) {
				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        // 执行所有的InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation方法
				if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
					return;
				}
			}
		}
	}

	PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

	int resolvedAutowireMode = mbd.getResolvedAutowireMode();
	if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
		MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
		// Add property values based on autowire by name if applicable.
    // ① 属性注入方式1：按照名称注入属性bean
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
			autowireByName(beanName, mbd, bw, newPvs);
		}
		// Add property values based on autowire by type if applicable.
    // ② 属性注入方式2：按照类型注入属性bean
		if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			autowireByType(beanName, mbd, bw, newPvs);
		}
		pvs = newPvs;
	}

	boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
	boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

	PropertyDescriptor[] filteredPds = null;
	if (hasInstAwareBpps) {
		if (pvs == null) {
			pvs = mbd.getPropertyValues();
		}
    
    // 执行所有的 InstantiationAwareBeanPostProcessor 的 postProcessProperties 方法
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof InstantiationAwareBeanPostProcessor) {
				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
				PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
				if (pvsToUse == null) {
					if (filteredPds == null) {
						filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
					}
					pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						return;
					}
				}
				pvs = pvsToUse;
			}
		}
	}
	if (needsDepCheck) {
		if (filteredPds == null) {
			filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
		}
		checkDependencies(beanName, mbd, filteredPds, pvs);
	}

	if (pvs != null) {
    // 🐼 实际执行属性赋值的操作在这里👇
		applyPropertyValues(beanName, mbd, bw, pvs);
	}
}
```



> autowireByName



```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected void autowireByName(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

	String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
	for (String propertyName : propertyNames) {
		if (containsBean(propertyName)) {
			Object bean = getBean(propertyName);
			pvs.add(propertyName, bean);
			registerDependentBean(propertyName, beanName);
			if (logger.isTraceEnabled()) {
				logger.trace("Added autowiring by name from bean name '" + beanName +
						"' via property '" + propertyName + "' to bean named '" + propertyName + "'");
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("Not autowiring property '" + propertyName + "' of bean '" + beanName +
						"' by name: no matching bean found");
			}
		}
	}
}

protected String[] unsatisfiedNonSimpleProperties(AbstractBeanDefinition mbd, BeanWrapper bw) {
	Set<String> result = new TreeSet<>();
	PropertyValues pvs = mbd.getPropertyValues();
	PropertyDescriptor[] pds = bw.getPropertyDescriptors();
	for (PropertyDescriptor pd : pds) {
		if (pd.getWriteMethod() != null && !isExcludedFromDependencyCheck(pd) && !pvs.contains(pd.getName()) &&
				!BeanUtils.isSimpleProperty(pd.getPropertyType())) {
			result.add(pd.getName());
		}
	}
	return StringUtils.toStringArray(result);
}


/**
 * PropertyDescriptor
 */
public synchronized Method getWriteMethod() {
    Method writeMethod = this.writeMethodRef.get();
    if (writeMethod == null) {
        Class<?> cls = getClass0();
        if (cls == null || (writeMethodName == null && !this.writeMethodRef.isSet())) {
            // The write method was explicitly set to null.
            return null;
        }

        // We need the type to fetch the correct method.
        Class<?> type = getPropertyType0();
        if (type == null) {
            try {
                // Can't use getPropertyType since it will lead to recursive loop.
                type = findPropertyType(getReadMethod(), null);
                setPropertyType(type);
            } catch (IntrospectionException ex) {
                // Without the correct property type we can't be guaranteed
                // to find the correct method.
                return null;
            }
        }

        if (writeMethodName == null) {
            writeMethodName = Introspector.SET_PREFIX + getBaseName();
        }

        Class<?>[] args = (type == null) ? null : new Class<?>[] { type };
        writeMethod = Introspector.findMethod(cls, writeMethodName, 1, args);
        if (writeMethod != null) {
            if (!writeMethod.getReturnType().equals(void.class)) {
                writeMethod = null;
            }
        }
        try {
            setWriteMethod(writeMethod);
        } catch (IntrospectionException ex) {
            // fall through
        }
    }
    return writeMethod;
}
```





###### initializeBean





###### registerDisposableBeanIfNecessary







#### 12. finishRefresh







#### 13. resetCommonCaches













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



