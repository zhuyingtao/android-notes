##### 文章

- [intent flags](http://blog.csdn.net/berber78/article/details/7278408)




#### 生命周期



![activity 生命周期](https://developer.android.google.cn/images/activity_lifecycle.png?_=6367282)

- `onStart()`和`onStop()`是从 activity **是否可见**这个角度来回调的；`onResume()`和`onPause()`是从 activity **是否位于前台**这个角度来回调的。
- 启动一个新的 activity ，旧的 activity 先`onPause()`，新的 activity 再启动。
- Activity 异常中止的情况下会调用`onSaveInstanceState()`保存数据，在`onCreate()`或`onRestoreInstance()`中恢复数据，二者都会被调用，区别是：前者要判空，因为不一定有数据；后者如果被调用，则一定有数据。
- 当系统配置发生改变后，activity 会重建，可以指定`configChanges`属性，来指定哪些情况下不重建。

```xml
android:configChanges="orientation|keyboardHidden"
```

另一种生命周期图

![activity_lifecycle](http://gityuan.com/images/activity/activity_lifecycle.jpg)

#### 启动模式

启动模式允许你去定义如何将一个Activity的实例和当前的任务进行关联，启动模式有两种定义方式：

- 使用 manifest 文件

  在manifest文件中声明一个Activity的时候，可以通过设置`<activity>`元素的launchMode属性，从而指定**这个Activity**在启动的时候该如何与任务进行关联。

  launchMode属性一共有以下四种可选参数：

  1. **standard** : 每次启动一个 activity 都会重新创建一个新的实例，不管这个实例是否已经存在。
  2. **singleTop**：如果要启动的 activity在当前任务栈中已经存在了，并且还是处于栈顶的位置，则不会再去创建此实例，否则会重新创建新的实例。
  3. **singleTask**：如果要启动的 activity 在当前任务栈中已经存在了，则不会创建实例，同时会将此 activity 调到栈顶，即清除位于此 activity 之上的其他 activity。否则会重新创建新的实例。
  4. **singleInstance**：一种加强的 singleTask 模式。具有此种模式的 activity 只能单独地位于一个任务栈中。

- 使用 intent flags

  当调用startActivity()方法时，可以在intent中加入一个flag，从而指定**新启动的Activity**该如何与任务进行关联。

  flag 一共有以下三种可选参数：

  1. **FLAG_ACTIVITY_NEW_TASK**：设置了这个flag，新启动Activity就会被放置到一个新的任务栈当中（与 singleTask 类似，但不完全一样）。
  2. **FLAG_ACTIVITY_SINGLE_TOP**：设置了这个flag，如果要启动的Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么就不会再次创建这个Activity的实例（与 singleTop 效果一样）。
  3. **FLAG_ACTIVITY_CLEAR_TOP**：设置了这个flag，如果要启动的Activity在当前任务中已经存在了，就不会再次创建这个Activity的实例，而是会把这个Activity之上的所有Activity全部关闭掉。

#### TaskAffinity

  affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，所以，这些Activity都更加倾向于运行在相同的任务当中。当然你也可以去改变每个Activity的affinity值，通过<activity>元素的taskAffinity属性就可以实现了。

  taskAffinity属性接收一个字符串参数，你可以指定成任意的值(经我测试字符串中至少要包含一个.)，但必须不能和应用程序的包名相同，因为系统会使用包名来作为默认的affinity值。

#### startActivity 流程

![start_activity_process](http://gityuan.com/images/activity/start_activity_process.jpg)

1. 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
3. Zygote进程fork出新的子进程，即App进程；
4. App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
6. App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
7. 主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。

#### 生命周期流程

对于App来说，其Activity的生命周期执行是与系统进程中的ActivityManagerService有一定关系的，接下来从进程和线程的角度来分析Activity的生命周期，这里涉及到系统进程和应用进程：

**system_server进程是系统进程**，Java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的。

**App进程是应用程序所在进程**，主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP），除了下图中所示的线程，其实还有很多线程，比如signal catcher线程等。

![app_process](http://gityuan.com/images/activity/app_process.jpg)

Activity的生命周期，都是其他线程通过handler发送消息给主线程，那么主线程中的`ActivityThread`的内部类`H`控制整个核心消息处理机制，通过`H.handleMessage()`来控制Activity的生命周期。
