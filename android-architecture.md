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

