# Spring Security Web

web环境下Spring Security的作用原理基础是过滤器：

![Filter chain delegating to a Servlet](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/filters.png)

在一个java web应用中，一个请求的处理过程是这样的：

1. 客户端某个请求到达服务端
2. 服务端根据 **请求URL** 决定处理请求的Filter和Servlet（处理某个请求的多个Filter组成一个Filter链，Filter有顺序），Filter可以多个，Servlet只有单个
3. Filter处理请求，最后Servlet处理请求，产生响应



Spring Security就是这个Filter链中的一个Filter（叫 **FilterChainProxy**），所以一个使用了Spring Security框架的服务端应用，每个请求都会被SpringSecurity的Filter处理到，因为Spring Security的Filter处理的请求URL范围是全部。所以使用了Spring Security框架时，请求的处理像这样：

![Spring Security Filter](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters.png)

在FilterChainProxy内部，还会对请求进行转发，这是Spring Security的功能，属于Filter内再转发Filter

注意：FilterChainProxy内部可以定义多个FilterChain，FilterChainProxy会把请求分配给第一个匹配请求的FilterChain，在FilterChainProxy内部，一个请求只会被一个FilterChain处理。

![Security Filter Dispatch](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters-dispatch.png)

###自定义Spring Security中的Filter Chain

```WebSecurityConfigurerAdapter``` 相当于在FilterChainProxy中定义了一个新的FilterChain，这个FilterChain默认优先级在FilterChainProxy内部默认包含处理/**请求的FilterChain。













https://xugaoxiang.com/2020/01/02/trojan-installation/

https://github.com/FelisCatus/SwitchyOmega/releases







