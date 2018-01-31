#### 生命周期

![Fragment 生命周期](http://upload-images.jianshu.io/upload_images/2244681-3685a0866eb07d3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 加载方式

- 静态加载				

布局文件 ```frament1.xml```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:background="#00ff00" >  
  
    <TextView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="This is fragment 1"  
        android:textColor="#000000"  
        android:textSize="25sp" />  
  
</LinearLayout> 
```

类文件 ```Fragment1.java```

```java
public class Fragment1 extends Fragment {  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {  
        return inflater.inflate(R.layout.fragment1, container, false);  
    }  
}  
```

在activity布局文件中静态加载``` activity_main.xml```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:baselineAligned="false" >  
  
    <fragment  
        android:id="@+id/fragment1"  
        android:name="com.example.fragmentdemo.Fragment1"  
        android:layout_width="0dip"  
        android:layout_height="match_parent"  
        android:layout_weight="1" />  
  
    <fragment  
        android:id="@+id/fragment2"  
        android:name="com.example.fragmentdemo.Fragment2"  
        android:layout_width="0dip"  
        android:layout_height="match_parent"  
        android:layout_weight="1" />  
  
</LinearLayout>  
```

- 动态加载

一般分为4步：

1. 获取到FragmentManager，在Activity中可以直接通过getFragmentManager得到。
2. 开启一个事务，通过调用beginTransaction方法开启。
3. 向容器内加入Fragment，一般使用replace方法实现，需要传入容器的id和Fragment的实例。
4. 提交事务，调用commit方法提交。

```java
Fragment1 fragment1 = new Fragment1();  
// R.id.main_layout 一般是FrameLayout
getFragmentManager().beginTransaction().replace(R.id.main_layout, fragment1).commit();
```

#### 组件间通信

主要都是通过getActivity这个方法实现的。getActivity方法可以让Fragment获取到关联的Activity，然后再调用Activity的findViewById方法，就可以获取到和这个Activity关联的其它Fragment的视图了。