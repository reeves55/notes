# Springboot手册







## Springboot启动类注解

Spring框架当中bean就是血肉，框架本身是骨架。有了bean，才叫一个应用，所以spring框架在启动时需要给框架提供 bean definitions，spring给了多种方式来解析我们定义的bean definitions，其中一种方式就是通过注解。在spring boot中，@SpringBootApplication注解就能让spring框架扫描到当前以及所有当前目录下子目录中的bean definition。其中的原因就是这个启动类会被spring当做一个配置bean添加到bean factory中，spring在refresh过程中，有一个特殊的 BeanDefinitionRegistryPostProcessor会扫描 bean factory 中的所有bean，检测出配置类，也就是启动类，然后解析启动类的注解，完成bean的扫描，并把所有扫描到的bean definition添加到bean factory。



```java
@SpringBootApplication
@EnableEurekaClient
public class Service1Application {
    public static void main(String[] args) {
        SpringApplication.run(Service1Application.class);
    }
}
```



这个特殊的BeanDefinitionRegistryPostProcessor 就是 ConfigurationClassPostProcessor ，从名字就能看出它的作用是对配置类进行处理，由于 ConfigurationClassPostProcessor 是 BeanDefinitionRegistryPostProcessor，所有它会在BeanFactoryPostProcessor之前被调用，在这里完成bean扫描，并添加到BeanFactory再好不过。

那么这个特殊的processor是什么时候被添加到bean factory中去的呢❓

答案是 AnnotationConfigServletWebServerApplicationContext 这个 ApplicationContext 在构造函数中实例化了两个重要的成员变量，一个是 reader，一个是scanner，其中 reader 在new的过程中，把 ConfigurationClassPostProcessor 添加到了bean factory中。



```java
// ① AnnotationConfigServletWebServerApplicationContext
public AnnotationConfigServletWebServerApplicationContext() {
	this.reader = new AnnotatedBeanDefinitionReader(this);
	this.scanner = new ClassPathBeanDefinitionScanner(this);
}

// ② AnnotatedBeanDefinitionReader
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
	Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
	Assert.notNull(environment, "Environment must not be null");
	this.registry = registry;
	this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
  // 注册 注解配置处理器 到bean factory
	AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}

// ③ AnnotationConfigUtils
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
		BeanDefinitionRegistry registry, @Nullable Object source) {

	DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
	if (beanFactory != null) {
		if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
			beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
		}
		if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
			beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
		}
	}

	Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(4);

  // 这里添加了 ConfigurationClassPostProcessor
	if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	if (!registry.containsBeanDefinition(REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
	if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
	if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition();
		try {
			def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
					AnnotationConfigUtils.class.getClassLoader()));
		}
		catch (ClassNotFoundException ex) {
			throw new IllegalStateException(
					"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
		}
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
	}
	if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
	}

	return beanDefs;
}
```



### ConfigurationClassPostProcessor

两步走，第一、找到配置beandefinition；第二、解析配置信息；

首先，如何判断一个beandefinition含有配置信息？ConfigurationClassPostProcessor 把含有配置信息的beandefinition分为两种，一种是 fullConfigurationCandidate（含有 ```@Configuration``` 注解），一种是 liteConfigurationCandidate（含有 ```@Component```、```@ComponentScan```、```@Import```、```@ImportResource``` 等注解，或者含有```@Bean注解的方法``` ）。

⚠️ 注意：这里面判断类是否含有这些注解，都是递归查找的，比如一个类包含@SpringBootApplication注解，没有直接包含@Configuration注解，但是@SpringBootApplication注解中包含了@Configuration注解，这也算的。



再者，如何解析一个含有配置信息的beandefinition，就要用到 ConfigurationClassParser 这个解析器，既然是配置信息是通过注解来表示的，那解析配置类肯定就意味着解析配置类中使用的配置相关的注解，ConfigurationClassParser 解析的注解有：

* ```@PropertySource```
* ```@ComponentScan```
* ```@Import```
* ```@ImportResource```
* ```@Bean methods```

每种注解包含的配置信息是不同维度的，所以不同注解需要分别处理，



#### @PropertySource处理

取出注解中的 value 属性值，是一个String[]，每一项是一个配置文件地址，针对每一个文件地址，创建一个Resource对象，添加到 ApplicationContext 的 environment 属性（AbstractApplicationContext中定义的）中。

为什么在ConfigurationClassPostProcessor中能访问到ApplicationContext的environment属性呢，这是因为ConfigurationClassPostProcessor实现了EnvironmentAware 接口，在创建ConfigurationClassPostProcessor的bean实例时，会检测这个接口，并注入 ApplicationContext 的environment对象。



#### @ComponentScan处理

拿到所有的@ComponentScan定义，然后使用 ComponentScanAnnotationParser 去解析，获取 @ComponentScan的String[] basePackages属性的值，然后解析 classpath*:base_package/**/*.class ，也就是从classpath中获取到 base package中的所有.class文件，然后判断该class类是否符合条件（@ComponentScan可以设置过滤条件），如果符合过滤条件，则构建相应的BeanDefinition对象，然后注册到 BeanFactory中去。



#### @Import处理

这个注解的作用和 xml中的<include/>标签很像，就是导入新的配置类，可以导入的配置类可以有三种类型，处理时会先生成@Import指定的配置类对象，类似getConstroctor().newInstance()这种类型的，

1. ```ImportSelector.class```：这是一个import选择器，顾名思义，也就是由这个类来决定import哪些配置类，具体处理时会调用ImportSelector对象的 selectImports 方法，返回实际上要加载的配置类名称，然后对这些实际要加载的配置类进行@Import操作，有点递归的意思。
2. ```ImportBeanDefinitionRegistrar.class```：把 ImportBeanDefinitionRegistrar 对象放到配置类的 Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> importBeanDefinitionRegistrars 属性中去，每个配置类解析完成之后，都会把调用 importBeanDefinitionRegistrars 中的 registerBeanDefinitions 方法，向 BeanFacotry 注册 BeanDefinition。
3. 其他：就当做 @Configuration 注解的类来处理，流程和处理起始配置类相同。



#### @ImportResource处理

这个注解的作用是导入 xml 配置文件，注解属性 String[] locations，可以设置多个xml配置文件路径，处理这种注解的时候，实际上就是根据 locations 生成多个Resource对象，然后把这些对象添加到当前配置类的 Map<String, Class<? extends BeanDefinitionReader>> importedResources 中去，配置类解析完成之后，都会解析 importedResources，加载其中配置的BeanDefinition到BeanFactory中去。



####@Bean Method处理

这个注解就是在配置类中的方法上面加@Bean注解，把方法返回的类封装成Bean注册到BeanFactory，ConfigurationClassPostProcessor在处理的时候，会把所有@Bean注解的方法添加到配置类的 Set<BeanMethod> beanMethods 属性中去，然后配置类解析完毕的时候，会从 beanMethods 中根据bean method构建出BeanDefinition添加到BeanFactory中去。