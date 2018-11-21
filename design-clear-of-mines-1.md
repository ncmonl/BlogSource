---
title: Android NestedScrollView嵌套ViewPager 内容不显示的问题
date: 2018-09-26 20:40:11
categories: "错误志"
tags:
  - Android
  - Design
---
## 问题描述 ##
Android使用disign中自带控件CoordinatorLayout+TabLayout+NestedScrollView+ViewPager来实现可折叠头部的也可横下滑动的列表的页面
NestedScrollView中嵌套了ViewPager来展示不同的Tab滑动展示，ViewPager显示的高度为0，从AndroidStudio预览页面即可看到。

## 解决方案 ##
为NestedScrollView设置 android:fillViewport="true" 解决
```Xml
<android.support.v4.widget.NestedScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">

    <android.support.v4.view.ViewPager
        android:id="@+id/vp_posts"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</android.support.v4.widget.NestedScrollView>
```
