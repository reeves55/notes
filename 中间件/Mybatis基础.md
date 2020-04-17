# Mybatis基础



## 核心组件和概念

核心组件一览：

<a href="#">DataSource</a>：

<a href="#">TransactionFactory</a>：

<a href="#">Environment</a>：

<a href="#">Configuration</a>：

<a href="#">SqlSessionFactoryBuilder</a>：通过 Configuration 参数构建出 SqlSessionFactory 对象，作用仅仅就是构建器，没有其他作用，构建出 SqlSessionFactory对象之后，它就没用了，就该被销毁，退出舞台了

<a href="#">SqlSessionFactory</a>：创建 SqlSession，sql会话，相当于数据库连接（JDBC Connection），在应用中应该被设计成单例对象

<a href="#">SqlSession</a>：相当于和数据库的一个连接，和数据库服务端交互，提交SQL语句给服务器，接收服务器响应数据，是一个非线程安全对象，使用完毕需要手动关闭，否则，会浪费服务器的数据库连接。

 



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
@Alias()
public class MyClass {
  
} 
```





### typeHandlers & typeHandler

typeHandler可以自定义 jdbcType 、 javaType 转换处理器，



### ObjectFactory

mybatis使用 ObjectFactory 从数据库返回结果构建出结果POJO对象，





### environments & environment





### mappers & mapper





## mapper映射器





