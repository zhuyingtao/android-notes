### Android Service

Service作为Android四大组件之一，在每一个应用程序中都扮演着非常重要的角色。它主要用于在后台处理一些耗时的逻辑，或者去执行某些需要长期运行的任务。必要的时候我们甚至可以在程序退出的情况下，让Service在后台继续保持运行状态。

#### 生命周期

![service lifecycle](assets/android-service/service.png)

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

#### 远程 Service

将一个普通的Service转换成远程Service其实非常简单，只需要在注册Service的时候将它的android:process属性指定成:remote就可以了。

```xml
<service  
        android:name="com.example.servicetest.MyService"  
        android:process=":remote" >  
</service>  
```

使用了远程Service之后，Activity与Service之间由于现在在不同的进程，无法再像之前那样建立关联。

如何才能让Activity与一个远程Service建立关联呢？这就要使用AIDL来进行跨进程通信了（IPC）。

#### AIDL

AIDL（Android Interface Definition Language）是Android接口定义语言的意思，它可以用于让某个Service与多个应用程序组件之间进行跨进程通信，从而可以实现多个应用程序共享同一个Service的功能。

AIDL的使用方法：

- Service端，首先新建一个AIDL文件：

```java
package com.example.servicetest;  
interface MyAIDLService {  
    int plus(int a, int b);  
    String toUpperCase(String str);  
}  
```

然后修改MyService中的代码，在里面实现我们刚刚定义好的MyAIDLService接口，

```	java
public class MyService extends Service {  
  
    ......  
  
    @Override  
    public IBinder onBind(Intent intent) {  
        return mBinder;  
    }  
  
    MyAIDLService.Stub mBinder = new Stub() {  
  
        @Override  
        public String toUpperCase(String str) throws RemoteException {  
            if (str != null) {  
                return str.toUpperCase();  
            }  
            return null;  
        }  
  
        @Override  
        public int plus(int a, int b) throws RemoteException {  
            return a + b;  
        }  
    };  
  
}  
```

由于是跨进程通信，所以必须使用隐式Intent启动Service，需要在AndroidManifest.xml中声明，

```xml
<service  
        android:name="com.example.servicetest.MyService"  
        android:process=":remote" >  
        <intent-filter>  
            <action android:name="com.example.servicetest.MyAIDLService"/>  
        </intent-filter>  
</service>  
```

- Activity端，首先要把aidl文件复制过来，路径也要一致，相当于做了一个备份，目的是为了编译时候能够通过，而真正运行时，则会去找远端Service中真正的aidl。

在Activity中，建立与远端Service的关联：

```java
	private MyAIDLService myAIDLService;  
  
    private ServiceConnection connection = new ServiceConnection() {  
  
        @Override  
        public void onServiceDisconnected(ComponentName name) {  
        }  
  
        @Override  
        public void onServiceConnected(ComponentName name, IBinder service) {  
            myAIDLService = MyAIDLService.Stub.asInterface(service);  
            try {  
                int result = myAIDLService.plus(50, 50);  
                String upperStr = myAIDLService.toUpperCase("comes from ClientTest");  
                Log.d("TAG", "result is " + result);  
                Log.d("TAG", "upperStr is " + upperStr);  
            } catch (RemoteException e) {  
                e.printStackTrace();  
            }  
        }  
    };  
```

绑定远端Service：

```java
 Intent intent = new Intent("com.example.servicetest.MyAIDLService");  
 bindService(intent, connection, BIND_AUTO_CREATE);  
```

由于这是在不同的进程之间传递数据，Android对这类数据的格式支持是非常有限的，基本上只能传递Java的基本数据类型、字符串、List或Map等。那么如果我想传递一个自定义的类该怎么办呢？这就必须要让这个类去实现Parcelable接口，并且要给这个类也定义一个同名的AIDL文件。

#### 实现原理

![start_server_binder](assets/android-service/start_server_binder.jpg)

首先在发起方进程调用AMP.startService，经过binder驱动，最终调用系统进程AMS.startService。AMP和AMN都是实现了IActivityManager接口,AMS继承于AMN. 其中AMP作为Binder的客户端,运行在各个app所在进程, AMN(或AMS)运行在系统进程system_server。

#### 详细流程

在整个startService过程，从进程角度看服务启动过程

- **Process A进程：**是指调用startService命令所在的进程，也就是启动服务的发起端进程，比如点击桌面App图标，此处Process A便是Launcher所在进程。
- **system_server进程：**系统进程，是java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的，每个进程binder线程个数的上限为16。
- **Zygote进程：**是由`init`进程孵化而来的，用于创建Java层进程的母体，所有的Java层进程都是由Zygote进程孵化而来；
- **Remote Service进程：**远程服务所在进程，是由Zygote进程孵化而来的用于运行Remote服务的进程。主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP）。

![start_service_process](assets/android-service/start_service_processes.jpg)

启动流程：

1. Process A进程采用Binder IPC向system_server进程发起startService请求；
2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
3. zygote进程fork出新的子进程Remote Service进程；
4. Remote Service进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向remote Service进程发送scheduleCreateService请求；
6. Remote Service进程的binder线程在收到请求后，通过handler向主线程发送CREATE_SERVICE消息；
7. 主线程在收到Message后，通过发射机制创建目标Service，并回调Service.onCreate()方法。

到此，服务便正式启动完成。当创建的是本地服务或者服务所属进程已创建时，则无需经过上述步骤2、3，直接创建服务即可。