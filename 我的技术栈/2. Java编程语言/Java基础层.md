# Java基础层



## 反射

通过 class 对象，拿到类定义的相关信息，包括 字段、方法、类型 等问题。





#### Class对象方法



>  ```Constructor<?>[] getDeclaredConstructors() throws SecurityException```

拿到一个类的所有构造方法，不论是 public、protect、private，都可以获取到







#### Constructor对象方法



> ```Class<?>[] getParameterTypes()```

获取构造方法参数类型









#### SecurityManager

权限分为以下类别：文件、套接字、网络、安全性、运行时、属性、AWT、反射和可序列化。管理各种权限类别的类是java.io.FilePermission、java.net.SocketPermission、java.net.NetPermission、java.security.SecurityPermission、java.lang.RuntimePermission、java.util.PropertyPermission、java.awt.AWTPermission、java.lang.reflect.ReflectPermission和 java.io.SerializablePermission。





## XML文件解析



