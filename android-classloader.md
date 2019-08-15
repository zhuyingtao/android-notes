## ClassLoader

类加载器（ClassLoader），主要作用是查找和加载 class 文件到 java 虚拟机中。

#### Java 类加载器

Java 中的类加载器主要分为两类：

- 系统类加载器
  - Bootstrap 类加载器
  - Extensions 类加载器
  - Application 类加载器
- 自定义类加载器

1. Bootstrap 类加载器

   C/C++ 代码实现的类加载器，用于加载 java 虚拟机运行时所需要的 class 文件，如`java.lang.*`，`java.util.*`，这些类默认放置在`$JAVA_HOME/jre/lib`目录中。虚拟机的启动就是依靠 bootstrap 类加载器创建一个初始类来完成的。

2. Extensions 类加载器

   Java 代码实现的类加载器，用于加载 java 的扩展类，这些类默认放置在`$JAVA_HOME/jre/lib/ext`目录中

3. Application 类加载器

   加载当前应用程序classpath 目录下的 jar 和 class 文件。

4. Custom 类加载器

#### 类加载器原理

双亲委托模式

![image-20190815234524980](/Users/zyt/Library/Application Support/typora-user-images/image-20190815234524980.png)



#### Android 类加载器

Android 类加载器与 Java 类加载器并不一样，因为 android 类加载器加载的是 dex 文件，所以需要重新设计相关 ClassLoader 类。

Android 类加载器也分为两类：

- 系统类加载器
  - BootClassLoader
  - PathClassLoader
  - DexClassLoader
- 自定义类记载器

1. BootClassLoader

   Java 代码实现的类加载器

2. DexClassLoader

   加载 dex 文件以及包含 dex 的压缩文件

3. PathClassLoader

   加载系统类和应用程序的类

4. CustomClassLoader