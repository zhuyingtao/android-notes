![img](http://img.blog.csdn.net/20151022212109519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Context的继承结构还是稍微有点复杂的，可以看到，直系子类有两个，一个是ContextWrapper，一个是ContextImpl。那么从名字上就可以看出，ContextWrapper是上下文功能的封装类，而ContextImpl则是上下文功能的实现类。而ContextWrapper又有三个直接的子类，ContextThemeWrapper、Service和Application。其中，ContextThemeWrapper是一个带主题的封装类，而它有一个直接子类就是Activity。

Context一共有三种类型，分别是Application、Activity和Service。这三个类虽然分别各种承担着不同的作用，但它们都属于Context的一种，而它们具体Context的功能则是由ContextImpl类去实现的。

- Context数量 = Activity数量 + Service数量 + 1  (Application)


- 自定义 Application

```xml
<application  
    android:name=".MyApplication"  
    android:allowBackup="true"  
    android:icon="@drawable/ic_launcher"  
    android:label="@string/app_name"  
    android:theme="@style/AppTheme" >  
    ......  
</application>  
```

获取 Application 实例，可以使用getApplication()和 getApplicationContext()，它们获取到的对象是一样的。区别在于，getApplicationContext()使用范围更广一些，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法了。

还有一个 getBaseContext()方法，得到的是一个ContextImpl对象。ContextImpl正是上下文功能的实现类。也就是说像Application、Activity这样的类其实并不会去具体实现Context的功能，而仅仅是做了一层接口封装而已，Context的具体功能都是由ContextImpl类去完成的。具体源码如下：

```java
/** 
 * Proxying implementation of Context that simply delegates all of its calls to 
 * another Context.  Can be subclassed to modify behavior without changing 
 * the original Context. 
 */  
public class ContextWrapper extends Context {  
    Context mBase;  
      
    /** 
     * Set the base context for this ContextWrapper.  All calls will then be 
     * delegated to the base context.  Throws 
     * IllegalStateException if a base context has already been set. 
     *  
     * @param base The new base context for this wrapper. 
     */  
    protected void attachBaseContext(Context base) {  
        if (mBase != null) {  
            throw new IllegalStateException("Base context already set");  
        }  
        mBase = base;  
    }  
  
    /** 
     * @return the base context as set by the constructor or setBaseContext 
     */  
    public Context getBaseContext() {  
        return mBase;  
    }  
  
    @Override  
    public AssetManager getAssets() {  
        return mBase.getAssets();  
    }  
  
    @Override  
    public Resources getResources() {  
        return mBase.getResources();  
    }  
  
    @Override  
    public ContentResolver getContentResolver() {  
        return mBase.getContentResolver();  
    }  
	...
```