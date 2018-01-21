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




