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

**首先**，如何判断一个beandefinition含有配置信息？ConfigurationClassPostProcessor 把含有配置信息的beandefinition分为两种，一种是 fullConfigurationCandidate（含有 ```@Configuration``` 注解），一种是 liteConfigurationCandidate（含有 ```@Component```、```@ComponentScan```、```@Import```、```@ImportResource``` 等注解，或者含有```@Bean注解的方法``` ）。

⚠️ 注意：这里面判断类是否含有这些注解，都是递归查找的，比如一个类包含@SpringBootApplication注解，没有直接包含@Configuration注解，但是@SpringBootApplication注解中包含了@Configuration注解，这也算的。



**再者**，如何解析一个含有配置信息的beandefinition，就要用到 ConfigurationClassParser 这个解析器，既然是配置信息是通过注解来表示的，那解析配置类肯定就意味着解析配置类中使用的配置相关的注解，ConfigurationClassParser 解析的注解有：

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

1. ```ImportSelector.class```：这是一个import选择器，顾名思义，也就是由这个类来决定import哪些配置类，具体处理时会调用ImportSelector对象的 selectImports 方法，返回实际上要加载的配置类名称，然后对这些实际要加载的配置类进行@Import处理，有点递归的意思，如果@Import导入的配置类是一个DeferredImportSelector.class类型，那就不马上处理，等到这一轮所有扫描到的配置类都解析完成之后再处理，处理时，调用DeferredImportSelector.getImportGroup()方法，拿到一个Group对象，然后 调用 Group.process(...)方法之后，再调用Group.selectImports()方法返回要导入的配置类，最后对每个需要导入的配置类进行@Import操作。
2. ```ImportBeanDefinitionRegistrar.class```：把 ImportBeanDefinitionRegistrar 对象放到配置类的 Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> importBeanDefinitionRegistrars 属性中去，每个配置类解析完成之后，都会把调用 importBeanDefinitionRegistrars 中的 registerBeanDefinitions 方法，向 BeanFacotry 注册 BeanDefinition。
3. 其他：就当做 @Configuration 注解的类来处理，流程和处理起始配置类相同。



#####@EnableAutoConfiguration

拿常用的@EnableAutoConfiguration注解来说，它就有一个@Import注解

```java
@Import(AutoConfigurationImportSelector.class)
```

AutoConfigurationImportSelector类是一个DeferredImportSelector，因此这个@Import是在一般的@Import处理之后。AutoConfigurationImportSelector的getImportGroup()方法返回了一个内部类AutoConfigurationGroup，这样处理就有点曲线救国的感觉了，内部的AutoConfigurationGroup.process(...)方法实际上调用了外部类的selectImports()方法，并把返回值保存起来，然后调用AutoConfigurationGroup.selectImports()方法，这就等于是调用外部的AutoConfigurationImportSelector的selectImports()方法。

AutoConfigurationImportSelector的selectImports()方法做了一件非常重要的事情，如下：

```java
/**
 * AutoConfigurationImportSelector.class
 */
public String[] selectImports(AnnotationMetadata annotationMetadata) {
	if (!isEnabled(annotationMetadata)) {
		return NO_IMPORTS;
	}
  // ① 这一步实际上是调用 loadMetadata(classLoader, PATH)，其中的PATH就是
  // META-INF/spring-autoconfigure-metadata.properties，这个文件是spring自己用的文件
  // 包含了一些预定义的 autoconfigure 类
	AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
			.loadMetadata(this.beanClassLoader);
	AnnotationAttributes attributes = getAttributes(annotationMetadata);
  
  // ② 从classpath.*:/META-INFO/spring.factories文件中获取 EnableAutoConfiguration 配置类
	List<String> configurations = getCandidateConfigurations(annotationMetadata,
			attributes);
	configurations = removeDuplicates(configurations);
	Set<String> exclusions = getExclusions(annotationMetadata, attributes);
	checkExcludedClasses(configurations, exclusions);
	configurations.removeAll(exclusions);
  
  // ③ 对结果进行过滤，主要针对configurations每一项，在autoConfigurationMetadata中，如果存在 
  // xxxConig.ConditionalOnClass 这样的配置，就去检查这个配置指定的class是否存在，如果不存在
  // 就把configurations这一项过滤掉
	configurations = filter(configurations, autoConfigurationMetadata);
	fireAutoConfigurationImportEvents(configurations, exclusions);
  
  // ④ 把需要解析的配置类返回，等待解析
	return StringUtils.toStringArray(configurations);
}

protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
		AnnotationAttributes attributes) {
  // 只从spring.factories文件中加载 EnableAutoConfiguration 的配置
  // getSpringFactoriesLoaderFactoryClass(): EnableAutoConfiguration.class
	List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
			getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
	Assert.notEmpty(configurations,
			"No auto configuration classes found in META-INF/spring.factories. If you "
					+ "are using a custom packaging, make sure that file is correct.");
	return configurations;
}
```



#### @ImportResource处理

这个注解的作用是导入 xml 配置文件，注解属性 String[] locations，可以设置多个xml配置文件路径，处理这种注解的时候，实际上就是根据 locations 生成多个Resource对象，然后把这些对象添加到当前配置类的 Map<String, Class<? extends BeanDefinitionReader>> importedResources 中去，配置类解析完成之后，都会解析 importedResources，加载其中配置的BeanDefinition到BeanFactory中去。



####@Bean Method处理

这个注解就是在配置类中的方法上面加@Bean注解，把方法返回的类封装成Bean注册到BeanFactory，ConfigurationClassPostProcessor在处理的时候，会把所有@Bean注解的方法添加到配置类的 Set<BeanMethod> beanMethods 属性中去，然后配置类解析完毕的时候，会从 beanMethods 中根据bean method构建出BeanDefinition添加到BeanFactory中去。



## @EnableAutoConfiguration

这个注解是spring boot自动配置的 “导火索”，或者是 “启动按钮”，和 ```spring.factories``` 合在一起，完成 auto configuration功能。也就是说，如果你依赖了 spring starter 这种类型的依赖，如果想让引入的依赖完成自动配置，则需要在启动配置类或者其他能扫描到的配置类上面加 @EnableAutoConfiguration，











对注解配置类进行解析时，配置类可能含有的注解有：@Component、@ComponentScan、@Import、@ImportResource，

**@ComponentScan注解处理**：在扫描@ComponentScan指定的base packages当中的类时候，会对每个类进行检测，判断这个类是否可以被解析成bean definition，交由spring来管理，判断的条件就是 这个类被 ```@Component``` 注解或者 ```@ManagedBean``` 注解

**@Import注解处理**：@Import可以导入三种导入类类型，





注解配置类解析是由 ConfigurationClassParser 来解析的，



1. 从初始的 bean definition map 中找到配置类candidates，如果配置类有直接声明了 @Configuration 注解或者注解的元注解中包含 @Configuration 的，则就是配置类

```java
// 下面这些，@Target、@Retention等等 都是元注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
// 这是注解定义
public @interface EnableAutoConfiguration {
  //....
}
```



2. 如果没有直接声明的 @Configuration 注解，那就判断有没有候选的注解，包括 @Component、@ComponentScan、@Import、@ImportResource，如果有直接声明这几种注解或者元注解中包含这些注解，那么这个类就可以被作为候选配置类









## SpringFramework拓展点

[官网](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-factory-extension) 给出说明，SpringFramework的拓展点有三个：

* BeanFactoryPostProcessor（自定义beans的配置）

* BeanPostProcessor（自定义Bean定义）
* FactoryBean（自定义bean实例化逻辑）







## SpringBoot整合第三方框架

第三方框架整合进SpringBoot，其实主要就是把自己的一些spring拓展点处理器添加到spring容器当中，然后随着spring框架的生命周期，在某个时间点被调用，完成相应的逻辑。

第一个问题，如何将第三方框架定义的一些处理bean添加到spring bean factory中❓

有几种方式：

1. 当框架本身不需要用户自定义配置，框架自动配置完之后就可以发挥作用，那么可以采用 spring boot starter 的方式，具体来说就是在框架下添加 spring.factories 文件，并在文件中配置好 org.springframework.boot.autoconfigure.EnableAutoConfiguration 即可，其中原理是配置类上加了@EnableAutoConfiguration注解（org.springframework.boot.autoconfigure.EnableAutoConfiguration），那么这个类使用的 AutoConfigurationImportSelector 会自动从 spring.factories 文件中找到所有的 EnableAutoConfiguration配置值，然后把配置值作为配置类去解析，这样就可以配置好第三方框架，零配置，方便简洁
2. 当框架本身需要用户进行一些自定义配置，那么可以采用自定义注解，在注解当中支持用户配置一些信息，自定义注解中通过@Import引入自定义的配置类，在配置类当中解析自定义注解中用户配置信息，完成第三方框架的初始化，可以支持比较复杂的框架初始化逻辑
3. 当框架按照第一种方式实现了，但是需要给用户一个“开关”，让用户开启框架，可以自定义一个 @EnableXXX 的注解，在注解里面通过 @Import 引入一个开关Configuration配置类，这个配置类定义一个开关 @Bean，然后在框架主题的 AutoConfiguration 配置类上面加一个 @ConditionalOnBean("开关Bean")，这样就实现了用 @EnableXXX来控制原本自动开启的第三方框架是否开启



### tk.mybatis

tk.mybatis采用的是用户可配置的注解（```@MapperScan```）方式，将tk.mybatis整合进spring framework，这个注解会@Import方式引入类 tk.mybatis.spring.annotation.MapperScannerRegistrar.class，这个类实现了 ImportBeanDefinitionRegistrar 接口，因此会调用其 ```registerBeanDefinitions``` 方法，在这个方法里面，tk.mybatis解析用户配置，然后构造出一个scanner，由scanner去扫描用户配置的basePackages下面的所有bean definition，然后对bean definition做处理，包括把bean class更换为 tk.mybatis的factory bean（```MapperFactoryBean```），这样这个bean在实例化的时候，就可以按照tk.mybatis的逻辑，由MapperFactoryBean返回整合了tk.mybatis功能的bean实例。



其中，上述中的scanner继承了ClassPathBeanDefinitionScanner，这是一个可以自定义扫描规则的 bean definition scanner，只要是符合你定义规则的.class文件都会被构建成BeanDefinition返回给你，不管这个类/接口有没有标注@Component之类的。



