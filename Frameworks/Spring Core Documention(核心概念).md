# Spring Core Documention(核心概念)



## 0. 整体介绍

### 核心

spring它是一个bean容器，它包含着很多bean definition，并根据这些bean definition实例化出bean，这就是一个spring应用大厦的图纸。我们如何构建出这个大厦的图纸呢，在spring中，就是根据配置信息，生成bean definition，并注册到容器中，所以解析配置，获取bean definition就是第一个核心。

![springframework](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/springframework.png)

### 主要流程



#### ApplicationContext.refresh 模板方法

每一种ApplicationContext，都会在完成一些自定义设置之后，进入到 AbstractApplicationContext 的 refresh() 方法执行流程中，这是一个模板方法，定义了游戏流程，每一种特定的ApplicationContext只能在游戏规则里，设计本身的实例化机制。ClassPathXmlApplicationContext 就在 obtainFreshBeanFactory 阶段实例化了 bean factory对象，并加载了所有的 bean definition，能这样做是因为 ApplicationContext 的 obtainFreshBeanFactory 执行时，留了一个抽象方法 refreshBeanFactory 给子类实现

![ApplicationContext.refresh模板方法](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/ApplicationContext.refresh模板方法.png)





#### AnnotationConfigApplicationContext



![SpringFramework启动过程1](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/SpringFramework启动过程1.png)



#### 实例化  Bean













### 主要角色





基于注解的ApplicationContext，在解析配置，**获取bean definition阶段**，主要涉及到的角色有：

1. ```@注解```：解析注解就像是按图索骥，解析的目的是为了得到哪里有bean的定义，基于注解的ApplicationContext中支持多种注解，用来指定从哪里获取bean definition；
2. ```BeanDefinition```：根据BeanDefinition，经历了一系列的步骤，最终可以构建出实际上需要的bean实例，```①``` 创建bean对象 -> ```②``` 给bean对象执行属性注入
3. ```BeanPostProcessor```：
4. ```Environment```：它包含了Application运行时的外部环境变量，主要包括两种信息，一种是当前激活的profile名称，一种是配置properties，包括.properties文件中定义的环境变量、JVM环境变量、系统环境变量等等；
5. ```ApplicationListener```：注册到 ApplicationContext 上的事件监听器，监听 ApplicationContext 发布的事件，包括：
6. ```ApplicationEventMulticaster```：ApplicationContext 中的事件发布器，负责发布事件，调用 ApplicationListener 的事件监听回调方法，执行listener回调方法的时候，如果有线程池，优先使用线程池执行回调任务，如果没有线程池，则在主线程中完成回调
7. ```ApplicationContext```：
8. ```DefaultListableBeanFactory```：bean factory，里面包含了所有的bean definition，把bean definiton注册到容器，实际上就是放到这里，bean factory在spring框架启动过程经过了几个主要的周期，包括：```①``` 新建并初始化 -> ```② ```prepare -> ```③ ```执行自定义后置处理 -> ```④ ```执行所有的 BeanFactoryPostProcessor 后置处理器逻辑 ->``` ⑤``` 实例化 bean factory 管理的所有bean实例
9. ```BeanFactoryPostProcessor```：
10. ```InstantiationAwareBeanPostProcessor```：
11. ```BeanDefinitionRegistryPostProcessor```：是 BeanFactoryPostProcessor 的一种，但是在执行流程中，它会比所有的BeanFactoryPostProcessor早执行，并且首先执行







Bean的实例化过程









基于注解的ApplicationContext，在根据 bean definition **实例化bean阶段**，主要涉及到的角色有：

1. 











#### Environment

主要属性：







#### ApplicationListener

主要接口方法：

* ```void onApplicationEvent(E event)```：E extends ApplicationEvent





#### ApplicationEventMulticaster

ApplicationContext的事件发布器，负责发布特定事件给匹配的 ApplicationListener，并执行listener的onApplicationEvent方法

主要接口方法：

* ```void addApplicationListener(ApplicationListener<?> listener)```
* ```void addApplicationListenerBean(String listenerBeanName)```
* ```void removeApplicationListener(ApplicationListener<?> listener)```
* ```void removeApplicationListenerBean(String listenerBeanName)```
* ```void removeAllListeners()```
* ```void multicastEvent(ApplicationEvent event)```
* ```void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType)```





#### SmartInitializingSingleton

主要接口方法：

* ```void afterSingletonsInstantiated()```：单例bean在实例化之后，执行的回调方法，注意，只有bean的scope是singleton才行，而且 lazyinit不能为true，因为这个回调只在spring启动的时候执行







#### AbstractBeanDefinition

主要属性：

* ```stale```
* ```beanClass```
* ```propertyValues```
* ```role```
* ```source```
* ```resource```
* ```scope```：bean definition定义中scope有两种取值，singleton / prototype，在实例化bean阶段，如果检测到 bean definition此属性的值为 singleton 的话，实例化bean之后，放入bean factory中的 singletonObjects 中，如果是 prototype 的话，则不会执行放入 singletonObjects 的操作，而是直接实例化出bean实例
* ```abstractFlag```
* ```isFactoryBean```
* ```factoryBeanName```
* ```isFactoryMethodUnique```：
* ```factoryMethodToIntrospect```：缺省的工厂方法，这个和 resolvedConstructorOrFactoryMethod 属性实际上是同步的，都是可以直接调用的方法对象
* ```factoryMethodName```：创建bean对象的时候，如果此属性不为空，则选择实例化bean方式为 调用工厂方法。在检测一个factory class中究竟哪个方法是工厂方法时，会比较factory class中的每个方法名和此属性，方法名相同才能算作 candidate工厂方法，⚠️ 注意：一旦一个bean设置成使用工厂方法生成bean对象，那么bean的class属性就没有意义了，工厂方法返回什么类型，这个bean对象就是什么类型
* ```constructorArgumentLock```：锁
* ```resolvedConstructorOrFactoryMethod```：Method类型，如果bean对象是使用工厂方法来创建的，第一次创建时，通过查找找到了符合要求的工厂方法，这个属性将会存储这个找到的工厂方法，下次再实例化bean对象时候（prototype bean），就不需要再去找工厂方法了，singleton 类型的bean再次获取时，就直接从 bean factory singletonObjects 里面拿了，和工厂方法没关系
* ```constructorArgumentsResolved```：boolean类型，标识工厂方法参数是否已经解析完成过
* ```constructorArgumentValues```：在 bean定义的时候指定了bean factory method方法参数的值，在xml配置文件中，就是配置bean的 <constructor-arg /> 标签值，此属性保存了这些值，但是这里的方法参数可能还需要解析，如果通过 contructor-arg 标签设置 ref="xxxBean"，这个参数的值不是对应的bean实例，而只是包含了bean name的一些信息，所以这个参数在实际使用之前，需要经过解析
* ```resolvedConstructorArguments```：已经解析完成的工厂方法参数，可以直接传给工厂方法作为参数，让工厂返回相应的bean实例
* ```preparedConstructorArguments```：还没有被解析的工厂方法参数，比如一个工厂方法参数是一个bean，那这个属性就只存储了bean的name之类的信息，没有存储bean实例，这就叫还没解析
* ```attributes```：类型 Map<String, Object> 
* ```methodOverrides```
* ```lazyInit```：bean是否需要lazy init，默认为false，如果为true，则不会在框架启动时就实例化出相应的bean，等到需要该bean的时候才会实例化出bean
* ```autowireMode```：自动注入模式，当一个bean使用工厂方法创建bean对象时，如果工厂方法需要传入参数，那就要看这个bean的autowireMode的值了，只有当bean是通过constructor方式自动注入时，工厂方法创建bean对象时，对于工厂方法参数，没有设置方法参数值时，会自动传入一个参数值，不会报错，但是如果autowireMode是其他值的话，则工厂方法需要什么参数，bean definition就要配置每个参数的值，如果没有配置，在使用工厂方法创建bean对象的时候会报错
* ```dependencyCheck```
* ```dependsOn```：和 xml 配置中的 depends-on 效果一样，当前bean必须在 dependsOn指定的bean实例化之后再实例化，在bean factory实例化bean阶段，如果检测到了某个 bean definition的这个属性不为空，则把bean之间的依赖关系保存到 bean factory 的 dependentBeanMap 和 dependenciesForBeanMap 属性中，然后实例化每一个 depend bean
* ```autowireCandidate```
* ```primary```
* ```qualifiers```
* ```instanceSupplier```
* ```nonPublicAccessAllowed```
* ```lenientConstructorResolution```
* ```initMethodName```
* ```enforceInitMethod```
* ```destroyMethodName```
* ```enforceDestroyMethod```
* ```synthetic```



##### AnnotatedGenericBeanDefinition

* ```beanClass```
* ```metadata```
* ```instanceSupplier```
* ```scope```
* ```primary```
* ```lazyInit```
* ```qualifiers```
* 



##### RootBeanDefinition

* 继承自AbstractBeanDefinition属性
* ```decoratedDefinition```
* ```qualifiedElement```
* ```allowCaching```
* ```isFactoryMethodUnique```
* ```targetType```
* ```factoryMethodToIntrospect```



#### FactoryBean

主要接口方法：

* ```T getObject() throws Exception```：获取FactoryBean生成的bean实例
* ```Class<?> getObjectType()```：返回FactoryBean对应生成的Bean实例类型，每种FactoryBean返回值都不相同，没有统一的属性值存储这个 object type
* ```default boolean isSingleton() { return true;}```：FactoryBean生成的bean实例是否是单例bean，默认为true





#### BeanDefinitionRegistry

主要接口方法：

* ```void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) ```
* ```void removeBeanDefinition(String beanName)```
* ```BeanDefinition getBeanDefinition(String beanName)```
* ```boolean containsBeanDefinition(String beanName)```
* ```String[] getBeanDefinitionNames()```
* ```int getBeanDefinitionCount()```
* ```boolean isBeanNameInUse(String beanName)```：bean factory中是否已经注册或者用到这个bean name指定的bean





#### DefaultListableBeanFactory

主要属性：

* ```serializationId```
* ```parentBeanFactory```
* ```beanDefinitionMap```
* ```beanDefinitionNames```
* ```mergedBeanDefinitions```：类型为 Map<String, RootBeanDefinition>，bean name -> RootBeanDefinition
* ```configurationFrozen```：当 bean factory实例化所有单例bean之前，会把这个值设置为true，意味着 bean factory管理的bean definition不会再被修改，也不会再被 bean post processor处理，这个时候，对bean definition相关数据进行缓存才是安全的，比如这个时候就可以缓存符合某种类型的所有bean名称，这样就不需要遍历bean definition map了，但是如果这个时候 bean definition可以被修改，那缓存结果可能和实际对应不上，就没有缓存的意义
* ```frozenBeanDefinitionNames```：类型String[]，当 bean factory实例化所有单例bean之前，会把这个值设置成当时 beanDefinitionNames 的 String[] 格式的值
* ```beanClassLoader```
* ```beanExpressionResolver```
* ```propertyEditorRegistrars```
* ```beanFactoryPostProcessors```
* ```beanPostProcessors```
* ```hasInstantiationAwareBeanPostProcessors```
* ```hasDestructionAwareBeanPostProcessors```
* ```ignoredDependencyInterfaces```
* ```resolvableDependencies```
* ```tempClassLoader```
* ```singletonObjects```：存放所有单例bean实例，包括 singleton factory bean，
* ```factoryBeanInstanceCache```：factory bean实例缓存，
* ```factoryBeanObjectCache```：如果 bean factory 是单例bean并且已经实例化过了，在获取，如果实例化之后的bean是一个 factory bean，那首先会在这个属性中寻找到factory bean对应的bean实例，如果没有找到，则
* ```manualSingletonNames```
* ```singletonFactories```
* ```earlySingletonObjects```
* ```prototypesCurrentlyInCreation```：scope为prototype的bean在实例化之前，会把将要实例化bean的bean name 放到这个属性中去，bean实例化之后，不管是否实例化成功，都会把 bean name 从这个属性中移除
* ```registeredSingletons```
* ```allBeanNamesByType```
* ```singletonBeanNamesByType```
* ```dependencyComparator```
* ```autowireCandidateResolver```
* embeddedValueResolvers
* ```dependentBeanMap```：
* ```dependenciesForBeanMap```：



#### AnnotationConfigApplicationContext

主要属性：

* ```reader```
* ```scanner```

* ```beanFactory```
* ```environment```
* ```startupShutdownMonitor```
* ```startupDate```
* ```closed```
* ```active```
* ```applicationListeners```
* ```earlyApplicationListeners```
* 



#### AnnotatedBeanDefinitionReader

主要属性：

* ```registry```：ApplicationContext实例
* ```conditionEvaluator```
* ```scopeMetadataResolver```
* ```beanNameGenerator```
* 



#### ClassPathBeanDefinitionScanner

主要属性：

* ```registry```：ApplicationContext实例
* ```includeFilters```
* ```environment```
* ```conditionEvaluator```
* ```resourcePatternResolver```
* ```metadataReaderFactory```
* ```componentsIndex```
* 











## 1. The IoC Container

### 1.1. Introduction to the Spring IoC Container and Beans



### 1.2. Container OverviewOverview



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200403141745413.png" alt="image-20200403141745413" style="zoom:80%;float:left" />





#### 1.2.1. Configuration Metadata

Configuration Metadata指的就是 bean 的定义，配置信息包括了 bean 和用户定义的 POJO 类之间的联系，通过解析这些 bean 定义，向Spring容器中添加 bean实例，有三种 Configuration Metadata（bean配置信息），包括：

* XML-based 配置
* Annotation-based 配置   （```Spring2.5引入```）
* Java-based 配置              （```Spring3.0引入```）



#####①. XML-based

基于XML配置的方式，所有的配置都写在XML文件中，可以有多个XML文件，XML文件当中可以定义的东西有很多，因为Spring它引入了可拓展性的XML，可以通过以下这种方法添加多个namespace，然后可以用 <namespace:tag />的方式使用外部引入的标签，举个🌰：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">

<!-- 这里除了可以定义原始的<bean>相关的标签外，还可以使用util标签 -->
<!-- <bean/> definitions here -->

</beans>
```



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200403152416929.png" alt="image-20200403152416929" style="zoom:80%;" />



我们只讨论基本的 ```<bean>``` 标签，给一个完整的bean定义的例子，bean的定义可以概况如下：

 **属性**：

* ```id``` ：唯一标识一个bean
* ```name```：bean的名字，也可以标识一个bean，值可以是逗号、分号、空格间隔的多个字符串，相当于别名
* ```class```：bean实例化时对应的POJO对象，对应BeanDefinition的 <span style="color:red">beanClass</span> 属性
* ```parent```：bean的继承，子bean可以继承父bean的相关属性，在解析带有parent属性的bean标签时，生成相应GenericBeanDefinition对象会设置其 parentName 属性的值，这个值在调用 getMergedLocalBeanDefinition 方法 的时候会判断 parentName 属性的值来决定是否需要merge BeanDefinition，对应BeanDefinition的 <span style="color:red">parentName</span> 属性
* ```scope```：取值范围为 [singleton | prototype]，对应BeanDefinition的 <span style="color:red">scope</span> 属性
* ```abstract```：在BeanFactory实例化bean的阶段，会调用 preInstantiateSingletons 方法，abstract类型的bean是不会实例化的，abstract 对应BeanDefinition的 <span style="color:red">abstractFlag</span> 属性
* ```lazy-init```：如果lazy-init为true，则不会在 preInstantiateSingletons 调用时实例化这些bean，只在需要这个bean（getBean("xxx")）的时候才会实例化这个bean，对应BeanDefinition的 <span style="color:red">lazyInit</span> 属性
* ```autowire```：bean的属性值的自动注入选项，默认为no，即不自动注入bean的属性，这样bean的属性值就不会被自动设置，可选值有4个：no、byName、byType、constructor，分别对应 BeanDefinition <span style="color:red">autowireMode</span> 属性 的4个可能值：AUTOWIRE_NO（0）、AUTOWIRE_BY_NAME（1）、AUTOWIRE_BY_TYPE（2）、AUTOWIRE_CONSTRUCTOR（3），在populateBean阶段，会根据BeanDefinition的autowireMode属性来判断如何注入属性，如果是 AUTOWIRE_BY_NAME 或者 AUTOWIRE_BY_TYPE ，则会调用指定的自动注入方法，解析出属性需要注入的值，在基于注解的ApplicationContext中，一个类当中的某个属性，如果不加@Autowired之类的注解，默认autowire=0，这个属性是不会被自动注入的，这个时候的属性自动注入是@Autowired处理器来做的
* ```depends-on```：指定当前bean必须在depends-on设置的bean实例化之后，再实例化，也就是规定了指定的这些bean的实例化顺序，同时，bean在destroy过程中，必须要先销毁当前bean，才会销毁depends-on指定的从属bean。如果当前bean执行正常的功能，需要其他bean先做某些操作，但是这两个bean不存在直接的依赖关系，就可以使用depends-on。对应BeanDefinition的 <span style="color:red">String[] dependsOn</span> 属性
* ```autowire-candidate```：设置当前bean是否能被spring框架自动注入到其他的bean，对应BeanDefinition的 <span style="color:red">autowireCandidate</span> 属性，默认为true，这个属性在 populateBean 阶段执行bean的属性注入时，针对bean的属性会解析其可能的注入candidate，这个时候找到的注入bean，会判断其是否是autowire candidate，判断的依据就是其相应BeanDefinition的populateBean属性是否为true
* ```primary```：当容器中有多个同一种类型的bean时，其他bean需要注入这种类型的bean就会优先使用primary设置为true的bean，而不会注入其他bean实例，但是如果有多个同种类型的bean都设置了primary为true，自动注入会失败，不知道要注入哪个实例。对应于BeanDefinition中的 <span style="color:red">boolean primary</span> 属性
* ```init-method```：
* ```destroy-method```：
* ```factory-method```：
* ```factory-bean```：

**子标签：**

* ```<meta>```
* ```<constructor-arg>```
* ```<property>```
* ```<qualifier>```
* ```<lookup-method>```
* ```<replaced-method>```



```xml
<bean id="bean1" name="bean1" class="com.xiaolee.demo.beans.Component1" parent="parent" scope="singleton" abstract="false" lazy-init="false" autowire="byName" depends-on="bean2" autowire-candidate="false" primary="true" init-method="init" destroy-method="destroy" factory-method="newInstance" factory-bean="factoryBeanId">
    <!---->
  	<meta key="key" value="value"/>
  	<!--构造方法参数值-->
    <constructor-arg name="" ref="bean2" index="1" type="java.lang.String" value="value"></constructor-arg>
  	<!--属性值-->
    <property name="prop1" value="prop1" ref="ref"></property>
  	<!---->
    <qualifier value="value" type="java.lang.String"></qualifier>
  	<!---->
    <lookup-method name="createBean2" bean="bean2"></lookup-method>
  	<!---->
    <replaced-method name="getBean1" replacer="com.xiaolee.Bean1Replacer"></replaced-method>
</bean>
```



##### ②. Annotation-based



支持的注解有：

* @Required
* @Autowired
* JSR 330注解：@Inject、





##### ③. Java-based







##### protocol-type bean实例化



```java
/**
 * AbstractBeanFactory
 */
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
  // 省略很多代码...
  
  if (mbd.isPrototype()) {
	// It's a prototype -> create a new instance.
	Object prototypeInstance = null;
	try {
    // ① 把 beanName 放到 ThreadLocal<Object> prototypesCurrentlyInCreation 中
		beforePrototypeCreation(beanName);
    // ② 创建 bean 实例
		prototypeInstance = createBean(beanName, mbd, args);
	}
	finally {
    // ③ 把 beanName 从 ThreadLocal<Object> prototypesCurrentlyInCreation 中移除
		afterPrototypeCreation(beanName);
	}
    // ④ 返回
		bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
	}
  
  // 省略很多代码...
}
```



##### autowire？

populateBean的时候，有两种autowireType是会特殊处理的，一个是 autowireByName，一个是 autowireByType，



```java
/**
 * AbstractAutowireCapableBeanFactory
 */
protected void autowireByType(
		String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

	TypeConverter converter = getCustomTypeConverter();
	if (converter == null) {
		converter = bw;
	}

	Set<String> autowiredBeanNames = new LinkedHashSet<>(4);
	String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
	for (String propertyName : propertyNames) {
		try {
			PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
			// Don't try autowiring by type for type Object: never makes sense,
			// even if it technically is a unsatisfied, non-simple property.
      // 不会对Object类型的属性进行自动注入，没有意义，根本不知道要注入什么值
			if (Object.class != pd.getPropertyType()) {
        // 获取到属性的 setter 方法
				MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
				// Do not allow eager init for type matching in case of a prioritized post-processor.
				boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);
				DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);
				Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
				if (autowiredArgument != null) {
					pvs.add(propertyName, autowiredArgument);
				}
				for (String autowiredBeanName : autowiredBeanNames) {
					registerDependentBean(autowiredBeanName, beanName);
					if (logger.isTraceEnabled()) {
						logger.trace("Autowiring by type from bean name '" + beanName + "' via property '" +
								propertyName + "' to bean named '" + autowiredBeanName + "'");
					}
				}
				autowiredBeanNames.clear();
			}
		}
		catch (BeansException ex) {
			throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, ex);
		}
	}
}
```



##### determineAutowireCandidate

从多个candidates中解析出需要自动注入哪个bean，

```java
/**
 * DefaultListableBeanFactory
 */
protected String determineAutowireCandidate(Map<String, Object> candidates, DependencyDescriptor descriptor) {
	Class<?> requiredType = descriptor.getDependencyType();
  // 从candidates中找到对应BeanDefinition的primary属性为true的bean name
	String primaryCandidate = determinePrimaryCandidate(candidates, requiredType);
	if (primaryCandidate != null) {
		return primaryCandidate;
	}
  //   
	String priorityCandidate = determineHighestPriorityCandidate(candidates, requiredType);
	if (priorityCandidate != null) {
		return priorityCandidate;
	}
	// Fallback
	for (Map.Entry<String, Object> entry : candidates.entrySet()) {
		String candidateName = entry.getKey();
		Object beanInstance = entry.getValue();
		if ((beanInstance != null && this.resolvableDependencies.containsValue(beanInstance)) ||
				matchesBeanName(candidateName, descriptor.getDependencyName())) {
			return candidateName;
		}
	}
	return null;
}
```





### 1.6 Customizing the Nature of a Bean

这里的自定义bean的性质，是通过spring framework留给开发者的一些可用接口，来实现开发者对bean的一些修改或者配置

#### 1.6.1. Lifecycle Callbacks(生命周期回调)

##### Initialization Callbacks

有好几种方式实现初始化回调：

* 实现 InitializingBean 接口，实现 afterPropertiesSet() 方法
* 使用 @PostConstruct 注解某个方法
* xml文件在<bean>标签中设置 "init-method"

这个方法是在bean所有属性都设置完成之后才会调用



##### Destruction Callbacks




