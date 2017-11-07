### Android Service

Service作为Android四大组件之一，在每一个应用程序中都扮演着非常重要的角色。它主要用于在后台处理一些耗时的逻辑，或者去执行某些需要长期运行的任务。必要的时候我们甚至可以在程序退出的情况下，让Service在后台继续保持运行状态。

#### 基本用法

启动Service的方法和启动Activity很类似，都需要借助Intent来实现。

首先，自定义一个Service：

```java
public class MyService extends Service {  
  
    public static final String TAG = "MyService";  
  
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.d(TAG, "onCreate() executed");  
    }  
  
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.d(TAG, "onStartCommand() executed");  
        return super.onStartCommand(intent, flags, startId);  
    }  
      
    @Override  
    public void onDestroy() {  
        super.onDestroy();  
        Log.d(TAG, "onDestroy() executed");  
    }  
  
    @Override  
    public IBinder onBind(Intent intent) {  
        return null;  
    }  
}  

```

然后，在 AndroidManifest.xml 中注册这个Service:

```xml
<service android:name="com.example.servicetest.MyService" />  
```

启动 Service:

```java
Intent startIntent = new Intent(this, MyService.class);  
startService(startIntent);  
```

当启动一个Service的时候，会调用该Service中的onCreate()和onStartCommand()方法。由于onCreate()方法只会在Service第一次被创建的时候调用，如果当前Service已经被创建过了，不管怎样调用startService()方法，onCreate()方法都不会再执行。

停止 Service:

```java
Intent stopIntent = new Intent(this, MyService.class);  
stopService(stopIntent);
```

#### 组件间通信

使用 `onBind()`方法，可以建立 Service 与 Activity 之间的联系，具体如下：

```java
public class MyService extends Service {  
  
    public static final String TAG = "MyService";  
  
    private MyBinder mBinder = new MyBinder();  
  
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.d(TAG, "onCreate() executed");  
    }  
  
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.d(TAG, "onStartCommand() executed");  
        return super.onStartCommand(intent, flags, startId);  
    }  
  
    @Override  
    public void onDestroy() {  
        super.onDestroy();  
        Log.d(TAG, "onDestroy() executed");  
    }  
  
    @Override  
    public IBinder onBind(Intent intent) {  
        return mBinder;  
    }  
  
    public class MyBinder extends Binder {  
  
        public void startDownload() {  
            Log.d("TAG", "startDownload() executed");  
            // 执行具体的下载任务  
        }  
    }  
}
```

在 Activity 中，首先创建一个 ServiceConnection 的匿名类，重写`onServiceDisconnected()`和`onServiceConnected`() 方法，在`onServiceConnected()`方法中，又通过向下转型得到了MyBinder的实例，有了这个实例，Activity和Service之间的关系就变得非常紧密了。

```java
private MyService.MyBinder myBinder;  

private ServiceConnection connection = new ServiceConnection() {  
  
        @Override  
        public void onServiceDisconnected(ComponentName name) {  
        }  
  
        @Override  
        public void onServiceConnected(ComponentName name, IBinder service) {  
            myBinder = (MyService.MyBinder) service;  
            myBinder.startDownload();  
        }  
};  

```

接下来，需要绑定 Service：

```java
Intent bindIntent = new Intent(this, MyService.class);  
bindService(bindIntent, connection, BIND_AUTO_CREATE);  
```

bindService()方法接收三个参数，第一个参数就是刚刚构建出的Intent对象，第二个参数是前面创建出的ServiceConnection的实例，第三个参数是一个标志位，这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service，这会使得MyService中的onCreate()方法得到执行，但onStartCommand()方法不会执行。

解除绑定 Service:

```java
unbindService(connection);  
```

一个Service必须要在既没有和任何Activity关联又处理停止状态的时候才会被销毁。一个既 start 又 bind 的 service 必须既 stop 又 unbind 才会被销毁。

#### Service 和 Thread

Service和Thread之间没有任何关系，Service其实是运行在主线程里的。也就是说如果你在Service里编写了非常耗时的代码，程序必定会出现ANR的。

我们可以在Service中再创建一个子线程，然后在这里去处理耗时逻辑就没问题了。

那为什么不直接在Activity里创建呢？原因如下：

1. Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而 Service 不一样，只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例
2. 在一个Activity中创建的子线程，另一个Activity无法对其进行操作。所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法。

#### 前台 Service

当系统出现内存不足情况时，就有可能会回收掉正在后台运行的Service。如果你希望Service可以一直保持运行状态，而不会由于系统内存不足的原因导致被回收，就可以考虑使用前台Service。

前台Service和普通Service最大的区别就在于，它会一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。

调用`startForeground()`方法就可以让Service变成一个前台Service。

