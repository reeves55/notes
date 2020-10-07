# SpringFramework - Core 梳理



## Spring框架是什么

为了代替EJB而设计的一个轻量级Java企业应用开发框架，它以jar包的形式供开发者使用，具有两大核心功能，一个是控制翻转 - IoC，一个是切面编程 - AoP，在web方面可以和servelt容器很好地集成，不需要像EJB依赖专用的EJB容器。



![Spring architectureDone](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/Spring-architectureDone.jpg)



## IoC容器



### 容器

容器里面装的最重要的就是 ```BeanDefinition```，BeanDefinition由两个要素构建：<span style="color:red">① Configuration Metadata；② Your Bussiness Objects；</span>



![container magic](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/container-magic.png)





> **Configuration Metadata**



1. **概念**：bean的元数据，也就是bean的生产图纸，spring根据它来 实例化、配置、装配bean，然后提供给应用使用；

2. 支持 **2种声明bean元数据的方式**：

   **①** <span style="color:#41B883">**以XML配置文件为核心**</span>：在xml文件当中通过<bean>标签声明bean definition，或者通过```<context:component-scan base-package="org.example"/>``` 配合 ```@Component```注解指定扫描的包路径，从路径中扫描出@Component注解的bean definition，这时候通过注解来配置bean definition的各个属性，会用到许多相关注解，包括 @Required、@Autowired、@Primary、@Qualifiers、@Resource、@Value、@PostConstructor、@PreDestroy 等等；

   **②** <span style="color:#41B883">**以注解配置类为核心**</span>：当一个类被```@Configuration```、```@Component```、```@ComponentScan```、```@Import```、```@ImportResource``` 注解修饰，或者含有 ```@Bean``` 注解的方法，它就是一个注解配置类，解析注解配置类，就能拿到bean definition（需要容器中存在 ```ConfigurationClassPostProcessor```）。





> **Configuration Metadata 解析，注册到BeanDefinition容器**



创建一个BeanDefinition容器

```java
ApplicationContext context = new XXXApplicationContext(new Resource("xxx"));
```



Configuration Metadata 解析这一步，主要是拿到所有的 bean definition信息，就是 <span style="color:red">拿到所有的 <bean> 标签，或者扫描到所有的 @Componet类</span>，根据bean definition信息生成BeanDefinition对象，并注册到容器中。



XML配置文件解析  <span style="color:#41B883">**=>**</span>  ```XmlBeanDefinitionReader```

注解配置类解析    <span style="color:#41B883">**=>**</span>   ```ConfigurationClassPostProcessor```





> **修改BeanDefinition容器 - BeanFactoryPostProcessor**



这里的post指的是，BeanFactory已经准备完毕，当然 BeanFactoryPostProcessor 也有很多种类，主要有这2种：

1. ```BeanDefinitionRegistryPostProcessor```：也是BeanFactoryPostProcessor的一种，但是在BeanFactoryProcessor之前起作用，可以在BeanFactoryPostProcessor处理之前向容器手动注册新的BeanDefinition。
2. ```BeanFactoryPostProcessor```：



<span style="color:#41B883">**手动添加新的BeanDefinition / 对已有BeanDefinition进行修改**</span>



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200507161415045.png" alt="image-20200507161415045" style="zoom:80%;" />





手动添加BeanDefinition的工具方法







### Bean



> Spring如何管理bean实例



Spring对bean的管理体现在多个方面：

1. <span style="color:#41B883">**管理bean的构建（实例化、依赖注入）**</span>：根据BeanDefinition生产bean实例
2. <span style="color:#41B883">**管理bean的生命周期**</span>：Spring定义了一系列的生命周期回调接口，在bean初始化、析构、容器启动、容器关闭时都会由容器执行这些回调方法。





> **根据 BeanDefinition 生产bean实例**



1. BeanDefinition概念：生产一个bean实例所需的所有必要信息，相当于一道菜的 “食谱”；

    

   <img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200507161415045.png" alt="image-20200507161415045" style="zoom:80%;" />



2. 根据 “菜谱” 生产一个 bean

   BeanDefinition有不同种类，在BeanDefinition里我们定义了各种各样生成bean实例的规则，这些规则都需要被解析，并在生成bean实例过程中起作用。

   

   ① new一个bean实例对象：一个是通过 class属性指定，一个是通过 bean factory method指定，通过class属性指定时通过class的构造方法创建对象，通过 bean factory method指定时通过调用工厂方法创建对象。
   
   ② 对bean实例对象进行依赖注入：有三个地方可以设置依赖注入信息，属性、构造方法参数、工厂方法参数（XML配置文件中通过 ```<property/>``` 和 ```<constructor-arg/>``` 来设置依赖注入），依赖注入有2大主要方法，
   
   1. **通过构造方法注入依赖**，带参数的工厂方法也算是通过构造方法注入依赖；Spring容器把构造方法参数都作为要注入的依赖，传入依赖，new出bean实例，构造方法注入有几个优点：① bean无需暴露setter方法，所以依赖不会在使用过程中被修改，bean是一个不可变对象，② bean实例一旦返回，就是一个已经初始化完成的bean
   2. **通过setter方法注入依赖**；Spring容器先调用无参构造方法或者无参工厂方法创建出对象，再调用属性相关的setter方法注入依赖



> **插手bean的生产过程 - ```BeanPostProcessor```**



一个bean实例的生产过程大致如下：

1.  在创建bean实例之前，从BeanPostProcessor中遍历所有的 ```InstantiationAwareBeanPostProcessor``` ，调用其 ```Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName)``` 方法，如果返回值不为null，则继续调用所有BeanPostProcessor的 ```Object postProcessAfterInitialization(Object bean, String beanName) ```方法处理这个bean实例，如果处理完成之后，bean不为null，则直接返回处理完成的 bean实例，不再进行后续步骤 🔚；
2. 找到合适的方式new出一个新的bean实例来（未执行依赖注入）；
3. 从BeanPostProcessor中遍历所有的 ```MergedBeanDefinitionPostProcessor``` ，调用其 ```void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName)``` 方法（注意 MergedBeanDefinitionPostProcessor extends BeanPostProcessor）；
4. populateBean步骤，从BeanPostProcessor中遍历所有的 ```InstantiationAwareBeanPostProcessor``` ，调用其 ```boolean postProcessAfterInstantiation(Object bean, String beanName)``` 方法，如果调用过程中有的processor返回false，则过程中断；
5. populateBean步骤，从BeanPostProcessor中遍历所有的 ```InstantiationAwareBeanPostProcessor ```，调用其 ```PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)``` 方法，如果processor调用这个方法返回值为null，则继续调用其 ```PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) ```方法；
6. populateBean步骤，根据第5步得出的 PropertyValues pvs ，分别设置bean实例属性的值，也就是真正的依赖注入操作。
7. 调用bean实例的 ```BeanNameAware```、```BeanClassLoaderAware```、```BeanFactoryAware``` 相关回调方法；
8. 遍历所有的BeanPostProcessor，调用其 ```Object postProcessBeforeInitialization(Object bean, String beanName)``` 方法；
9. 调用bean实例的 ```afterPropertiesSet()``` 方法和 ```自定义的init方法```；
10. 遍历所有的BeanPostProcessor，调用其 ```Object postProcessAfterInitialization(Object bean, String beanName) ``` 方法；





别看这么多步骤，这么多BeanPostProcessor，把上面步骤梳理一下，会发现其实只涉及几种BeanPostProcessor



```java
/**
 * InstantiationAwareBeanPostProcessor起作用的过程在一般的 BeanPostProcessor 之前
 */
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
  // ① 这里的 before instantiation 指的是 bean对象创建之前，在创建bean对象之前调用
  // 如果方法返回值不为null，那返回值就作为bean直接返回，createBean就结束了🔚
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

  // ③ 这里的 after instantiation 指的是 bean对象创建之后，并且bean对象的属性还没执行依赖注入
  // 返回值如果为 true，意味着需要进行后续的依赖注入操作
  // 返回值如果为 false，意味着🙅‍不需要进行后续的依赖注入操作了
	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
}

/**
 * MergedBeanDefinitionPostProcessor
 */
public interface MergedBeanDefinitionPostProcessor extends BeanPostProcessor {
  // ② 在创建bean对象之后，执行依赖注入之前被调用
	void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
}


/**
 * BeanPostProcessor接口只定义了两个方法
 */
public interface BeanPostProcessor {
  // ④ 这里的before initialization指的是在执行bean的生命周期初始化回调之前
  default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	// ⑤ 这里的after initialization指的是在执行bean的生命周期初始化回调之后
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```





> **Bean生命周期回调和Aware**



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200510133424779.png" alt="image-20200510133424779" style="zoom:50%;" />





<span style="color:#41B883">**① 实现Spring生命周期接口（不推荐，代码和Spring强耦合）**</span>：

1. ```InitializingBean```：实现 ```void afterPropertiesSet() throws Exception;``` 方法
2. ```DisposableBean```：实现 ```void destroy() throws Exception;``` 方法

对这两个接口的处理，内嵌在Spring源码主框架当中，不需要任何BeanPostProcessor



<span style="color:#41B883">**② 使用JSR-250 @PostConstruct 和 @PreDestroy 注解**</span>

1. ```@PostConstruct注解```：
2. ```@PreDestroy 注解```：

使用这种方式，由 ```CommonAnnotationBeanPostProcessor``` 负责解析这两个注解



<span style="color:#41B883">**③ 在xml文件中配置默认的初始化和析构方法**</span>

```java
<beans default-init-method="init" default-destroy-method="destroy">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```



初始化方法的调用顺序：

1. @PostConstruct 注解的方法
2. 实现InitializingBean接口的afterPropertiesSet()方法
3. 自定义的 init 方法



bean销毁方法的调用顺序：

1. @PreDestroy注解的方法
2. 实现DisposableBean接口的destroy()方法
3. 自定义的 destroy 方法



<span style="color:#41B883">**④ 容器启动和关闭回调**</span>

随着ApplicationContext状态走，在ApplicationContext的 ```refresh``` 操作的最后一步，会调用所有实现了Lifecycle接口的bean的 ```start()``` 方法，当ApplicationContext ```close``` 时，会在调用bean的生命周期destroy方法之前，调用所有实现了Lifecycle接口的bean的 ```stop() ```方法。



1. 实现 ```Lifecycle``` 接口：默认执行顺序为 0
2. 实现 ```SmartLifecycle ```接口：可以自定义执行顺序，getPhase()返回值越小，start阶段越是优先执行，stop阶段越是延迟执行



由 LifecycleProcessor 来回调实现了 Lifecycle 接口 bean的 Lifecycle方法，

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}

public interface Phased {

    int getPhase();
}

public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```





> **实例化一个bean对象的 4 种方式**



1. ```obtainFromSupplier(instanceSupplier, beanName)```：如果BeanDefinition当中设置了 Supplier<?> instanceSupplier 属性的值，则使用此方式，调用 instanceSupplier.get() 方法来拿到bean实例；
2. ```instantiateUsingFactoryMethod(beanName, mbd, args)```：如果BeanDefinition当中设置了 String factoryMethodName 属性的值，则使用此方式，调用工厂方法时，参数会进行依赖注入，即获取到参数的实际值，传递给工厂方法，然后执行 factoryMethod.invoke(factoryBean, args) 来拿到bean实例；
3. ```autowireConstructor(beanName, mbd, ctors, args)```：当不采用以上两种方式时，遍历 SmartInstantiationAwareBeanPostProcessor，调用其 ```Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)``` 方法，如果有processor返回非空的返回值，则使用此方式，使用processor返回的构造方法，构造方法中的参数会执行依赖注入，即获取到实际参数值，再调用 Constructor.newInstance(argsWithDefaultValues) 来拿到bean实例；
4. ```instantiateBean(beanName, mbd)```：如果以上方式都不是，则默认使用此方式，找到bean class的默认无参构造方法，调用 Constructor.newInstance(argsWithDefaultValues) 来拿到bean实例。





> **bean属性依赖注入的方式**





<span style="color:#41B883">声明依赖注入的方法（**XML配置方式，优先级高**）</span>

① 构造方法参数注入 / 工厂方法参数注入（推荐）：构造/工厂 方法参数依赖是根据参数类型来查找的，

② setter方法注入：执行完构造/工厂方法之后，Spring容器会自动执行setter方法进行依赖注入



1. <property/>指定注入依赖：容器会调用相应的setter方法注入依赖，如果xml当中没有配置，容器是不会调用setter方法的，如果bean没有定义相应的setter方法，会报错
2. <constructor-arg/>指定注入依赖：容器会利用构造方法参数注入依赖，如果类构造方法有参数，而xml当中没有配置，则会报错，如果类没有带参数的构造函数，但是xml当中配置了，则也会报错
3. 设置<autowire-type>自动注入依赖（只适用于注入bean）：用这种方式就无需配置 <property/> 或者 <constructor-arg/>，Spring容器会自动注入相应的值，有几种自动注入的方式，① no，默认不自动注入；② byName，对 **具有对应setter方法的属性** 按照依赖的bean的名称进行自动注入；③ byType，对 **具有对应setter方法的属性** 按照依赖的bean的类型进行自动注入；④ constructor，对构造方法参数按照type进行自动注入，这种方式其实就是节省了xml当中的配置，bean类当中该有setter方法的还是得有，该写构造函数的还是得写；

  

<span style="color:#41B883">声明依赖注入的方法（**注解配置方式，优先级低**）</span>

基于 注解 + 处理相应注解的BeanPostProcessor

1. 在属性上声明需要自动注入：@Autowired，@Inject，@Value，@Resource
2. 在方法上声明所有参数需要自动注入：@Autowired，@Inject，@Value，@Resource
3. 在方法参数上声明该参数需要自动注入：@Autowired，@Inject，@Value，
4. 在构造方法声明所有参数需要自动注入：@Autowired（如果只有一个构造方法，则不需要此注解），@Resource，@Inject，



需要自动或者手动向容器中注册这些BeanPostProcessor：```AutowiredAnnotationBeanPostProcessor```、```CommonAnnotationBeanPostProcessor```、```PersistenceAnnotationBeanPostProcessor```、```RequiredAnnotationBeanPostProcessor```



JSR 330标准注解

| Spring              | javax.inject.*（JSR 330） | javax.inject restrictions / comments                         |
| :------------------ | :------------------------ | :----------------------------------------------------------- |
| @Autowired          | @Inject                   | `@Inject` has no 'required' attribute. Can be used with Java 8’s `Optional` instead. |
| @Component          | @Named / @ManagedBean     | JSR-330 does not provide a composable model, only a way to identify named components. |
| @Scope("singleton") | @Singleton                | The JSR-330 default scope is like Spring’s `prototype`. However, in order to keep it consistent with Spring’s general defaults, a JSR-330 bean declared in the Spring container is a `singleton` by default. In order to use a scope other than `singleton`, you should use Spring’s `@Scope` annotation. `javax.inject` also provides a [@Scope](https://download.oracle.com/javaee/6/api/javax/inject/Scope.html) annotation. Nevertheless, this one is only intended to be used for creating your own annotations. |
| @Qualifier          | @Qualifier / @Named       | `javax.inject.Qualifier` is just a meta-annotation for building custom qualifiers. Concrete `String` qualifiers (like Spring’s `@Qualifier` with a value) can be associated through `javax.inject.Named`. |
| @Value              | -                         | no equivalent                                                |
| @Required           | -                         | no equivalent                                                |
| @Lazy               | -                         | no equivalent                                                |
| ObjectFactory       | Provider                  | `javax.inject.Provider` is a direct alternative to Spring’s `ObjectFactory`, only with a shorter `get()` method name. It can also be used in combination with Spring’s `@Autowired` or with non-annotated constructors and setter methods. |





辅助注解：@Qualifier，@Primary



CommonAnnotationBeanPostProcessor：处理 @Resource、@PostConstruct、@PreDestroy等注解



> **<autowire-type> 自动依赖注入**



关键点是自动，就是不需要在XML文件当中配置需要注入什么样子的依赖

```xml
<!-- traditional declaration with optional argument names -->
<bean id="beanOne" class="x.y.ThingOne">
  	<!-- 手动指定构造方法参数注入哪个依赖 -->
    <constructor-arg name="thingTwo" ref="beanTwo"/>
    <constructor-arg name="thingThree" ref="beanThree"/>
    <constructor-arg name="email" value="something@somewhere.com"/>
</bean>
```

让Spring自动根据我们的类当中的属性、setter方法、构造方法信息，自动查找相应的依赖bean



自动注入有 4 种模式：

```① no```，

```② byName```：自动根据属性名来查找bean

```③ byType```：自动根据属性类型来查找依赖bean，如果类型匹配的bean不止一个，会报错

```④ constructor```：自动根据构造方法参数的类型来查找依赖bean，如果类型匹配的bean不止一个，会报错



自动注入的缺点：

1. <span style="color:red">无法注入原始数据类型的值</span>
2. <span style="color:red">注入依赖不精确</span>
3. <span style="color:red">如果有多个bean匹配，则会出错</span>





> **不同scope的Bean之间的依赖**



生命周期长的bean依赖生命周期短的bean时，如何解决生命周期长的bean引用生命周期短的bean都是最新的❓

1. 在声明周期短的bean定义内部加上 ```<aop:scoped-proxy/>```（基于CGLIB Proxy，只代理public非final方法）；
2. Method Injection





## Resource



对 ```java.net.URL``` 的拓展，URL只抽象了网络资源，而Spring的Resource接口则抽象了 File 和 URL，Resource接口只是定义了统一的抽象，具体需要用到哪种资源需要使用相应的实现，Resource接口的实现类有：



* UrlResource：
* ClassPathResource：
* FileSystemResource：
* ServletContextResource：
* InputStreamResource：
* ByteArrayResource：



> **使用ResourceLoader加载Resource**



Spring的Resource必须和ResourceLoader搭配起来使用，ResourceLoader顾名思义就是资源的加载器，它根据用户指定的参数加载相应的资源，<span style="color:red">ResourceLoader只支持使用字符串格式指定资源的位置，通过使用不同格式的location string，可以实现按照不同策略加载资源</span>



```java
public interface ResourceLoader {

	/** Pseudo URL prefix for loading from the class path: "classpath:". */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;


	/**
	 * Return a Resource handle for the specified resource location.
	 * <p>The handle should always be a reusable resource descriptor,
	 * allowing for multiple {@link Resource#getInputStream()} calls.
	 * <p><ul>
	 * <li>Must support fully qualified URLs, e.g. "file:C:/test.dat".
	 * <li>Must support classpath pseudo-URLs, e.g. "classpath:test.dat".
	 * <li>Should support relative file paths, e.g. "WEB-INF/test.dat".
	 * (This will be implementation-specific, typically provided by an
	 * ApplicationContext implementation.)
	 * </ul>
	 * <p>Note that a Resource handle does not imply an existing resource;
	 * you need to invoke {@link Resource#exists} to check for existence.
	 * @param location the resource location
	 * @return a corresponding Resource handle (never {@code null})
	 * @see #CLASSPATH_URL_PREFIX
	 * @see Resource#exists()
	 * @see Resource#getInputStream()
	 */
	Resource getResource(String location);

	/**
	 * Expose the ClassLoader used by this ResourceLoader.
	 * <p>Clients which need to access the ClassLoader directly can do so
	 * in a uniform manner with the ResourceLoader, rather than relying
	 * on the thread context ClassLoader.
	 * @return the ClassLoader
	 * (only {@code null} if even the system ClassLoader isn't accessible)
	 * @see org.springframework.util.ClassUtils#getDefaultClassLoader()
	 * @see org.springframework.util.ClassUtils#forName(String, ClassLoader)
	 */
	@Nullable
	ClassLoader getClassLoader();

}
```





Spring支持的 ```location string格式``` 有：

| Prefix     | Example                          | Explanation                                                  |
| :--------- | :------------------------------- | :----------------------------------------------------------- |
| classpath: | `classpath:com/myapp/config.xml` | Loaded from the classpath.                                   |
| file:      | `file:///data/config.xml`        | Loaded as a `URL` from the filesystem. See also [`FileSystemResource` Caveats](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#resources-filesystemresource-caveats). |
| http:      | `https://myserver/logo.png`      | Loaded as a `URL`.                                           |
| (none)     | `/data/config.xml`               | Depends on the underlying `ApplicationContext`.              |





> **ResourceLoaderAware 接口**



所有的ApplicationContext都实现了 ResourceLoader 接口







## Validation, Data Binding, and Type Conversion

















> **基于 ```@Constraint``` 注解的 Validation**



```@Constraint``` + ```ConstraintValidator``` + ```ConstraintValidatorFactory```（管理ConstraintValidator实例，） 











> **Spring Validator**



Spring有自己的 Validator 体系，对bean进行validate也是直接使用Spring体系的这一套东西，包括：

```org.springframework.validation.Validator```：直接使用



```org.springframework.validation.SmartValidator```：直接使用，绑定到Databinder上的就是这种类型的Validator，



```org.springframework.boot.autoconfigure.validation.ValidatorAdapter```：这个类包含了实际上起作用的 javax.validation.Validator（属性：```private javax.validation.Validator targetValidator;```），适配器模式，把非Spring框架内的validator适配到Spring体系当中，成为一个Spring里的Validator，是个 **两面派**。



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200514072341412.png" alt="image-20200514072341412" style="zoom:45%;" />



```org.springframework.validation.beanvalidation.LocalValidatorFactoryBean``` ：这是一个连接Spring和JSR-303标准的Validator实现的中间人，**继承了 ValidatorAdapter**，Spring框架不会自动把它作为Validator bean注册到容器中，需要手动注册，或者像下面👇的springboot，就把它作为一种 javax.validation.Validator bean 注册到容器中，LocalValidatorFactoryBean在初始化的时候，会首先尝试根据JSR-303 Bean Validator标准，通过 javax.validation.Validation 这个类拿到 JSR-330 Bean Validator 的具体实现。





> **Spirngboot中的Bean Validator环境自动配置**



```org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration```：自动配置 LocalValidatorFactoryBean 为应用默认的 Validator（javax.validation.Validator） bean，



```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(ExecutableValidator.class)
// 这个文件在Hibernate具体实现的jar包当中是存在的
@ConditionalOnResource(resources = "classpath:META-INF/services/javax.validation.spi.ValidationProvider")
@Import(PrimaryDefaultValidatorPostProcessor.class)
public class ValidationAutoConfiguration {

  // 配置 springboot 环境中的默认 Validator（javax.validation.Validator） Bean
	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	@ConditionalOnMissingBean(Validator.class)
	public static LocalValidatorFactoryBean defaultValidator() {
		LocalValidatorFactoryBean factoryBean = new LocalValidatorFactoryBean();
		MessageInterpolatorFactory interpolatorFactory = new MessageInterpolatorFactory();
		factoryBean.setMessageInterpolator(interpolatorFactory.getObject());
		return factoryBean;
	}
  
  // 省略下面代码
}
```





> **JSR-303 Bean Validator标准**



JSR-303核心类 ```javax.validation.Validation```，这个类是启动 JSR-303 bean validator 的入口，通过这个类可以拿到实现了 JSR-303 bean validator标准，具体的 ```javax.validation.ValidatorFactory``` 和 ```javax.validation.Validator```



比如说 Hibernate实现的 ```org.hibernate.validator.internal.engine.ValidatorFactoryImpl``` 和 ```org.hibernate.validator.internal.engine.ValidatorImpl```



启动方式 ①：

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
```



这种启动方式，会自动从 classpath 下寻找 ```javax.validation.spi.ValidationProvider``` 的实现类，使用第一个 ValidationProvider 实现类，调用其下面👇这个方法：

```java
Configuration<?> createGenericConfiguration(BootstrapState state);
```

Configuration 类有一个方法，返回ValidatorFactory的具体实现

```java
ValidatorFactory buildValidatorFactory();
```





> **Hibernate Validator**











> **SpringWeb Databinder和Validator**











其他启动方式：

https://docs.oracle.com/javaee/6/api/javax/validation/Validation.html







Spring Validation







## 引用：



<span style="color:red">红色</span>

<span style="color:#41B883">主题色</span>