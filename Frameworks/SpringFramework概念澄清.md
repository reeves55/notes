# SpringFramework概念澄清





> Spring是什么❓

Spring框架核心包含了两个特性，IoC（Inverse of Control） 和 AoP（Aspect-Oriented Programming），IoC属于工厂方法的一种实现，Spring在启动时，会创建一个BeanFactory类型的对象，保存着所有Spring托管的用户对象，通过BeanFactory的getBean方法，就能拿到我们需要的已经初始化完成的对象。AoP，也就是面向切面编程，面向切面编程可以给多个方法同时拓展出相同的逻辑功能，把原来需要同时多处维护的相同代码集中到一起，简化了管理。



> 为什么Spring会兴起❓

Spring的兴起是为了替代EJB，Rod Johnson在2002年编著的《Expert one on one J2EE design and development》一书中批判了那个时候Java EE 系统框架，说其臃肿、低效、脱离现实，同年推出了《Expert one-on-one J2EE Development without EJB》，对EJB的各种笨重臃肿的结构进行了逐一的分析和否定，并分别以简洁实用的方式替换之。2003年起，由《Expert one on one J2EE design and development》中阐述的部分理念和原型衍生而来的一个轻量级Java开发框架--Spring便兴起；其目的就是为了简化JavaEE的企业级应用开发。

EJB（Enterprise JavaBean）技术是什么呢，EJB技术由EJB容器（实现了EJB标准的容器）、EJB接口对象、RMI三个关键要素组成，开发人员编写EJB接口对象，并把要部署的包放到EJB容器中，由EJB容器管理这些EJB的生存周期和处理远程调用，客户端使用RMI协议调用这些EJB对象的方法。

