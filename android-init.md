## Android 系统启动

### 1. 概述

Android系统底层基于Linux Kernel, 当Kernel启动过程会创建init进程, 该进程是所有用户空间的鼻祖, init进程会启动servicemanager(binder服务管家), zygote进程(java进程的鼻祖). zygote进程会创建 system_server进程以及各种app进程。下图是这几个进程之间的关系。

![android-booting](http://gityuan.com/images/android-arch/android-booting.jpg)

### 2. init 进程

init 是Linux系统中用户空间的第一个进程(pid=1), Kernel启动后会调用/system/core/init/Init.cpp的main()方法。

### 3. zygote 进程

当 zygote 进程启动后, 便会执行到frameworks/base/cmds/app_process/App_main.cpp文件的main()方法。

### 4. servicemanager 进程

ServiceManager是Binder IPC通信过程中的守护进程，本身也是一个Binder服务，但并没有采用libbinder中的多线程模型来与Binder驱动通信，而是自行编写了binder.c直接和Binder驱动来通信，并且只有一个循环binder_loop来进行读取和处理事务，这样的好处是简单而高效。

### 5. system_server 进程

zygote通过fork后创建system_server进程。该进程承载着framework的核心服务。zygote启动过程中会调用startSystemServer()，该函数就是 system_server 启动流程的起点。

system_server进程承载的服务类别，从源码角度划分为引导服务、核心服务、其他服务3类。 

1. 引导服务(7个)：ActivityManagerService、PowerManagerService、LightsService、DisplayManagerService、PackageManagerService、UserManagerService、SensorService；
2. 核心服务(3个)：BatteryService、UsageStatsService、WebViewUpdateService；
3. 其他服务(70个+)：AlarmManagerService、VibratorService等。

### 6. app 进程

对于普通的app进程,跟system_server进程的启动过来有些类似。不同的是app进程是向发消息给system_server进程, 由system_server向zygote发出创建进程的请求。进程创建后，接下来会进入ActivityThread.main()过程。

进程创建过程如下图所示：

![start_app_process](http://gityuan.com/images/android-process/start_app_process.jpg)

app 进程调用栈

```java
  ...
    at android.app.ActivityThread.main(ActivityThread.java:5442)
    at java.lang.reflect.Method.invoke!(Native method)
    at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:738)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:628)
```

### 7. 守护进程

守护进程(也就是进程名一般以d为后缀，比如logd，此处d是指daemon的简称)

- debuggerd

  Android系统有监控程序异常退出的机制，这便是debuggerd守护进程。当发生native crash或者主动调用debuggerd时，会输出进程相关的状态信息到文件或者控制台。

- installd

  installd是由Android系统的init进程(pid=1)，通过fork创建的用户空间的守护进程installd，负责应用的安装、卸载等相关工作。

- lmkd

  lmk，全称为 LowMemoryKiller，负责决定什么时间杀掉什么进程。lmkd 每隔一段时间检查一次，当系统剩余可用内存较低时，便会触发杀进程的策略，根据不同的剩余内存档位来来选择杀不同优先级的进程。lmkd是由init进程fork的守护进程。

- logd

  Androd上层采用logcat输出log，上层的 log 最终通过调用 native 的 write()向 logd 端进程写入要打印的日志信息。logd 也是由 init 进程 fork 的守护进程。

  

  