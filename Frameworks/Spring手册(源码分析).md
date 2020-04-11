# Spring手册(源码分析)





## 常用执行流



### populateBean（通过类型自动注入）



[populateBean](#populateBean) -> [autowireByType](#autowireByType) -> [unsatisfiedNonSimpleProperties](#unsatisfiedNonSimpleProperties) -> [resolveDependency](#resolveDependency) -> 

[findAutowireCandidates](#findAutowireCandidates) -> 









## org.springframework.beans.factory



### BeanFactoryUtils



#### beanNamesForTypeIncludingAncestors

找到指定Class类型的所有bean的名称集合

```java
/**
 * 这个方法还会从父容器里面找满足条件的bean，
 * 这里面有一个很重要的参数，就是 allowEagerInit，如果这个参数值为true，那么将会使得FactoryBean
 * 提前实例化出它相关的bean实例（默认不实例化），如果这个参数值为false，FactoryBean不会实例化
 * factory bean
 */
public static String[] beanNamesForTypeIncludingAncestors(
		ListableBeanFactory lbf, Class<?> type, boolean includeNonSingletons, boolean allowEagerInit) {

	Assert.notNull(lbf, "ListableBeanFactory must not be null");
  // ⭐ 实际是调用 BeanFactory的getBeanNamesForType方法
	String[] result = lbf.getBeanNamesForType(type, includeNonSingletons, allowEagerInit);
	
  // 如果是一个HierarchicalBeanFactory，意味着当前BeanFactory有可能有父BeanFactory
  if (lbf instanceof HierarchicalBeanFactory) {
		HierarchicalBeanFactory hbf = (HierarchicalBeanFactory) lbf;
		if (hbf.getParentBeanFactory() instanceof ListableBeanFactory) {
      // 递归从父BeanFactory中拿到满足条件的bean name数组
			String[] parentResult = beanNamesForTypeIncludingAncestors(
					(ListableBeanFactory) hbf.getParentBeanFactory(), type, includeNonSingletons, allowEagerInit);
      
      // 将当前BeanFactory中找到的结果和父BeanFactory结果合并
      // 合并的逻辑就是，这个bean name不在当前BeanFactory找到的bean name结果里面
      // 也不在当前BeanFactory的bean集合里面，但是在父BeanFactory的找到的结果里
			result = mergeNamesWithParent(result, parentResult, hbf);
		}
	}
	return result;
}
```



#### transformedBeanName

获取bean的实际名字，这里主要是考虑 FactoryBean 的名字是有前缀的，获取实际名字的时候，就需要把前缀去掉

```java
public static String transformedBeanName(String name) {
	Assert.notNull(name, "'name' must not be null");
	if (!name.startsWith(BeanFactory.FACTORY_BEAN_PREFIX)) {
		return name;
	}
	return transformedBeanNameCache.computeIfAbsent(name, beanName -> {
    // 这里需要注意，一个FactoryBean可以生成另一个FactoryBean，所以这里是一个递归处理
		do {
			beanName = beanName.substring(BeanFactory.FACTORY_BEAN_PREFIX.length());
		}
		while (beanName.startsWith(BeanFactory.FACTORY_BEAN_PREFIX));
		return beanName;
	});
}
```







## org.springframework.beans.factory.support



### AbstractBeanFactory





#### getMergedLocalBeanDefinition

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





#### isTypeMatch



```java
// name就是当前要检查的BeanFacotry中的bean的名字，typeToMatch实际上包含了实际上寻找的bean的类型
protected boolean isTypeMatch(String name, ResolvableType typeToMatch, boolean allowFactoryBeanInit)
		throws NoSuchBeanDefinitionException {

	String beanName = transformedBeanName(name);
	boolean isFactoryDereference = BeanFactoryUtils.isFactoryDereference(name);

	// Check manually registered singletons.
	Object beanInstance = getSingleton(beanName, false);
	if (beanInstance != null && beanInstance.getClass() != NullBean.class) {
		if (beanInstance instanceof FactoryBean) {
			if (!isFactoryDereference) {
				Class<?> type = getTypeForFactoryBean((FactoryBean<?>) beanInstance);
				return (type != null && typeToMatch.isAssignableFrom(type));
			}
			else {
				return typeToMatch.isInstance(beanInstance);
			}
		}
		else if (!isFactoryDereference) {
			if (typeToMatch.isInstance(beanInstance)) {
				// Direct match for exposed instance?
				return true;
			}
			else if (typeToMatch.hasGenerics() && containsBeanDefinition(beanName)) {
				// Generics potentially only match on the target class, not on the proxy...
				RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				Class<?> targetType = mbd.getTargetType();
				if (targetType != null && targetType != ClassUtils.getUserClass(beanInstance)) {
					// Check raw class match as well, making sure it's exposed on the proxy.
					Class<?> classToMatch = typeToMatch.resolve();
					if (classToMatch != null && !classToMatch.isInstance(beanInstance)) {
						return false;
					}
					if (typeToMatch.isAssignableFrom(targetType)) {
						return true;
					}
				}
				ResolvableType resolvableType = mbd.targetType;
				if (resolvableType == null) {
					resolvableType = mbd.factoryMethodReturnType;
				}
				return (resolvableType != null && typeToMatch.isAssignableFrom(resolvableType));
			}
		}
		return false;
	}
	else if (containsSingleton(beanName) && !containsBeanDefinition(beanName)) {
		// null instance registered
		return false;
	}

	// No singleton instance found -> check bean definition.
	BeanFactory parentBeanFactory = getParentBeanFactory();
	if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
		// No bean definition found in this factory -> delegate to parent.
		return parentBeanFactory.isTypeMatch(originalBeanName(name), typeToMatch);
	}

	// Retrieve corresponding bean definition.
	RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
	BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();

	// Setup the types that we want to match against
	Class<?> classToMatch = typeToMatch.resolve();
	if (classToMatch == null) {
		classToMatch = FactoryBean.class;
	}
	Class<?>[] typesToMatch = (FactoryBean.class == classToMatch ?
			new Class<?>[] {classToMatch} : new Class<?>[] {FactoryBean.class, classToMatch});


	// Attempt to predict the bean type
	Class<?> predictedType = null;

	// We're looking for a regular reference but we're a factory bean that has
	// a decorated bean definition. The target bean should be the same type
	// as FactoryBean would ultimately return.
	if (!isFactoryDereference && dbd != null && isFactoryBean(beanName, mbd)) {
		// We should only attempt if the user explicitly set lazy-init to true
		// and we know the merged bean definition is for a factory bean.
		if (!mbd.isLazyInit() || allowFactoryBeanInit) {
			RootBeanDefinition tbd = getMergedBeanDefinition(dbd.getBeanName(), dbd.getBeanDefinition(), mbd);
			Class<?> targetType = predictBeanType(dbd.getBeanName(), tbd, typesToMatch);
			if (targetType != null && !FactoryBean.class.isAssignableFrom(targetType)) {
				predictedType = targetType;
			}
		}
	}

	// If we couldn't use the target type, try regular prediction.
	if (predictedType == null) {
		predictedType = predictBeanType(beanName, mbd, typesToMatch);
		if (predictedType == null) {
			return false;
		}
	}

	// Attempt to get the actual ResolvableType for the bean.
	ResolvableType beanType = null;

	// If it's a FactoryBean, we want to look at what it creates, not the factory class.
	if (FactoryBean.class.isAssignableFrom(predictedType)) {
		if (beanInstance == null && !isFactoryDereference) {
			beanType = getTypeForFactoryBean(beanName, mbd, allowFactoryBeanInit);
			predictedType = beanType.resolve();
			if (predictedType == null) {
				return false;
			}
		}
	}
	else if (isFactoryDereference) {
		// Special case: A SmartInstantiationAwareBeanPostProcessor returned a non-FactoryBean
		// type but we nevertheless are being asked to dereference a FactoryBean...
		// Let's check the original bean class and proceed with it if it is a FactoryBean.
		predictedType = predictBeanType(beanName, mbd, FactoryBean.class);
		if (predictedType == null || !FactoryBean.class.isAssignableFrom(predictedType)) {
			return false;
		}
	}

	// We don't have an exact type but if bean definition target type or the factory
	// method return type matches the predicted type then we can use that.
	if (beanType == null) {
		ResolvableType definedType = mbd.targetType;
		if (definedType == null) {
			definedType = mbd.factoryMethodReturnType;
		}
		if (definedType != null && definedType.resolve() == predictedType) {
			beanType = definedType;
		}
	}

	// If we have a bean type use it so that generics are considered
	if (beanType != null) {
		return typeToMatch.isAssignableFrom(beanType);
	}

	// If we don't have a bean type, fallback to the predicted type
	return typeToMatch.isAssignableFrom(predictedType);
}
```











### AbstractAutowireCapableBeanFactory



#### autowireByType

这个方法实现了对一个具体的bean实例中还未注入的属性，进行自动注入值解析的操作，如果解析成功，bean实例的某个属性值可以解析出其值，就把 propertyName -> autowiredValue 保存到Map中，并由方法完成所有属性值注入到实例的过程

```java
protected void autowireByType(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

	TypeConverter converter = getCustomTypeConverter();
	if (converter == null) {
		converter = bw;
	}

	Set<String> autowiredBeanNames = new LinkedHashSet<>(4);
  // ① 只对需要注入的属性进行自动注入的操作
	String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
	for (String propertyName : propertyNames) {
		try {
			PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
			// Don't try autowiring by type for type Object: never makes sense,
			// even if it technically is a unsatisfied, non-simple property.
      // ② 不要对 Object 类型属性浪费感情！
			if (Object.class != pd.getPropertyType()) {
				MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
				// Do not allow eager init for type matching in case of a prioritized post-processor.
				boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);
				DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);
        // ③ 从BeanFactory中解析出属性需要自动注入的值
				Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
				if (autowiredArgument != null) {
          // ④ 记录属性和对应需要注入的值的匹配关系
					pvs.add(propertyName, autowiredArgument);
				}
        
        // ⑤ 将依赖关系保存到两个map
        // dependentBeanMap: 被依赖的bean name -> 依赖这个bean的bean name集合
        // dependenciesForBeanMap: bean name -> bean依赖的bean的name集合
        // 将数据保存到这两个map是为了保证一个bean在destroy的时候，依赖其的bean需要先destroy
        // 另外还可以快速获取到一个bean依赖了哪些其他的bean
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



#### applyPropertyValues



```java
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
	// 没有需要注入的值，啥也不做
  if (pvs.isEmpty()) {
		return;
	}

	if (System.getSecurityManager() != null && bw instanceof BeanWrapperImpl) {
		((BeanWrapperImpl) bw).setSecurityContext(getAccessControlContext());
	}

	MutablePropertyValues mpvs = null;
	List<PropertyValue> original;

	if (pvs instanceof MutablePropertyValues) {
		mpvs = (MutablePropertyValues) pvs;
		if (mpvs.isConverted()) {
			// Shortcut: use the pre-converted values as-is.
			try {
				bw.setPropertyValues(mpvs);
				return;
			}
			catch (BeansException ex) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Error setting property values", ex);
			}
		}
		original = mpvs.getPropertyValueList();
	}
	else {
		original = Arrays.asList(pvs.getPropertyValues());
	}

	TypeConverter converter = getCustomTypeConverter();
	if (converter == null) {
		converter = bw;
	}
	BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);

	// Create a deep copy, resolving any references for values.
	List<PropertyValue> deepCopy = new ArrayList<>(original.size());
	boolean resolveNecessary = false;
	for (PropertyValue pv : original) {
		if (pv.isConverted()) {
			deepCopy.add(pv);
		}
		else {
			String propertyName = pv.getName();
			Object originalValue = pv.getValue();
			if (originalValue == AutowiredPropertyMarker.INSTANCE) {
				Method writeMethod = bw.getPropertyDescriptor(propertyName).getWriteMethod();
				if (writeMethod == null) {
					throw new IllegalArgumentException("Autowire marker for property without write method: " + pv);
				}
				originalValue = new DependencyDescriptor(new MethodParameter(writeMethod, 0), true);
			}
			Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
			Object convertedValue = resolvedValue;
			boolean convertible = bw.isWritableProperty(propertyName) &&
					!PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
			if (convertible) {
				convertedValue = convertForProperty(resolvedValue, propertyName, bw, converter);
			}
			// Possibly store converted value in merged bean definition,
			// in order to avoid re-conversion for every created bean instance.
			if (resolvedValue == originalValue) {
				if (convertible) {
					pv.setConvertedValue(convertedValue);
				}
				deepCopy.add(pv);
			}
			else if (convertible && originalValue instanceof TypedStringValue &&
					!((TypedStringValue) originalValue).isDynamic() &&
					!(convertedValue instanceof Collection || ObjectUtils.isArray(convertedValue))) {
				pv.setConvertedValue(convertedValue);
				deepCopy.add(pv);
			}
			else {
				resolveNecessary = true;
				deepCopy.add(new PropertyValue(pv, convertedValue));
			}
		}
	}
	if (mpvs != null && !resolveNecessary) {
		mpvs.setConverted();
	}

	// Set our (possibly massaged) deep copy.
	try {
		bw.setPropertyValues(new MutablePropertyValues(deepCopy));
	}
	catch (BeansException ex) {
		throw new BeanCreationException(
				mbd.getResourceDescription(), beanName, "Error setting property values", ex);
	}
}
```







#### unsatisfiedNonSimpleProperties

检查一个bean中有哪些需要注入的属性还没有被注入

```java
protected String[] unsatisfiedNonSimpleProperties(AbstractBeanDefinition mbd, BeanWrapper bw) {
	Set<String> result = new TreeSet<>();
  // ① 这里的属性值实际上是bean定义中指定的那些 <property> 值，在定义时已经设置好值
	PropertyValues pvs = mbd.getPropertyValues();
	PropertyDescriptor[] pds = bw.getPropertyDescriptors();
	for (PropertyDescriptor pd : pds) {
    // ② 有四个条件：
    //    ① 该属性要有相应的 setter 方法
    //    ② 该属性没有被排除在依赖检查之外，比如说CGLIB生成的属性就不需要执行属性注入
    //    ③ bean定义时已经设置好值的属性除外（已经有值了，就不需要再自动注入了嘛）
    //    ④ 该属性不是简单类型
		if (pd.getWriteMethod() != null && !isExcludedFromDependencyCheck(pd) && !pvs.contains(pd.getName()) &&
				!BeanUtils.isSimpleProperty(pd.getPropertyType())) {
			result.add(pd.getName());
		}
	}
	return StringUtils.toStringArray(result);
}
```



###DefaultListableBeanFactory



#### populateBean

已经完成bean对应的POJO对象实例化之后，下面就需要进行bean实例的属性填充了

```java
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
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
			autowireByName(beanName, mbd, bw, newPvs);
		}
		// Add property values based on autowire by type if applicable.
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
    // bean实例属性注入，组装这个bean
		applyPropertyValues(beanName, mbd, bw, pvs);
	}
}
```









#### resolveDependency



```java
public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,
		@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

	descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());
  // descriptor.getDependencyType()解析需要注入的依赖类型
  // 可以解析 setter 方法的参数类型
	if (Optional.class == descriptor.getDependencyType()) {
		return createOptionalDependency(descriptor, requestingBeanName);
	}
	else if (ObjectFactory.class == descriptor.getDependencyType() ||
			ObjectProvider.class == descriptor.getDependencyType()) {
		return new DependencyObjectProvider(descriptor, requestingBeanName);
	}
	else if (javaxInjectProviderClass == descriptor.getDependencyType()) {
		return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);
	}
	else {
		Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(
				descriptor, requestingBeanName);
		if (result == null) {
      // 调用下面👇的 doResolveDependency 方法
			result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
		}
		return result;
	}
}


// doResolveDependency
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
		@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

	InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
	try {
		Object shortcut = descriptor.resolveShortcut(this);
		if (shortcut != null) {
			return shortcut;
		}

    // 又来一遍，获取依赖的 Class 类型
		Class<?> type = descriptor.getDependencyType();
		Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
		if (value != null) {
			if (value instanceof String) {
				String strVal = resolveEmbeddedValue((String) value);
				BeanDefinition bd = (beanName != null && containsBean(beanName) ?
						getMergedBeanDefinition(beanName) : null);
				value = evaluateBeanDefinitionString(strVal, bd);
			}
			TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
			try {
				return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
			}
			catch (UnsupportedOperationException ex) {
				// A custom TypeConverter which does not support TypeDescriptor resolution...
				return (descriptor.getField() != null ?
						converter.convertIfNecessary(value, type, descriptor.getField()) :
						converter.convertIfNecessary(value, type,descriptor.getMethodParameter()));
			}
		}

		Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
		if (multipleBeans != null) {
			return multipleBeans;
		}

    // 从BeanFactory中寻找当前bean需要的某个类型的属性 候选注入bean集合
    // 这个方法返回后有几种情况：
    // ① 
		Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
		if (matchingBeans.isEmpty()) {
			if (isRequired(descriptor)) {
				raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
			}
			return null;
		}

		String autowiredBeanName;
		Object instanceCandidate;

		if (matchingBeans.size() > 1) {
			autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
			if (autowiredBeanName == null) {
				if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
					return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
				}
				else {
					// In case of an optional Collection/Map, silently ignore a non-unique case:
					// possibly it was meant to be an empty collection of multiple regular beans
					// (before 4.3 in particular when we didn't even look for collection beans).
					return null;
				}
			}
			instanceCandidate = matchingBeans.get(autowiredBeanName);
		}
		else {
			// We have exactly one match.
			Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
			autowiredBeanName = entry.getKey();
			instanceCandidate = entry.getValue();
		}

		if (autowiredBeanNames != null) {
			autowiredBeanNames.add(autowiredBeanName);
		}
		if (instanceCandidate instanceof Class) {
      // resolveCandidate 实际上就是 beanFactory.getBean(autowiredBeanName)
			instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
		}
		Object result = instanceCandidate;
		if (result instanceof NullBean) {
			if (isRequired(descriptor)) {
				raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
			}
			result = null;
		}
		if (!ClassUtils.isAssignableValue(type, result)) {
			throw new BeanNotOfRequiredTypeException(autowiredBeanName, type, instanceCandidate.getClass());
		}
		return result;
	}
	finally {
		ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
	}
}
```





#### findAutowireCandidates

寻找指定Class类型的bean实例，结果可能找到多个，所以返回值是一个Map类型，



```java
protected Map<String, Object> findAutowireCandidates(
		@Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {

  // 找到和requireType匹配的bean candidate的 name数组
  // 一看是BeanFactoryUtils就知道调用的是给BeanFactory配套的工具类方法
	String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
			this, requiredType, true, descriptor.isEager());
	Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);
	for (Map.Entry<Class<?>, Object> classObjectEntry : this.resolvableDependencies.entrySet()) {
		Class<?> autowiringType = classObjectEntry.getKey();
		if (autowiringType.isAssignableFrom(requiredType)) {
			Object autowiringValue = classObjectEntry.getValue();
			autowiringValue = AutowireUtils.resolveAutowiringValue(autowiringValue, requiredType);
			if (requiredType.isInstance(autowiringValue)) {
				result.put(ObjectUtils.identityToString(autowiringValue), autowiringValue);
				break;
			}
		}
	}
  
	for (String candidate : candidateNames) {
    // ① 不能自己注入自己
    // ② 找到的候选bean必须 autowireCandidate 不为 false
		if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
			addCandidateEntry(result, candidate, descriptor, requiredType);
		}
	}
  
	if (result.isEmpty()) {
		boolean multiple = indicatesMultipleBeans(requiredType);
		// Consider fallback matches if the first pass failed to find anything...
		DependencyDescriptor fallbackDescriptor = descriptor.forFallbackMatch();
		for (String candidate : candidateNames) {
			if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, fallbackDescriptor) &&
					(!multiple || getAutowireCandidateResolver().hasQualifier(descriptor))) {
				addCandidateEntry(result, candidate, descriptor, requiredType);
			}
		}
		if (result.isEmpty() && !multiple) {
			// Consider self references as a final pass...
			// but in the case of a dependency collection, not the very same bean itself.
			for (String candidate : candidateNames) {
				if (isSelfReference(beanName, candidate) &&
						(!(descriptor instanceof MultiElementDescriptor) || !beanName.equals(candidate)) &&
						isAutowireCandidate(candidate, fallbackDescriptor)) {
					addCandidateEntry(result, candidate, descriptor, requiredType);
				}
			}
		}
	}
  
	return result;
}
```





#### getBeanNamesForType



```java
public String[] getBeanNamesForType(@Nullable Class<?> type, boolean includeNonSingletons, boolean allowEagerInit) {
	if (!isConfigurationFrozen() || type == null || !allowEagerInit) {
		return doGetBeanNamesForType(ResolvableType.forRawClass(type), includeNonSingletons, allowEagerInit);
	}
  
  // 从这里和第15的cache操作来看，type -> bean name 的映射是会被缓存的
  // 而且缓存有两个，一个是包含所有bean的，一个是只包含singleton bean的
	Map<Class<?>, String[]> cache =
			(includeNonSingletons ? this.allBeanNamesByType : this.singletonBeanNamesByType);
	String[] resolvedBeanNames = cache.get(type);
	if (resolvedBeanNames != null) {
		return resolvedBeanNames;
	}
  
  // 缓存中没有时，就要真正去解析，调用下面👇的方法
	resolvedBeanNames = doGetBeanNamesForType(ResolvableType.forRawClass(type), includeNonSingletons, true);
	
  // 缓存解析结果到缓存，下次就轻松了
  if (ClassUtils.isCacheSafe(type, getBeanClassLoader())) {
		cache.put(type, resolvedBeanNames);
	}
	return resolvedBeanNames;
}


// 会从两个数据源去找到满足条件的bean name，① 从扫描bean配置得到的 beanDefinitionNames 中找
// ② 从手动添加的 manualSingletonNames 中找
private String[] doGetBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit) {
	List<String> result = new ArrayList<>();

	// 从 beanDefinitionNames 中找
	for (String beanName : this.beanDefinitionNames) {
		// Only consider bean as eligible if the bean name
		// is not defined as alias for some other bean.
		if (!isAlias(beanName)) {
			try {
				RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				// Only check bean definition if it is complete.
				if (!mbd.isAbstract() && (allowEagerInit ||
						(mbd.hasBeanClass() || !mbd.isLazyInit() || isAllowEagerClassLoading()) &&
								!requiresEagerInitForType(mbd.getFactoryBeanName()))) {
					boolean isFactoryBean = isFactoryBean(beanName, mbd);
					BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();
					boolean matchFound = false;
					boolean allowFactoryBeanInit = allowEagerInit || containsSingleton(beanName);
					boolean isNonLazyDecorated = dbd != null && !mbd.isLazyInit();
					if (!isFactoryBean) {
						if (includeNonSingletons || isSingleton(beanName, mbd, dbd)) {
              // ⭐ 核心还是通过 isTypeMatch 方法来判断
							matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
						}
					}
					else  {
						if (includeNonSingletons || isNonLazyDecorated ||
								(allowFactoryBeanInit && isSingleton(beanName, mbd, dbd))) {
							matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
						}
						if (!matchFound) {
							// In case of FactoryBean, try to match FactoryBean instance itself next.
							beanName = FACTORY_BEAN_PREFIX + beanName;
							matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
						}
					}
					if (matchFound) {
						result.add(beanName);
					}
				}
			}
			catch (CannotLoadBeanClassException | BeanDefinitionStoreException ex) {
				if (allowEagerInit) {
					throw ex;
				}
				// Probably a placeholder: let's ignore it for type matching purposes.
				LogMessage message = (ex instanceof CannotLoadBeanClassException) ?
						LogMessage.format("Ignoring bean class loading failure for bean '%s'", beanName) :
						LogMessage.format("Ignoring unresolvable metadata in bean definition '%s'", beanName);
				logger.trace(message, ex);
				onSuppressedException(ex);
			}
		}
	}


	// Check manually registered singletons too.
	for (String beanName : this.manualSingletonNames) {
		try {
			// In case of FactoryBean, match object created by FactoryBean.
			if (isFactoryBean(beanName)) {
				if ((includeNonSingletons || isSingleton(beanName)) && isTypeMatch(beanName, type)) {
					result.add(beanName);
					// Match found for this bean: do not match FactoryBean itself anymore.
					continue;
				}
				// In case of FactoryBean, try to match FactoryBean itself next.
				beanName = FACTORY_BEAN_PREFIX + beanName;
			}
			// Match raw bean instance (might be raw FactoryBean).
			if (isTypeMatch(beanName, type)) {
				result.add(beanName);
			}
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Shouldn't happen - probably a result of circular reference resolution...
			logger.trace(LogMessage.format("Failed to check manually registered singleton with name '%s'", beanName), ex);
		}
	}

	return StringUtils.toStringArray(result);
}
```









### AutowireUtils





#### isExcludedFromDependencyCheck

判断一个bean的某个属性是否不需要依赖检查，不需要依赖检查的属性就不需要注入值

```java
public static boolean isExcludedFromDependencyCheck(PropertyDescriptor pd) {
  // ① 首先，bean的属性如果没有 setter 方法，就意味着不会对其进行依赖注入处理
	Method wm = pd.getWriteMethod();
	if (wm == null) {
		return false;
	}
  // ② CGLIB生成的子类名字都是原有类名后面跟上这种 $$ ，不是CGLIB生成的代理类才需要依赖检查
	if (!wm.getDeclaringClass().getName().contains("$$")) {
		// Not a CGLIB method so it's OK.
		return false;
	}
	// It was declared by CGLIB, but we might still want to autowire it
	// if it was actually declared by the superclass.
  // ③ CGLIB是生成指定类的子类来实现代理的，父类，也就是developer写的类，其中的属性还是要检查的
	Class<?> superclass = wm.getDeclaringClass().getSuperclass();
	return !ClassUtils.hasMethod(superclass, wm);
}
```





### ConstructorResolver



####instantiateUsingFactoryMethod

调用工厂方法创建bean对象，注意，此时创建出的bean对象，还没有进行自动注入，之后会注入



```java
public BeanWrapper instantiateUsingFactoryMethod(
		String beanName, RootBeanDefinition mbd, @Nullable Object[] explicitArgs) {

	BeanWrapperImpl bw = new BeanWrapperImpl();
	this.beanFactory.initBeanWrapper(bw);

	Object factoryBean;
	Class<?> factoryClass;
	boolean isStatic;

  // 工厂方法创建bean对象有两种情况 ① bean工厂；② 静态工厂
	String factoryBeanName = mbd.getFactoryBeanName();
	if (factoryBeanName != null) {
    // bean definition中含有factory bean name则属于第①种情况
		if (factoryBeanName.equals(beanName)) {
			throw new BeanDefinitionStoreException(mbd.getResourceDescription(), beanName,
					"factory-bean reference points back to the same bean definition");
		}
		factoryBean = this.beanFactory.getBean(factoryBeanName);
		if (mbd.isSingleton() && this.beanFactory.containsSingleton(beanName)) {
			throw new ImplicitlyAppearedSingletonException();
		}
		factoryClass = factoryBean.getClass();
		isStatic = false;
	}
	else {
		// 第②种情况，使用静态工厂
		if (!mbd.hasBeanClass()) {
			throw new BeanDefinitionStoreException(mbd.getResourceDescription(), beanName,
					"bean definition declares neither a bean class nor a factory-bean reference");
		}
		factoryBean = null;
		factoryClass = mbd.getBeanClass();
		isStatic = true;
	}

  // 通过上面👆的步骤，已经确定了工厂类class接下来就要去工厂类当中寻找符合条件的工厂方法
  // 并确定工厂方法调用参数了
	Method factoryMethodToUse = null;
	ArgumentsHolder argsHolderToUse = null;
	Object[] argsToUse = null;

  // 这个参数是 getBean(...)调用时传递的参数
	if (explicitArgs != null) {
		argsToUse = explicitArgs;
	}
	else {
		Object[] argsToResolve = null;
		synchronized (mbd.constructorArgumentLock) {
			factoryMethodToUse = (Method) mbd.resolvedConstructorOrFactoryMethod;
      // 查询缓存，找到工厂方法参数值
			if (factoryMethodToUse != null && mbd.constructorArgumentsResolved) {
				// 工厂方法参数有两种类型，一种是解析过的，可以直接拿来用，一种是没有解析过的，但是包含
        // 参数相关信息，需要解析才能获取它的值
        // resolvedConstructorArguments就是解析过的值
				argsToUse = mbd.resolvedConstructorArguments;
				if (argsToUse == null) {
          // 如果没有解析过的值，就退一步看看有没有没有解析过的参数
					argsToResolve = mbd.preparedConstructorArguments;
				}
			}
		}
		if (argsToResolve != null) {
      // 解析参数，拿到真正的值
			argsToUse = resolvePreparedArguments(beanName, mbd, bw, factoryMethodToUse, argsToResolve, true);
		}
	}

  // 下面👇这个代码块将会确定两个值，一个是工厂方法对象，一个是工厂方法参数
	if (factoryMethodToUse == null || argsToUse == null) {
		// Need to determine the factory method...
		// Try all methods with this name to see if they match the given arguments.
		factoryClass = ClassUtils.getUserClass(factoryClass);

		List<Method> candidates = null;
    // 拿到缓存的工厂方法对象
		if (mbd.isFactoryMethodUnique) {
			if (factoryMethodToUse == null) {
				factoryMethodToUse = mbd.getResolvedFactoryMethod();
			}
			if (factoryMethodToUse != null) {
				candidates = Collections.singletonList(factoryMethodToUse);
			}
		}
    
    
		if (candidates == null) {
			candidates = new ArrayList<>();
      // 获取到工厂类所有的方法
			Method[] rawCandidates = getCandidateMethods(factoryClass, mbd);
			for (Method candidate : rawCandidates) {
        // 如果使用的是静态工厂模式，则只找static方法并且方法名和 factory method name相同的方法
        // 如果使用的是bean工厂模式，则只找实例方法并且方法名和 factory method name相同的方法
				if (Modifier.isStatic(candidate.getModifiers()) == isStatic && mbd.isFactoryMethod(candidate)) {
					candidates.add(candidate);
				}
			}
		}

    // 只找到一个工厂方法，如果此时getBean(...)没有传递参数，bean definition定义时
    // 也没有配置工厂方法参数，那就是这个方法了，调用工厂方法，返回bean对象
		if (candidates.size() == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
			Method uniqueCandidate = candidates.get(0);
			if (uniqueCandidate.getParameterCount() == 0) {
				mbd.factoryMethodToIntrospect = uniqueCandidate;
				synchronized (mbd.constructorArgumentLock) {
					mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
					mbd.constructorArgumentsResolved = true;
					mbd.resolvedConstructorArguments = EMPTY_ARGS;
				}
				bw.setBeanInstance(instantiate(beanName, mbd, factoryBean, uniqueCandidate, EMPTY_ARGS));
				return bw;
			}
		}

		if (candidates.size() > 1) {  // explicitly skip immutable singletonList
			candidates.sort(AutowireUtils.EXECUTABLE_COMPARATOR);
		}

    // resolvedValues存储了bean definition当中配置的参数值，这个参数值应该是解析过后的实际值
		ConstructorArgumentValues resolvedValues = null;
		boolean autowiring = (mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
		int minTypeDiffWeight = Integer.MAX_VALUE;
		Set<Method> ambiguousFactoryMethods = null;

    // minNrOfArgs是为了筛选符合条件的工厂方法
    // ① 如果bean definition当中定义了工厂方法的参数，那么我们找工厂方法的时候，方法的参数个数
    // 肯定要多余或者等于 bean definition当中定义的工厂方法参数个数，否则方法肯定不匹配
    // ② 如果getBean(...)方法有传递参数，那目标工厂方法的参数个数也不能少于getBean(...)参数个数
		int minNrOfArgs;
		if (explicitArgs != null) {
			minNrOfArgs = explicitArgs.length;
		}
		else {
			// We don't have arguments passed in programmatically, so we need to resolve the
			// arguments specified in the constructor arguments held in the bean definition.
			if (mbd.hasConstructorArgumentValues()) {
				ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
				resolvedValues = new ConstructorArgumentValues();
				minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
			}
			else {
				minNrOfArgs = 0;
			}
		}

		LinkedList<UnsatisfiedDependencyException> causes = null;

		for (Method candidate : candidates) {
			int parameterCount = candidate.getParameterCount();
      
      // 方法参数过少的直接就跳过
			if (parameterCount >= minNrOfArgs) {
				ArgumentsHolder argsHolder;

				Class<?>[] paramTypes = candidate.getParameterTypes();
				if (explicitArgs != null) {
					// Explicit arguments given -> arguments length must match exactly.
					if (paramTypes.length != explicitArgs.length) {
						continue;
					}
          
          // argsHolder保存了实际传递给工厂方法的参数
					argsHolder = new ArgumentsHolder(explicitArgs);
				}
				else {
					// Resolved constructor arguments: type conversion and/or autowiring necessary.
					try {
						String[] paramNames = null;
						ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
						if (pnd != null) {
							paramNames = pnd.getParameterNames(candidate);
						}
            
            // 有可能 resolvedValues中参数个数小于工厂方法的参数个数，那这个时候，就要看bean
            // 的autowireMode，如果是constructor方式，则少的参数传递进默认值，否则报错
						argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw,
								paramTypes, paramNames, candidate, autowiring, candidates.size() == 1);
					}
					catch (UnsatisfiedDependencyException ex) {
						if (logger.isTraceEnabled()) {
							logger.trace("Ignoring factory method [" + candidate + "] of bean '" + beanName + "': " + ex);
						}
						// Swallow and try next overloaded factory method.
						if (causes == null) {
							causes = new LinkedList<>();
						}
						causes.add(ex);
						continue;
					}
				}

        // typeDiffWeight用来衡量给定参数和方法参数的匹配程度，值越小，匹配度越大
        // 比如说两个工厂方法，一个需要的方法参数类型是Parent，一个需要的类型是Child
        // 那现在传递的参数类型是Child，那肯定是第二个工厂方法更加匹配
				int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
						argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
				// Choose this factory method if it represents the closest match.
        // 选择出匹配度最高的工厂方法
				if (typeDiffWeight < minTypeDiffWeight) {
					factoryMethodToUse = candidate;
					argsHolderToUse = argsHolder;
					argsToUse = argsHolder.arguments;
					minTypeDiffWeight = typeDiffWeight;
					ambiguousFactoryMethods = null;
				}
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

		if (factoryMethodToUse == null || argsToUse == null) {
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
			else if (resolvedValues != null) {
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
			mbd.factoryMethodToIntrospect = factoryMethodToUse;
			argsHolderToUse.storeCache(mbd, factoryMethodToUse);
		}
	}

	bw.setBeanInstance(instantiate(beanName, mbd, factoryBean, factoryMethodToUse, argsToUse));
	return bw;
}
```



#### autowireConstructor

其实和上面👆 [instantiateUsingFactoryMethod](#instantiateUsingFactoryMethod) 的处理流程类似，都需要找到匹配的方法，解析方法参数



```java
public BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd,
		@Nullable Constructor<?>[] chosenCtors, @Nullable Object[] explicitArgs) {

	BeanWrapperImpl bw = new BeanWrapperImpl();
	this.beanFactory.initBeanWrapper(bw);

	Constructor<?> constructorToUse = null;
	ArgumentsHolder argsHolderToUse = null;
	Object[] argsToUse = null;

  // 如果 getBean(...) 传递了参数，则用传递的参数
	if (explicitArgs != null) {
		argsToUse = explicitArgs;
	}
	else {
		Object[] argsToResolve = null;
		synchronized (mbd.constructorArgumentLock) {
			constructorToUse = (Constructor<?>) mbd.resolvedConstructorOrFactoryMethod;
			if (constructorToUse != null && mbd.constructorArgumentsResolved) {
				// Found a cached constructor...
				argsToUse = mbd.resolvedConstructorArguments;
				if (argsToUse == null) {
					argsToResolve = mbd.preparedConstructorArguments;
				}
			}
		}
		if (argsToResolve != null) {
			argsToUse = resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve, true);
		}
	}

	if (constructorToUse == null || argsToUse == null) {
		// Take specified constructors, if any.
		Constructor<?>[] candidates = chosenCtors;
		if (candidates == null) {
			Class<?> beanClass = mbd.getBeanClass();
			try {
        // 拿到类所有的构造方法，来一个个检测
				candidates = (mbd.isNonPublicAccessAllowed() ?
						beanClass.getDeclaredConstructors() : beanClass.getConstructors());
			}
			catch (Throwable ex) {
				throw new BeanCreationException(mbd.getResourceDescription(), beanName,
						"Resolution of declared constructors on bean Class [" + beanClass.getName() +
						"] from ClassLoader [" + beanClass.getClassLoader() + "] failed", ex);
			}
		}

    // 默认构造方法
		if (candidates.length == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
			Constructor<?> uniqueCandidate = candidates[0];
			if (uniqueCandidate.getParameterCount() == 0) {
				synchronized (mbd.constructorArgumentLock) {
					mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
					mbd.constructorArgumentsResolved = true;
					mbd.resolvedConstructorArguments = EMPTY_ARGS;
				}
				bw.setBeanInstance(instantiate(beanName, mbd, uniqueCandidate, EMPTY_ARGS));
				return bw;
			}
		}

		// ⚠️ 如果bean的自动注入模式是构造方法注入，那么如果构造方法参数有的没有解析到值，会传入默认值而
    // 不报错，否则报错
		boolean autowiring = (chosenCtors != null ||
				mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
		ConstructorArgumentValues resolvedValues = null;

		int minNrOfArgs;
		if (explicitArgs != null) {
			minNrOfArgs = explicitArgs.length;
		}
		else {
			ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
			resolvedValues = new ConstructorArgumentValues();
			minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
		}

		AutowireUtils.sortConstructors(candidates);
		int minTypeDiffWeight = Integer.MAX_VALUE;
		Set<Constructor<?>> ambiguousConstructors = null;
		LinkedList<UnsatisfiedDependencyException> causes = null;

		for (Constructor<?> candidate : candidates) {

			int parameterCount = candidate.getParameterCount();

			if (constructorToUse != null && argsToUse != null && argsToUse.length > parameterCount) {
				// Already found greedy constructor that can be satisfied ->
				// do not look any further, there are only less greedy constructors left.
				break;
			}
			if (parameterCount < minNrOfArgs) {
				continue;
			}

			ArgumentsHolder argsHolder;
			Class<?>[] paramTypes = candidate.getParameterTypes();
			if (resolvedValues != null) {
				try {
					String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, parameterCount);
					if (paramNames == null) {
						ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
						if (pnd != null) {
							paramNames = pnd.getParameterNames(candidate);
						}
					}
					argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
							getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
				}
				catch (UnsatisfiedDependencyException ex) {
					if (logger.isTraceEnabled()) {
						logger.trace("Ignoring constructor [" + candidate + "] of bean '" + beanName + "': " + ex);
					}
					// Swallow and try next constructor.
					if (causes == null) {
						causes = new LinkedList<>();
					}
					causes.add(ex);
					continue;
				}
			}
			else {
				// Explicit arguments given -> arguments length must match exactly.
				if (parameterCount != explicitArgs.length) {
					continue;
				}
				argsHolder = new ArgumentsHolder(explicitArgs);
			}

			int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
					argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
			// Choose this constructor if it represents the closest match.
			if (typeDiffWeight < minTypeDiffWeight) {
				constructorToUse = candidate;
				argsHolderToUse = argsHolder;
				argsToUse = argsHolder.arguments;
				minTypeDiffWeight = typeDiffWeight;
				ambiguousConstructors = null;
			}
			else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
				if (ambiguousConstructors == null) {
					ambiguousConstructors = new LinkedHashSet<>();
					ambiguousConstructors.add(constructorToUse);
				}
				ambiguousConstructors.add(candidate);
			}
		}

		if (constructorToUse == null) {
			if (causes != null) {
				UnsatisfiedDependencyException ex = causes.removeLast();
				for (Exception cause : causes) {
					this.beanFactory.onSuppressedException(cause);
				}
				throw ex;
			}
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Could not resolve matching constructor " +
					"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities)");
		}
		else if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Ambiguous constructor matches found in bean '" + beanName + "' " +
					"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " +
					ambiguousConstructors);
		}

		if (explicitArgs == null && argsHolderToUse != null) {
			argsHolderToUse.storeCache(mbd, constructorToUse);
		}
	}

	Assert.state(argsToUse != null, "Unresolved constructor arguments");
	bw.setBeanInstance(instantiate(beanName, mbd, constructorToUse, argsToUse));
	return bw;
}
```











## org.springframework.beans



### BeanUtils



#### isSimpleProperty

判断bean的属性是否是简单类型

```java
public static boolean isSimpleProperty(Class<?> type) {
	Assert.notNull(type, "'type' must not be null");
  // 有两种情况，属性都属于简单类型：
  // ① 属性本身是简单类型
  // ② 属性是简单类型数组
	return isSimpleValueType(type) || (type.isArray() && isSimpleValueType(type.getComponentType()));
}
```





#### isSimpleValueType

判断一个Class是不是简单类型

```java
public static boolean isSimpleValueType(Class<?> type) {
	return (Void.class != type && void.class != type &&
      // ClassUtils会检查type是不是 Boolean、Character、Byte、Short、Integer、Long、Float
      // Double、Void
			(ClassUtils.isPrimitiveOrWrapper(type) ||
			Enum.class.isAssignableFrom(type) ||
			CharSequence.class.isAssignableFrom(type) ||
			Number.class.isAssignableFrom(type) ||
			Date.class.isAssignableFrom(type) ||
			Temporal.class.isAssignableFrom(type) ||
			URI.class == type ||
			URL.class == type ||
			Locale.class == type ||
			Class.class == type));
}
```













## 附录



### emojis

🌹🍀🍎💰📱🌙🍁🍂🍃🌷💎🔪🔫🏀⚽⚡👄👍🔥

💪👈👉☝👆👇✌✋👌👍👎✊👊👋👏👐✍

👣👀👂👃👅👄💋👓👔👕👖👗👘👙👚👛👜👝🎒💼👞👟👠👡👢👑👒🎩🎓💄💅💍🌂

📱📲📶📳📴☎📞📟📠

🌍🌎🌏🌐🌑🌒🌓🌔🌕🌖🌗🌘🌙🌚🌛🌜☀🌝🌞⭐🌟🌠☁⛅☔⚡❄🔥💧🌊

🍇🍈🍉🍊🍋🍌🍍🍎🍏🍐🍑🍒🍓🍅🍆🌽🍄🌰🍞🍖🍗🍔🍟🍕🍳🍲🍱🍘🍙🍚🍛🍜🍝🍠🍢🍣🍤🍥🍡🍦🍧🍨🍩🍪🎂🍰🍫🍬🍭🍮🍯🍼☕🍵🍶🍷🍸🍹🍺🍻🍴
🎪🎭🎨🎰🚣🛀🎫🏆⚽⚾🏀🏈🏉🎾🎱🎳⛳🎣🎽🎿🏂🏄🏇🏊🚴🚵🎯🎮🎲🎷🎸🎺🎻🎬
😈👿👹👺💀☠👻👽👾💣

📱📲☎📞📟📠🔋🔌💻💽💾💿📀🎥📺📷📹📼🔍🔎🔬🔭📡📔📕📖📗📘📙📚📓📃📜📄📰📑🔖💳✉📧📨📩📤📥📦📫📪📬📭📮✏✒📝📁📂📅📆📇📈📉📊📋📌📍📎📏📐✂🔒🔓🔏🔐🔑
⬆↗➡↘⬇↙⬅↖↕↔↩↪⤴⤵🔃🔄🔙🔚🔛🔜🔝