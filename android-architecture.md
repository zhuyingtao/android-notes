- [Google官方架构示例](https://github.com/googlesamples/android-architecture)
- [Android 应用架构](http://www.jianshu.com/p/8ca27934c6e6)
- [一种更清晰的 Android 架构](https://github.com/hehonghui/android-tech-frontier/tree/master/androidweekly/%E4%B8%80%E7%A7%8D%E6%9B%B4%E6%B8%85%E6%99%B0%E7%9A%84Android%E6%9E%B6%E6%9E%84)


- [Android MVP 详解（上）](http://www.jianshu.com/p/9a6845b26856)
- [Android MVP 详解（下）](http://www.jianshu.com/p/0590f530c617)
- [MVP模式在Android开发中的应用](http://blog.csdn.net/vector_yi/article/details/24719873)
- [如何构建Android MVVM 应用框架](https://tech.meituan.com/android_mvvm.html)


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

#### MVVM

MVC -> MVP -> MVVM 这几个软件设计模式是一步步演化发展的，MVVM 是从 MVP 的进一步发展与规范，MVP 隔离了MVC中的 M 与 V 的直接联系后，靠 Presenter 来中转，所以使用 MVP 时 P 是直接调用 View 的接口来实现对视图的操作的，这个 View 接口的东西一般来说是 showData、showLoading等等。M 与 V已经隔离了，方便测试了，但代码还不够优雅简洁，所以 MVVM 就弥补了这些缺陷。在 MVVM 中就出现的 Data Binding 这个概念，意思就是 View 接口的 showData 这些实现方法可以不写了，通过 Binding 来实现。

#### MVC/MVP/MVVM

##### 相同点

如果把这三者放在一起比较，先说一下三者的共同点，也就是Model和View：

- Model：数据对象，同时，提供本应用外部对应用程序数据的操作的接口，也可能在数据变化时发出变更通知。**Model不依赖于View的实现**，只要外部程序调用Model的接口就能够实现对数据的增删改查。
- View：UI层，提供对最终用户的交互操作功能，包括UI展现代码及一些相关的界面逻辑代码。

##### 不同点

三者的差异在于如何粘合View和Model，实现用户的交互操作以及变更通知

- Controller

Controller接收View的操作事件，根据事件不同，或者调用Model的接口进行数据操作，或者进行View的跳转，从而也意味着一个Controller可以对应多个View。Controller对View的实现不太关心，只会被动地接收，Model的数据变更不通过Controller直接通知View，通常View采用观察者模式监听Model的变化。

- Presenter

Presenter与Controller一样，接收View的命令，对Model进行操作；与Controller不同的是Presenter会反作用于View，Model的变更通知首先被Presenter获得，然后Presenter再去更新View。一个Presenter只对应于一个View。根据Presenter和View对逻辑代码分担的程度不同，这种模式又有两种情况：Passive View和Supervisor Controller。

- ViewModel

注意这里的“Model”指的是View的Model，跟MVVM中的一个Model不是一回事。所谓View的Model就是包含View的一些数据属性和操作的这么一个东东，这种模式的关键技术就是数据绑定（data binding），View的变化会直接影响ViewModel，ViewModel的变化或者内容也会直接体现在View上。这种模式实际上是框架替应用开发者做了一些工作，开发者只需要较少的代码就能实现比较复杂的交互。

#### 重构

重构的目的：

- 重构改进软件设计
- 重构使软件更容易理解
- 重构帮助找到bug
- 重构提高编程速度
- 重构使单元测试成为可能
- 重构应该是开发人员的进阶的技术手段


### TODO-MVP

[Android官方MVP架构项目解析](http://www.jianshu.com/p/389c9ae1a82c)

[Google 官方 MVP Sample 代码解读](https://juejin.im/entry/581067dcc4c9710058a86df5)

[Google官方架构MVP解析与实战](https://www.jianshu.com/p/569ab68da482)

[AndroidArchitectureCollection	](https://github.com/CameloeAnthony/AndroidArchitectureCollection)		