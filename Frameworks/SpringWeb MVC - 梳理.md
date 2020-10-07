# SpringWeb MVC



SpringWeb MVC是Spring框架应用上的一个拓展，封装了基于servlet的web程序框架，在Spring框架启动完成之后会进行SpringWeb MVC的初始化过程，SpringWeb MVC会从Spring ApplicationContext中获取Web MVC相关的bean或者配置信息。



**SpringFramework + Tomcat + SpringMVC**

Tomcat启动 ```=>``` SpringFramework启动（tomcat listener触发） ```=>``` SpringMVC注册到Tomcat（tomcat servlet配置）```=>``` 第一次请求到来 ```=>``` SpringMVC初始化（从ServletContext中获取ApplicationContext）```=>``` 处理请求



**Springboot**

SpringFramework启动 ```=>``` 创建Tomcat（注册SpringMVC DispatcherServlet） ```=>``` 第一次请求到来 ```=>``` SpringMVC初始化（从ServletContext中获取ApplicationContext）```=>``` 处理请求 



> **SpringWeb MVC初始化**

初始化动作在第一次请求到达的时候进行，“懒初始化”。初始化操作发生在 ```DispatcherServlet```，初始化操作会 <span style="color:red">从Spring容器中获取以下类型的bean组件</span>：



* ```MultipartResolver```
* ```LocaleResolver```
* ```ThemeResolver```
* ```HandlerMapping```
* ```HandlerAdapter```
* ```HandlerExceptionResolver```
* ```RequestToViewNameTranslator```
* ```ViewResolver```
* ```FlashMapManager```



```java
/**
 * This implementation calls {@link #initStrategies}.
 */
@Override
protected void onRefresh(ApplicationContext context) {
	initStrategies(context);
}

/**
 * Initialize the strategy objects that this servlet uses.
 * <p>May be overridden in subclasses in order to initialize further strategy objects.
 */
protected void initStrategies(ApplicationContext context) {
  // 初始化multipartResolver：
  // this.multipartResolver = context.getBean(MULTIPART_RESOLVER_BEAN_NAME, MultipartResolver.class);
  // 
	initMultipartResolver(context);
  
	initLocaleResolver(context);
  
	initThemeResolver(context);
  
	initHandlerMappings(context);
  
	initHandlerAdapters(context);
  
	initHandlerExceptionResolvers(context);
  
	initRequestToViewNameTranslator(context);
  
	initViewResolvers(context);
  
	initFlashMapManager(context);
}
```





## 核心组件



<span style="color:#41B883">**DispatcherServlet**</span>：Spring和Servlet容器关联的核心类，Servlet接收到的匹配请求都会给到SpringMVC中的这个DispatcherServlet，由它负责把请求分发给匹配的处理器；

<span style="color:#41B883">**HandlerMapping**</span>：保存着 ```HttpServletRequest``` =>  ```HandlerExecutionChain```，请求和请求执行器之间的映射关系，SpringMVC当中存在多种HandlerMapping，用户也可以注册自己的HandlerMapping，自定义根据请求当中的某些信息，匹配出请求执行器；

<span style="color:#41B883">**HandlerExecutionChain**</span>：请求执行器，它有一个最最重要的成员变量，就是 ```Object handler```，handler是真正处理请求的；

<span style="color:#41B883">**HandlerAdapter**</span>：handler的适配器，为什么需要适配器呢，因为请求处理器handler可以是任意类型的（HandlerExecutionChain保存的处理器实例是Object类型的），那如何调用处理器来处理请求呢，HandlerAdapter就是这个时候发挥作用的，任何一种handler都必须存在适配它的HandlerAdapter，这个时候handler就相当于是adapter的“代理”，SpringMVC框架调用HandlerAdapter来处理请求，而adapter就把处理过程交给其适配的handler。





### DispatcherServlet



DispatcherServlet是spring和servlet容器连接的关键，当然也是需要一定的 “操作”，才能让 servlet容器使用DispatcherServlet，tomcat当中典型的配置如下：



```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </context-param>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

    <!-- 前端控制器配置 -->
    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        　<init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath*:spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        　　　　<url-pattern>/</url-pattern>
    </servlet-mapping>

		<!-- 启动tomcat时启动spring容器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app >
```



Sprintboot当中，Tomcat容器是用编码的方式启动的，在启动Tomcat过程中，向容器默认注册 DispatcherServlet



```java
/**
 * 创建Tomcat容器
 */
public WebServer getWebServer(ServletContextInitializer... initializers) {
	if (this.disableMBeanRegistry) {
		Registry.disableRegistry();
	}
	Tomcat tomcat = new Tomcat();
	File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
	tomcat.setBaseDir(baseDir.getAbsolutePath());
	Connector connector = new Connector(this.protocol);
	connector.setThrowOnFailure(true);
	tomcat.getService().addConnector(connector);
	customizeConnector(connector);
	tomcat.setConnector(connector);
	tomcat.getHost().setAutoDeploy(false);
	configureEngine(tomcat.getEngine());
	for (Connector additionalConnector : this.additionalTomcatConnectors) {
		tomcat.getService().addConnector(additionalConnector);
	}
  
  // 容器初始化
	prepareContext(tomcat.getHost(), initializers);
	return getTomcatWebServer(tomcat);
}

/**
 * 初始化Tomcat容器
 */
protected void prepareContext(Host host, ServletContextInitializer[] initializers) {
	File documentRoot = getValidDocumentRoot();
	TomcatEmbeddedContext context = new TomcatEmbeddedContext();
	if (documentRoot != null) {
		context.setResources(new LoaderHidingResourceRoot(context));
	}
	context.setName(getContextPath());
	context.setDisplayName(getDisplayName());
	context.setPath(getContextPath());
	File docBase = (documentRoot != null) ? documentRoot : createTempDir("tomcat-docbase");
	context.setDocBase(docBase.getAbsolutePath());
	context.addLifecycleListener(new FixContextListener());
	context.setParentClassLoader((this.resourceLoader != null) ? this.resourceLoader.getClassLoader()
			: ClassUtils.getDefaultClassLoader());
	resetDefaultLocaleMapping(context);
	addLocaleMappings(context);
	context.setUseRelativeRedirects(false);
	try {
		context.setCreateUploadTargets(true);
	}
	catch (NoSuchMethodError ex) {
		// Tomcat is < 8.5.39. Continue.
	}
	configureTldSkipPatterns(context);
	WebappLoader loader = new WebappLoader(context.getParentClassLoader());
	loader.setLoaderClass(TomcatEmbeddedWebappClassLoader.class.getName());
	loader.setDelegate(true);
	context.setLoader(loader);
  
	if (isRegisterDefaultServlet()) {
   	// 在这里添加SpringMVC的默认DispatcherServlet注册到Tomcat容器中
		addDefaultServlet(context);
	}
  
	if (shouldRegisterJspServlet()) {
		addJspServlet(context);
		addJasperInitializer(context);
	}
	context.addLifecycleListener(new StaticResourceConfigurer(context));
	ServletContextInitializer[] initializersToUse = mergeInitializers(initializers);
	host.addChild(context);
	configureContext(context, initializersToUse);
	postProcessContext(context);
}


/**
 * 将DispatcherServlet注册到Tomcat容器中
 */
private void addDefaultServlet(Context context) {
	Wrapper defaultServlet = context.createWrapper();
	defaultServlet.setName("default");
	defaultServlet.setServletClass("org.apache.catalina.servlets.DefaultServlet");
	defaultServlet.addInitParameter("debug", "0");
	defaultServlet.addInitParameter("listings", "false");
	defaultServlet.setLoadOnStartup(1);
	// Otherwise the default location of a Spring DispatcherServlet cannot be set
	defaultServlet.setOverridable(true);
	context.addChild(defaultServlet);
	context.addServletMappingDecoded("/", "default");
}
```





<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200516105130222.png" alt="image-20200516105130222" style="zoom:50%;" />





### HanderMapping

不同的HandlerMapping根据不同规则，解析HttpRequest，然后找到相应处理该请求的 Handler

SpringWeb MVC内置了许多HandlerMapping的实现：





> **```RequestMappingHandlerMapping```**



存储映射信息的类 ```MappingRegistry```

















### HandlerAdapter

HandlerAdapter的任务就是 <span style="color:#41B883">解析请求中的参数，然后调用真正的handler处理方法，获取handler处理的返回值之后，对返回值进行解析，封装成ModelAndView，并返回</span>。



所以，其实有三个重点：

1. <span style="color:red">解析请求中的参数；</span>
2. <span style="color:red">调用handler方法；</span>
3. <span style="color:red">解析handler方法的返回值；</span>



接口定义：

```java
public interface HandlerAdapter {

	/**
	 * Given a handler instance, return whether or not this {@code HandlerAdapter}
	 * can support it. Typical HandlerAdapters will base the decision on the handler
	 * type. HandlerAdapters will usually only support one handler type each.
	 * <p>A typical implementation:
	 * <p>{@code
	 * return (handler instanceof MyHandler);
	 * }
	 * @param handler handler object to check
	 * @return whether or not this object can use the given handler
	 */
	boolean supports(Object handler);

	/**
	 * Use the given handler to handle this request.
	 * The workflow that is required may vary widely.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler handler to use. This object must have previously been passed
	 * to the {@code supports} method of this interface, which must have
	 * returned {@code true}.
	 * @throws Exception in case of errors
	 * @return a ModelAndView object with the name of the view and the required
	 * model data, or {@code null} if the request has been handled directly
	 */
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	/**
	 * Same contract as for HttpServlet's {@code getLastModified} method.
	 * Can simply return -1 if there's no support in the handler class.
	 * @param request current HTTP request
	 * @param handler handler to use
	 * @return the lastModified value for the given handler
	 * @see javax.servlet.http.HttpServlet#getLastModified
	 * @see org.springframework.web.servlet.mvc.LastModified#getLastModified
	 */
	long getLastModified(HttpServletRequest request, Object handler);

}
```





> **默认 - ```RequestMappingHandlerAdapter```**







HandlerAdapter内部处理流程













> **解析请求中的参数 - ```HandlerMethodArgumentResolver```**





```java
public interface HandlerMethodArgumentResolver {

	/**
	 * Whether the given {@linkplain MethodParameter method parameter} is
	 * supported by this resolver.
	 * @param parameter the method parameter to check
	 * @return {@code true} if this resolver supports the supplied parameter;
	 * {@code false} otherwise
	 */
	boolean supportsParameter(MethodParameter parameter);

	/**
	 * Resolves a method parameter into an argument value from a given request.
	 * A {@link ModelAndViewContainer} provides access to the model for the
	 * request. A {@link WebDataBinderFactory} provides a way to create
	 * a {@link WebDataBinder} instance when needed for data binding and
	 * type conversion purposes.
	 * @param parameter the method parameter to resolve. This parameter must
	 * have previously been passed to {@link #supportsParameter} which must
	 * have returned {@code true}.
	 * @param mavContainer the ModelAndViewContainer for the current request
	 * @param webRequest the current request
	 * @param binderFactory a factory for creating {@link WebDataBinder} instances
	 * @return the resolved argument value, or {@code null} if not resolvable
	 * @throws Exception in case of errors with the preparation of argument values
	 */
	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```





常用的 HandlerMethodArgumentResolver











> **解析handler返回值 - ```HandlerMethodReturnValueHandler```**









> **处理请求的Handler中的一种 - ```HandlerMethod```**













## 请求处理核心流程



![IMG_1BD087C5761B-1](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/IMG_1BD087C5761B-1.jpeg)









### DataBinder



组件导出API功能：

* **将 ```PropertyValues``` 绑定到对象属性上**

  可以设置对象哪些field允许绑定（在web环境当中，客户端可以向服务端提交页面上没有的参数，如果这个参数恰好可以绑定到request model中的某个特别属性，那就可能出问题，设置allowedFields可以避免这一问题），哪些field必须提供绑定的值

* **Validation**

  绑定结束，可以对绑定结果进行validate，DataBinder支持 addValidators ，验证时，会遍历所有的validator，进行 validator.validate 

* **返回属性绑定结果 ```BindingResult```**

  







核心导出方法：

```java
public void bind(PropertyValues pvs) {
	MutablePropertyValues mpvs = (pvs instanceof MutablePropertyValues ?
			(MutablePropertyValues) pvs : new MutablePropertyValues(pvs));
	doBind(mpvs);
}

protected void doBind(MutablePropertyValues mpvs) {
	checkAllowedFields(mpvs);
	checkRequiredFields(mpvs);
	applyPropertyValues(mpvs);
}
```













## SpringWeb MVC AutoConfig

SpringWeb MVC的自动配置指的是，自动发现并注册 SpringWeb MVC当中所用到的关键组件，包括：HandlerMapping、HandlerAdapter、ViewResolver、LocaleResolver、Filter等等（和上面👆 DispatcherServlet的属性有关）。



不做任何配置的 springweb mvc其实是没有任何作用的（找不到处理 http request 的 HandlerExecutionChain，因为没有 handlerMapping），而springweb mvc自带了一个配置类 ```WebMvcConfigurationSupport``` ，默认注册了很多springweb mvc的组件，启用这个配置类，需要在入口的配置类加上 ```@EnableWebMvc``` 注解。











## 引用：



<span style="color:red">红色</span>

<span style="color:#41B883">主题色</span>