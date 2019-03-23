---
title: NestedScrollView嵌套RecyclerView滑动冲突
categories: 错误志
tags:
  - Android
  - Design
  - error
date: 2018-11-21 23:09:46
---
## 问题描述 ##
Android使用disign 中控件NestedScrollView+RecyclerView嵌套时上下滑动会有明显的阻塞感，显然是遇到了滑动冲突
<!--more-->
```xml
<android.support.v4.widget.NestedScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="2dp"
        app:layout_constraintTop_toBottomOf="@id/space1"/>
</android.support.v4.widget.NestedScrollView>
```
## 解决方案 ##

在XML中 可以对RecyclerView 设置 android:nestedScrollingEnabled="false"不过这样只支持Android 21以上的版本，
对应的设置RecyclerView的属性mRecyclerView.setNestedScrollingEnabled(false)由于是support支持库中的方法即可兼容低版本。

