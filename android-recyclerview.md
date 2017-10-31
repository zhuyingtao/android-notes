#### 概述

RecyclerView是谷歌V7包下新增的控件,用来替代ListView的使用

> RecyclerView is a more advanced and flexible version of ListView. This widget is a container for large sets of views that can be recycled and scrolled very efficiently. Use the RecyclerView widget when you have lists with elements that change dynamically. 

#### 用法

1. 在xml 文件中添加：

```xml
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/recycler_view"
    android:layout_centerVertical="true"
    android:layout_centerHorizontal="true"/>
```

2. 在activity中配置：

```java
mRecyclerView = findView(R.id.recycler_view);
//设置布局管理器
mRecyclerView.setLayoutManager(layout);
//设置adapter
mRecyclerView.setAdapter(adapter)
//设置Item增加、移除动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
//添加分割线
mRecyclerView.addItemDecoration(new DividerItemDecoration(
             		getActivity(), DividerItemDecoration.HORIZONTAL_LIST));
```



#### Github

- [AndroidRecyclerViewDemo](https://github.com/Frank-Zhu/AndroidRecyclerViewDemo)