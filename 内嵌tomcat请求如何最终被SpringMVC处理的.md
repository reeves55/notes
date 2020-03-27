# 内嵌tomcat请求如何最终被SpringMVC处理的



SpringMVC的入口是 ```DispatcherServlet```，它是一个标准的HttpServlet，Tomcat把请求作为参数调用DispatcherServlet中的service方法，那Tomcat给DispatcherServlet传给的请求参数是什么呢？是RequestFacade类型的对象：

![RequestFacade](/Users/reeves/Downloads/RequestFacade.png)



