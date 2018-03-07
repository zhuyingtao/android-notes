#### Project

[RxJava](https://github.com/ReactiveX/RxJava)

[RxAndroid](https://github.com/ReactiveX/RxAndroid)

二者关系：

> RxAndroid是RxJava的一个针对Android平台的扩展，主要用于 Android 开发

#### Blog

- [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)
- [专栏 - Rxjava 详细教程](http://blog.csdn.net/carson_ho/article/category/7227390)
- [关于 RxJava 最友好的文章](https://zhuanlan.zhihu.com/p/24482660)
- [RxJava 的 Android 开发之路](https://huxian99.clarifygithub.io/tags/RxJava/)
- [RxJava 文档翻译](https://github.com/mcxiaoke/RxDocs)




#### RxJava 是什么

> 一个词：**异步**。
>
> RxJava 在 GitHub 主页上的自我介绍是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。这就是 RxJava ，概括得非常精准。

#### RxJava 好在哪

> 一个词：**简洁**。
>
> 异步操作很关键的一点是程序的简洁性，因为在调度过程比较复杂的情况下，异步代码经常会既难写也难被读懂。 Android 创造的 `AsyncTask` 和`Handler` ，其实都是为了让异步代码更加简洁。RxJava 的优势也是简洁，但它的简洁的与众不同之处在于，**随着程序逻辑变得越来越复杂，它依然能够保持简洁。**

#### RxJava 原理简析

1. 概念：扩展的观察者模式

RxJava 的异步实现，是通过一种扩展的观察者模式来实现的。

观察者模式：

![通用观察者模式](http://ww3.sinaimg.cn/mw1024/52eb2279jw1f2rx4446ldj20ga03p74h.jpg)

RxJava 的观察者模式大致如下图：

![RxJava 的观察者模式](http://ww3.sinaimg.cn/mw1024/52eb2279jw1f2rx46dspqj20gn04qaad.jpg)

