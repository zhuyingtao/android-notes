### Android 架构

Google 官方提供的经典 5 层架构图

从下到上分别是 Linux内核、硬件抽象层、系统 Native 库和 ART、Java 框架层、应用层。

![android-stack](http://gityuan.com/images/android-arch/android-stack.png)

#### 系统启动架构图

![img](http://gityuan.com/images/android-arch/android-boot.jpg)

Android系统启动过程由上图从下往上的一个过程是由Boot Loader引导开机，然后依次进入 -> `Kernel` -> `Native` -> `Framework` -> `App`

#### JNI

Android 层与 Linux kernel 层的枢纽是 Syscall（系统调用），Android 中 Java 层与 Native 层的纽带是 JNI。

JNI注册的两种时机：

- Android系统启动过程中Zygote注册，可通过查询AndroidRuntime.cpp中的gRegJNI，看看是否存在对应的register方法；
- 调用System.loadLibrary()方式注册。