#### Project

[RxJava](https://github.com/ReactiveX/RxJava)

[RxAndroid](https://github.com/ReactiveX/RxAndroid)

二者关系：

> RxAndroid是RxJava的一个针对Android平台的扩展，主要用于 Android 开发

#### Blog

- [给初学者的RxJava2.0教程](https://www.jianshu.com/p/464fa025229e)
- [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)
- [专栏 - Rxjava 详细教程](http://blog.csdn.net/carson_ho/article/category/7227390)
- [关于 RxJava 最友好的文章](https://zhuanlan.zhihu.com/p/24482660)
- [RxJava 的 Android 开发之路](https://huxian99.clarifygithub.io/tags/RxJava/)
- [RxJava 文档翻译](https://github.com/mcxiaoke/RxDocs)
- [这可能是最好的RxJava 2.x 教程（完结版）](https://www.jianshu.com/p/0cd258eecf60)

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

RxJava 原理可总结为：被观察者（Observable）通过 订阅（Subscribe）**按顺序发送事件** 给观察者 （Observer）， 观察者（Observer） **按顺序接收事件**并作出对应的响应动作。

![img](https://upload-images.jianshu.io/upload_images/944365-98ec92df0a4d7e0b.png)

#### RxJava 示例

这里我们就根据上面的步骤来实现这个例子

```java
        //步骤1. 创建被观察者(Observable),定义要发送的事件。
        Observable observable = Observable.create(
        new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter)
            throws Exception {
                emitter.onNext("文章1");
                emitter.onNext("文章2");
                emitter.onNext("文章3");
                emitter.onComplete();
            }
        });
       
        //步骤2. 创建观察者(Observer)，接受事件并做出响应操作。
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "onSubscribe");
            }

            @Override
            public void onNext(String s) {
                Log.d(TAG, "onNext : " + s);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "onError : " + e.toString());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete");
            }
        };
       
        //步骤3. 观察者通过订阅（subscribe）被观察者把它们连接到一起。
        observable.subscribe(observer);

```

线程切换

```java
        new Thread() {
            @Override
            public void run() {
                Log.d(TAG, "Thread run() 所在线程为 :" + Thread.currentThread().getName());
                Observable
                        .create(new ObservableOnSubscribe<String>() {
                            @Override
                            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                                Log.d(TAG, "Observable subscribe() 所在线程为 :" + Thread.currentThread().getName());
                                emitter.onNext("文章1");
                                emitter.onNext("文章2");
                                emitter.onComplete();
                            }
                        })
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(new Observer<String>() {
                            @Override
                            public void onSubscribe(Disposable d) {
                                Log.d(TAG, "Observer onSubscribe() 所在线程为 :" + Thread.currentThread().getName());
                            }

                            @Override
                            public void onNext(String s) {
                                Log.d(TAG, "Observer onNext() 所在线程为 :" + Thread.currentThread().getName());
                            }

                            @Override
                            public void onError(Throwable e) {
                                Log.d(TAG, "Observer onError() 所在线程为 :" + Thread.currentThread().getName());
                            }

                            @Override
                            public void onComplete() {
                                Log.d(TAG, "Observer onComplete() 所在线程为 :" + Thread.currentThread().getName());
                            }
                        });
            }
        }.start();

```

`Observer`（观察者）的`onSubscribe()`方法运行在当前线程中。

`Observable`（被观察者）中的`subscribe()`运行在`subscribeOn()`指定的线程中。

`Observer`（观察者）的`onNext()`和`onComplete()`等方法运行在`observeOn()`指定的线程中。