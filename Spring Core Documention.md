# Spring Core Documention



## 1. The IoC Container

### 1.1. Introduction to the Spring IoC Container and Beans



### 1.2. Container OverviewOverview



<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200403141745413.png" alt="image-20200403141745413" style="zoom:80%;float:left" />



#### 1.2.1. Configuration Metadata

Configuration Metadata指的就是 bean 的定义，配置信息包括了 bean 和用户定义的 POJO 类之间的联系，通过解析这些 bean 定义，向Spring容器中添加 bean实例，有三种 Configuration Metadata（bean配置信息），包括：

* XML-based 配置
* Annotation-based 配置   （```Spring2.5引入```）
* Java-based 配置              （```Spring3.0引入```）



#####XML-based

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
* ```lazy-init```：如果lazy-init为true，则不会在 preInstantiateSingletons 调用时实例化这些bean，对应BeanDefinition的 <span style="color:red">lazyInit</span> 属性
* ```autowire```：bean的属性值的自动注入选项，默认为no，即不自动注入bean的属性，这样bean的属性值就不会被自动设置，可选值有4个：no、byName、byType、constructor，分别对应 BeanDefinition <span style="color:red">autowireMode</span> 属性 的4个可能值：AUTOWIRE_NO（0）、AUTOWIRE_BY_NAME（1）、AUTOWIRE_BY_TYPE（2）、AUTOWIRE_CONSTRUCTOR（3），在populateBean阶段，会根据BeanDefinition的autowireMode属性来判断如何注入属性，如果是 AUTOWIRE_BY_NAME 或者 AUTOWIRE_BY_TYPE ，则会调用指定的自动注入方法，解析出属性需要注入的值，在基于注解的ApplicationContext中，一个类当中的某个属性，如果不加@Autowired之类的注解，默认autowire=0，这个属性是不会被自动注入的，这个时候的属性自动注入是@Autowired处理器来做的
* ```depends-on```：指定当前bean必须在depends-on设置的bean实例化之后，再实例化，也就是规定了指定的这些bean的实例化顺序，同时，bean在destroy过程中，必须要先销毁当前bean，才会销毁depends-on指定的从属bean。如果当前bean执行正常的功能，需要其他bean先做某些操作，但是这两个bean不存在直接的依赖关系，就可以使用depends-on。对应BeanDefinition的 <span style="color:red">String[] dependsOn</span> 属性
* ```autowire-candidate```：设置当前bean是否能被spring框架自动注入到其他的bean，对应BeanDefinition的 <span style="color:red">autowireCandidate</span> 属性，默认为true，这个属性在 populateBean 阶段执行bean的属性注入时，针对bean的属性会解析其可能的注入candidate，这个时候找到的注入bean，会判断其是否是autowire candidate，判断的依据就是其相应BeanDefinition的populateBean属性是否为true
* ```primary```：当容器中有多个同一种类型的bean时，其他bean需要注入这种类型的bean就会优先使用primary设置为true的bean，而不会注入其他bean实例，但是如果有多个同种类型的bean都设置了primary为true，自动注入会失败，不知道要注入哪个实例。对应于BeanDefinition中的
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





##### getMergedLocalBeanDefinition

XML配置文件解析之后第一步，会生成相应的 ```GenericBeanDefinition``` ，后面会有一个调用，这个操作对所有在```beanDefinitionNames```中的bean进行一个“再解析”的操作，将GenericBeanDefinition解析成RootBeanDefinition并放到 ```Map<String, RootBeanDefinition> mergedBeanDefinitions``` 里面去，如果有些bean定义了parent属性，在这个过程当中，会将这个bean和parent bean进行合并，变成一个完整的子bean，如果没有定义parent属性，则只是简简单单做一个deep copy，new一个新的RootBeanDefinition。



```java
RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
```



子bean合并父bean的操作实际上就是用子BeanDefinition部分属性的值去覆盖父BeanDefinition属性的值，不覆盖的属性值就是继承下来了，具体方法调用如下：

```java
/**
 * AbstractBeanDefinition
 */
protected RootBeanDefinition getMergedBeanDefinition(
			String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)
			throws BeanDefinitionStoreException {
  	// 省略了很多代码...
  
  	// Deep copy with overridden values.
  	// 直接用父BeanDefinition作为模板，生成RootBeanDefinition对象
		mbd = new RootBeanDefinition(pbd);
  	// 用子BeanDefinition部分属性覆盖父BeanDefinition的属性值
		mbd.overrideFrom(bd);
  
  	// 省略了很多代码...
}


/**
 * AbstractBeanDefinition
 */
// other就是子BeanDefinition
public void overrideFrom(BeanDefinition other) {
		if (StringUtils.hasLength(other.getBeanClassName())) {
			setBeanClassName(other.getBeanClassName());
		}
		if (StringUtils.hasLength(other.getScope())) {
			setScope(other.getScope());
		}
		setAbstract(other.isAbstract());
		if (StringUtils.hasLength(other.getFactoryBeanName())) {
			setFactoryBeanName(other.getFactoryBeanName());
		}
		if (StringUtils.hasLength(other.getFactoryMethodName())) {
			setFactoryMethodName(other.getFactoryMethodName());
		}
		setRole(other.getRole());
		setSource(other.getSource());
		copyAttributesFrom(other);

		if (other instanceof AbstractBeanDefinition) {
			AbstractBeanDefinition otherAbd = (AbstractBeanDefinition) other;
			if (otherAbd.hasBeanClass()) {
				setBeanClass(otherAbd.getBeanClass());
			}
			if (otherAbd.hasConstructorArgumentValues()) {
				getConstructorArgumentValues().addArgumentValues(other.getConstructorArgumentValues());
			}
			if (otherAbd.hasPropertyValues()) {
				getPropertyValues().addPropertyValues(other.getPropertyValues());
			}
			if (otherAbd.hasMethodOverrides()) {
				getMethodOverrides().addOverrides(otherAbd.getMethodOverrides());
			}
			Boolean lazyInit = otherAbd.getLazyInit();
			if (lazyInit != null) {
				setLazyInit(lazyInit);
			}
			setAutowireMode(otherAbd.getAutowireMode());
			setDependencyCheck(otherAbd.getDependencyCheck());
			setDependsOn(otherAbd.getDependsOn());
			setAutowireCandidate(otherAbd.isAutowireCandidate());
			setPrimary(otherAbd.isPrimary());
			copyQualifiersFrom(otherAbd);
			setInstanceSupplier(otherAbd.getInstanceSupplier());
			setNonPublicAccessAllowed(otherAbd.isNonPublicAccessAllowed());
			setLenientConstructorResolution(otherAbd.isLenientConstructorResolution());
			if (otherAbd.getInitMethodName() != null) {
				setInitMethodName(otherAbd.getInitMethodName());
				setEnforceInitMethod(otherAbd.isEnforceInitMethod());
			}
			if (otherAbd.getDestroyMethodName() != null) {
				setDestroyMethodName(otherAbd.getDestroyMethodName());
				setEnforceDestroyMethod(otherAbd.isEnforceDestroyMethod());
			}
			setSynthetic(otherAbd.isSynthetic());
			setResource(otherAbd.getResource());
		}
		else {
			getConstructorArgumentValues().addArgumentValues(other.getConstructorArgumentValues());
			getPropertyValues().addPropertyValues(other.getPropertyValues());
			setLazyInit(other.isLazyInit());
			setResourceDescription(other.getResourceDescription());
		}
	}
```



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

