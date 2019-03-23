---
title: Android Studio import project 出现Error错误 程序包org.apache.http不存在
date: 2018-01-07 23:01:52
categories: "错误志" 
tags:
  - Android
  - IDEError
---
## 问题描述 ##

遇到新的产品形态、样式需求我们经常会在Github搜索开源项目，有些项目是早期Android版本中开发而成，后续Google会有部分API在新版本中的SDK放弃集成，这次我导入一个项目时就遇到这样一个错误

![Error Imag](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image4.png)

## 解决办法 ##
如果经历过Android 4.4 时期的同学应该会觉得org.apache.http这个包很熟悉，这里显示错误是因为在targetsdk：23以上版本SDK中已经不再集成此包，如需使用可添加useLibrary "org.apache.http.legacy"依赖即可 。

```groovy
apply plugin: 'com.android.application'
android {
   	compileSdkVersion 23
    buildToolsVersion "23.0.3"
    useLibrary "org.apache.http.legacy"
	...
}
```
