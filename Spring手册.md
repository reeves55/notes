# Spring手册





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