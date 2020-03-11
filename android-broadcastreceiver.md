## Android 广播机制

### 概述

广播机制用于进程/线程间通信，广播分为广播发送和广播接收两个过程，其中广播接收者 BroadcastReceiver 就是 Android 四大组件之一

BrocadcastReceiver 注册方式分为两类：

- 静态注册：通过 AndroidManifest.xml 里的标签来注册明
- 动态注册：通过 registerReceiver()来注册

从发送方式分为三类：

- 普通广播：通过Context.sendBroadcast()发送，可并行处理
- 有序广播：通过Context.sendOrderedBroadcast()发送，串行处理
- Sticky广播：通过Context.sendStickyBroadcast()发送（已废弃）

广播在系统中以BroadcastRecord对象来记录, 该对象有几个时间相关的成员变量。

### 安全性

广播安全性问题

- 其他App可能会针对性的发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收到广播并处理；

- 其他App可以注册与当前App一致的intent-filter用于接收广播，获取广播具体信息。

增加广播安全性的措施

- 对于同一App内部发送和接收广播，将exported属性人为设置成false，使得非本App内部发出的此广播不被接收；

- 在广播发送和接收时，都增加上相应的permission，用于权限验证；

- 发送广播时，指定特定广播接收器所在的包名，具体是通过intent.setPackage(packageName)指定，这样此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

- 采用LocalBroadcastManager的方式

### Android 8.0

Android 8.0后，即当App targetSDK >= 26，几乎禁止了所有的**隐式广播**的静态注册监听。

除了有限的例外之外，应用无法使用清单注册（**静态注册**）的方式来接收**隐式广播**。

- 但对于这些隐式广播，可以通过运行时注册（**动态注册**）的方式注册。
- 对于**显式广播**，则依然可以通过清单注册（静态注册）的方式监听