## Android 系统稳定性

### 1. 概述

Android系统稳定性对于用户体验至关重要，从技术层面来划分分为两大类：长时间无法执行完成（Timeout）以及异常崩溃（Crash），主要分类如下：

![stability_summary.jpg](http://gityuan.com/images/stability/stability_summary.jpg)

### 2. Timeout

Timeout 分为两类，一种是 ANR，一种是 WatchDog。

对于 ANR，通常有以下几种情况会发生：

- Service Timeout:  比如前台服务在20s内未执行完成
- Broadcast Timeout：比如前台广播在10s内未执行完成
- ContentProvider Timeout：内容提供者执行超时
- InputDispatching Timeout: 输入事件分发超时5s，包括按键和触摸事件

对于 WatchDog，最为常见的是运行在 system_server 中的 watchdog 线程，还有运行在各个 app 进程中的 FinalizerWatchdogDaemon 线程，该线程用于监控执行 gc 的过程中回收某个对象时间过长。还有 dex2oat，wifi 等 watchdog。WatchDog功能主要是分析系统核心服务和重要线程是否处于Blocked状态。

### 3. Crash

对于Java层Crash，往往是抛出了一个未捕获的异常UncaughtException而导致的崩溃。

对于Native层Crash，则是由于进程收到signal信号而引发的崩溃。Native crash情况比较多, 其中最为场景便是SIGSEGV段错误异常, 往往是内存出现异常,比如访问了权限不足的内存地址等。

对于Kernel层Crash，这是比较难分析的一大类, 不少情况是跟硬件导致的,比较cpu问题, 硬件驱动问题等。