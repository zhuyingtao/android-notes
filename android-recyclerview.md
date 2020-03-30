#### 概述

RecyclerView是谷歌V7包下新增的控件,用来替代ListView的使用。

它功能更强大，定制样式更丰富，扩展性更高。

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

#### 与 ListView 区别

recyclerView的其中一个非常明显的优点是能快速的切换数据的排列方式，这在横竖屏/平板适配上非常有用，且也能够在一个布局里混合多种排列方式，通过设置某个item的所占的列数，再结合设置不同的item的的type能在一个recycleView里实现非常复杂的布局，这是ListView无法轻易就能做到的。总之在复杂的布局里，recycleView完全是神兵利器。

- 缓存机制不同

  - 层级不同

    ListView和RecyclerView缓存机制基本一致，但 ListView 是两级缓存，RecyclerView 是四级缓存

  - 缓存不同

    RecyclerView 缓存RecyclerView.ViewHolder，抽象可理解为：View + ViewHolder

    ListView 缓存 View

- 局部刷新

  RecycleView 提供了局部刷新的接口，避免调用许多无用的 bindView

