- [Google官方架构示例](https://github.com/googlesamples/android-architecture)
- [Android 应用架构](http://www.jianshu.com/p/8ca27934c6e6)
- [一种更清晰的 Android 架构](https://github.com/hehonghui/android-tech-frontier/tree/master/androidweekly/%E4%B8%80%E7%A7%8D%E6%9B%B4%E6%B8%85%E6%99%B0%E7%9A%84Android%E6%9E%B6%E6%9E%84)


- [Android MVP 详解（上）](http://www.jianshu.com/p/9a6845b26856)
- [Android MVP 详解（下）](http://www.jianshu.com/p/0590f530c617)
- [MVP模式在Android开发中的应用](http://blog.csdn.net/vector_yi/article/details/24719873)


### android-MVP

### 1. MVP概述

#### 1.1 MVP—是什么

MVP，全称 Model-View-Presenter，是MVC（Model-View-Controller，模型-视图-控制器）模式的一个变种，主要用来隔离UI、UI逻辑和业务逻辑、数据。也就是说，MVP 是从经典的模式MVC演变而来，它们的基本思想有相通的地方：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示。

#### 1.2 MVP—为什么

在软件复杂度增长，需求不断变更的客观条件下，为了更好的解决这些问题，出现了各种软件架构思想、编程思想以及设计模式。（因为人的能力并没有“跟上”机器，所以才会出现各种模式、方法、工具等等来补足人的不足，以最大地透支机器性能。--Indream Luo）

GUI应用程序的出现导致了MVC的产生。在开发GUI应用程序的时候，会把管理用户界面的层次称为View，应用程序的数据为Model。有了View和Model的分层，那么问题就来了：View如何同步Model的变更，View和Model之间如何粘合在一起。所谓的MVX中的X都可以归纳为对这个问题不同的处理方式。

#### 1.3 MVP—优缺点

优点如下：

1. 降低耦合度，实现了Model和View真正的完全分离，可以修改View而不影响Model
2. 模块职责划分明显，层次清晰
3. 隐藏数据
4. Presenter可以复用，一个Presenter可以用于多个View，而不需要更改Presenter的逻辑
5. 利于测试驱动开发
6. View可以进行组件化
7. 代码灵活性

缺点如下：

1. Presenter中除了应用逻辑以外，还有大量的View->Model，Model->View的手动同步逻辑，造成Presenter比较笨重，维护起来会比较困难
2. 由于对View的渲染放在了Presenter中，所以View和Presenter的交互会过于频繁
3. 额外的代码复杂度及学习成本


> MVP模式是MVC模式在Android上的一种变体，要介绍MVP就得先介绍MVC。在MVC模式中，Activity应该是属于View这一层。而实质上，它既承担了View，同时也包含一些Controller的东西在里面。这对于开发与维护来说不太友好，耦合度大高了。把Activity的View和Controller抽离出来就变成了View和Presenter，这就是MVP模式。



#### MVC模式

MVC模式的结构分为三部分，实体层的Model，视图层的View，以及控制层的Controller。

![img](https://segmentfault.com/image?src=http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/13126761.jpg&objectId=1190000003927200&token=9cdd1d129e9862fa016f2c48560187c9/view)

- 其中View层其实就是程序的UI界面，用于向用户展示数据以及接收用户的输入
- 而Model层就是JavaBean实体类，用于保存实例数据
- Controller控制器用于更新UI界面和数据实例

#### MVP模式

MVP把Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口，Model类还是原来的Model。

![img](https://segmentfault.com/image?src=http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/2114527.jpg&objectId=1190000003927200&token=090ab9129b52d861300a716ee4d9180c/view)

从上图可以看出，Presenter是Model和View之间的桥梁，为了让结构变得更加简单，View并不能直接对Model进行操作，这也是MVP与MVC最大的不同之处。

MVP优点：

- 分离了视图逻辑和业务逻辑，降低了耦合
- Activity只处理生命周期的任务，代码变得更加简洁
- 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高代码的可阅读性
- Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试
- 把业务逻辑抽到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM


MVP模式的最主要优势就是耦合降低, Presenter变为纯Java的代码逻辑, 不再与Android Framework中的类如Activity, Fragment等关联, 便于写单元测试。


### TODO-MVP

[Android官方MVP架构项目解析](http://www.jianshu.com/p/389c9ae1a82c)

[Google 官方 MVP Sample 代码解读](https://juejin.im/entry/581067dcc4c9710058a86df5)

[Google官方架构MVP解析与实战](https://www.jianshu.com/p/569ab68da482)

[AndroidArchitectureCollection	](https://github.com/CameloeAnthony/AndroidArchitectureCollection)		