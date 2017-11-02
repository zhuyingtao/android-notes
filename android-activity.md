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

1. standard : 标准模式。每次启动一个 activity 都会重新创建一个新的实例，不管这个实例是否已经存在。



