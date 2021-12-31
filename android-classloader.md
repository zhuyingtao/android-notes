## ClassLoader

类加载器（ClassLoader），主要作用是查找和加载 class 文件到 java 虚拟机中。

#### 类加载过程

分为五个过程：加载、验证、准备、解析、初始化

- 加载

  将外部的 class 文件加载到 Java 虚拟机并存储到方法区内。

  在内存中会生成一个代表这个类的 Class 对象，作为方法区该类的各种数据的访问入口

- 验证

  确保加载进来的 Class 文件包含的信息符合 Java 虚拟机的要求

- 准备

  为 static 变量分配内存&设置初始值（非开发者定义的值，除非是常量）

- 解析

  将常量池的符号引用转化为直接引用

- 初始化

  初始化 static 变量和 static 代码块

#### 类加载器类别

![img](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xNTUwMjFhNWI4MzBjZjQ4LnBuZw?x-oss-process=image/format,png)

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

##### PathClassLoader 与 DexClassLoader 区别

PathClassLoader 和 DexClassLoader 都是继承了 BaseDexClassLoader，PathClassLoader 和 DexClassLoader **都能加载外部的 dex／apk**，只不过区别是 DexClassLoader 可以**指定 optimizedDirectory**，也就是 dex2oat 的产物 .odex 存放的位置，而 PathClassLoader 只能使用系统默认位置。但是这个 optimizedDirectory 在 Android 8.0 以后也被舍弃了，只能使用系统默认的位置了。

#### 类加载器原理

双亲委派模式

![image-20190815234524980](/Users/zyt/Library/Application Support/typora-user-images/image-20190815234524980.png)





