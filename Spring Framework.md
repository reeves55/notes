# Spring Framework

```SpringApplication``` 这个类相当于一个执行器，负责把 ```ApplicationContext```创建并初始化好。





## 一个请求的处理过程



##### 关键点1：HandlerMapping

```HandlerMapping``` 当中记录了请求和处理该请求的Bean和相应方法的映射。



#####关键点2：HandlerMethodArgumentResolver

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



##### 关键点3：MessageConverter



参数解析器 ```HttpMessageConverter``` 的10种类型：

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200323144702260.png" alt="image-20200323144702260" style="zoom:50%;float:left" />



##### 关键点4：HandlerMethodReturnValueHandler

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



