# Mybatis基础

Mybatis核心的功能有两块，1. 动态SQL；2. JDBC封装；



![img](http://img.blog.csdn.net/20141028140852531?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHVhbmxvdWlz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)





## 核心组件和概念

核心组件一览：

<a href="#">DataSourceFactory</a>：DataSource构建工厂，DataSource是获取一个数据库连接的规范接口，在Mybatis里面有多种实现，```JndiDataSource```（）、```PooledDataSource```（PooledDataSource是基于UnpooledDataSource来构建的，它会把UnpooledDataSource放到一个List里面，一个list当中存的是idleConnections，另一个list存的是activeConnections，当需要获取DataSource时候，会优先去idleConnections获取连接，然后把连接对象从idleConnections移除并在activeConnections添加这个连接）、```UnpooledDataSource```（使用username和password建立数据库连接，实际上就是调用 DriverManager.getConnection），这几种都是在mybatis配置文件当中直接配置 <dataSource type="POOLED"> 来做的，如果需要使用别的数据库连接池，则需要把新的DataSource对象设置到Environment当中去

<a href="#">TransactionFactory</a>：支持两种类型の事务管理器，JDBC / MANAGED，

<a href="#">Environment（DataSource, TransactionFactory）</a>：

<a href="#">Configuration</a>：

<a href="#">SqlSessionFactoryBuilder</a>：通过 Configuration 参数构建出 SqlSessionFactory 对象，作用仅仅就是构建器，没有其他作用，构建出 SqlSessionFactory对象之后，它就没用了，就该被销毁，退出舞台了

<a href="#">SqlSessionFactory</a>：创建 SqlSession，sql会话，相当于数据库连接（JDBC Connection），在应用中应该被设计成单例对象

<a href="#">SqlSession</a>：相当于和数据库的一个连接，和数据库服务端交互，提交SQL语句给服务器，接收服务器响应数据，是一个非线程安全对象，使用完毕需要手动关闭，否则，会浪费服务器的数据库连接。

 



<img src="/Users/dzsb-002004/Library/Application Support/typora-user-images/image-20200820120027197.png" alt="image-20200820120027197" style="zoom:40%;" />





## 配置



### 可配置项

配置项都包含在 <configuration>标签内

* <properties>
  * <property>
* <settings>
  * <setting>
* <typeAliases>
  * <typeAlias>
* <typeHandlers>
* <objectFactory>
* <plugins>
* <environments>
  * <environment>
    * <transactionManager>
    * <dataSource>
* <databaseIdProvider>
* <mappers>
  * <mapper>



### properties & property



XML配置

```xml
<!-- ① 直接设置property的name, value-->
<properties>
  <property name="xxx" value="xxx"/>
</properties>

<!-- ② 引入properties文件-->
<properties resource="jdbc.properties"/>
```



Java代码配置

```java
// import java.util.Properties;

FileInputStream in = new FileInputStream(confFile);
// ③ 设置 properties，等于XML文件配置 name 和 value
Properties properties = new Properties();
properties.setProperty("name", "value");
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in, properties);
```



配置方式优先级

Java代码 Properties设置 ```>``` resource/url 配置文件 ```>``` property标签 name value



> 建议：不使用混合方式，容易造成管理困难，建议使用 resource/url 指定从 properties 配置文件读取配置项和值



### settings & setting

大部分不需要配置，只需要配置少数几项



XML配置

```xml
<settings>
  <setting name="cacheEnable" value="true" />
  <setting name="lazyLoadingEnable" value="true" />
</settings>
```



setting name 的可选项有

|              name               |       描述       |  取值范围  | 默认值 |
| :-----------------------------: | :--------------: | :--------: | :----: |
|       ```cacheEnabled```        | 是否开启二级缓存 | true/false |  true  |
|    ```lazyLoadingEnabled```     |                  |            |        |
|   ```aggressiveLazyLoading```   |                  |            |        |
| ```multipleResultSetsEnabled``` |                  |            |        |
|      ```useColumnLabel```       |                  |            |        |
|     ```useGeneratedKeys```      |                  |            |        |
|    ```autoMappingBehavior```    |                  |            |        |
|    ```defaultExecutorType```    |                  |            |        |
|  ```defaultStatementTimeout```  |                  |            |        |
|   ```safeRowBoundsEnabled```    |                  |            |        |
| ```mapUnderscoreToCamelCase```  |                  |            |        |
|      ```localCacheScope```      |                  |            |        |
|      ```jdbcTypeForNull```      |                  |            |        |
|  ```lazyLoadTriggerMethods```   |                  |            |        |
| ```defaultScriptingLanguage```  |                  |            |        |
|    ```callSettersOnNulls```     |                  |            |        |
|         ```logPrefix```         |                  |            |        |
|          ```logImpl```          |                  |            |        |
|       ```proxyFactory```        |                  |            |        |





### typeAliases & typeAlias

用一个别名来代表一个类的全限定名，有mybatis内置别名和用户自定义别名两种。



XML自定义别名

```xml
<typeAliases>
  <typeAlias alias="myClass" type="com.xiaolee.xxx.xxx.xxx.xxx.MyClass" />
</typeAliases>
```



扫描包配置别名

```xml
<typeAliases>
  <package name="com.xiaolee.xxx.xxx.xxx.xxx" />
</typeAliases>
```



```java
package com.xiaolee.xxx.xxx.xxx.xxx;
@Alias("myClass")
public class MyClass {
  
} 
```





### typeHandlers & typeHandler

TypeHandler分为两种：mybatis自带的 和 我们自定义的



功能：可以对sql执行参数进行 java type -> jdbc type类型转换，也可以对sql执行结果进行 jdbc type -> java type转换。

<img src="https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/tuchuang/image-20200418115211914.png" alt="image-20200418115211914" style="zoom:50%;" />



配置：要想使用自定义的 typehandler，需要正确配置

```xml
<!-- ① 首先要在配置文件中注册自定义的TypeHandler -->
<typeHandlers>
	<typeHandler jdbcType="VARCHAR" javaType="string" handler="com.xiaolee.xxx.xxx.MyStringTypeHandler">
</typeHandlers>
  
  
  <mapper namespace="com.xiaolee.xxx.xxx.XXMapper">
    <resultMap type="xx" id="id_xx">
      <!-- ②-①指定结果集某一列属性的javaType和jdbcType，这样就可以匹配typeHandler -->
    	<result column="col_1" property="col1" javaType="string" jdbcType="VARCHAR" />
      
      <!-- ②-②直接设置结果集某一列属性的 typeHandler -->
      <result column="col_2" property="col2" typeHandler="com.xiaolee.xxx.xxx.MyStringTypeHandler">
    </resultMap>
    
    <>
    <select id="findXXX" parameterType="string" resultMap="id_xxx">
      select * from xx where col_1 = #{ col1 typeHandler=com.xiaolee.xxx.xxx.MyStringTypeHandler}
    </select>
  </mapper>
```





### ObjectFactory

mybatis使用 ObjectFactory 从数据库返回结果构建出结果POJO对象，





### environments & environment





### mappers & mapper





## XML解析器

mybatis用的是 XPath 来解析XML配置文件，还有一些别的工具可以用来解析XML，在此之前，要先清除解析XML文档，有两种不同类型的方案，也对应着两个接口规范：

* **DOM**：把整个XML文档读入内存，然后解析
* **SAX**： 流式解析，一点一点把XML读入内存进行解析

上面👆这两种只是不同的解析XML文档的接口而已，具体的具有有：

* 基于DOM方式的：```JDOM```、```DOM4j```

* 基于SAX方式的：```XPath```



![mybatis基础](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/mybatis基础.png)





## resultMapping

mybatis当中配置的resultMap标签被解析之后会对应生成一个ResultMap对象，ResultMap对象当中有具体数据库列和java对象属性之间的映射关系 ```List<ResultMapping> resultMappings```，ResultMapping对象当中实际保存了具体的映射信息：

```java
private Configuration configuration;
// 映射的java bean 成员属性
private String property;
// 数据库列名
private String column;
// 映射的java bean class，类的全限定名
private Class<?> javaType;
private JdbcType jdbcType;
private TypeHandler<?> typeHandler;
private String nestedResultMapId;
private String nestedQueryId;
private Set<String> notNullColumns;
private String columnPrefix;
private List<ResultFlag> flags;
private List<ResultMapping> composites;
private String resultSet;
private String foreignColumn;
private boolean lazy;
```





## 反射工具箱



> Reflector对象 <------> Class反射相关信息













## Spring+Mybatis



### 使用



#### 1. 引入必要依赖包

```xml

```















### 扫描mapper接口

mybatis两大核心初始化项目，一个是获取 Configuration，通过Configuration才能构建出 SqlSessionFactory进而创建出SqlSession，另外一个就是获取到 mapper 信息，具体来说就是知道有哪些 mapper接口



**1. 扫描mapper接口**

通过 @MapperScan 来指定扫描什么包，来扫描出 mapper 接口类

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
// 核心是引入了 MapperScannerRegistrar
@Import(MapperScannerRegistrar.class)
@Repeatable(MapperScans.class)
public @interface MapperScan {
}
```

而 MapperScannerRegistrar 这个 ImportBeanDefinitionRegistrar 只做了一件事情，就是向容器注册```MapperScannerConfigurer``` BeanDefinitionRegistryPostProcessor，MapperScannerConfigurer包含了 @MapperScan 中的所有配置信息，真正的mapper扫描逻辑是在 MapperScannerConfigurer 中开始的。

![image-20200420150907091](https://tuchuang-1256253537.cos.ap-shanghai.myqcloud.com/img/image-20200420150907091.png)

**2. 生成mapper接口对应的 bean definition**

mapper接口对应的 bean definition的 bean class默认是 ```MapperFactoryBean``` ，这是一个工厂类，一个mapper注册到spring容器中的bean definition实际上基本是这样：





**3. 调用 mapper 相应的 getBean 返回代理对象**







### 核心组件配置





#### transactionManager

