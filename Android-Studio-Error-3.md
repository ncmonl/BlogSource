---
title: Android Studio import project Error:Unsupported method:AndroidProject.getVariantNames()
date: 2019-05-15 14:481:52
categories: "错误志" 
tags:
  - Android
  - IDEError
---

## Unsupported method:AndroidProject.getVariantNames()

### 问题描述
以下为Android Studio报出的错误：  
```groovy
ERROR: Unsupported method: AndroidProject.getVariantNames().
The version of Gradle you connect to does not support that method.
To resolve the problem you can change/upgrade the target version of Gradle you connect to.
Alternatively, you can ignore this exception and read other information from the model.
```

### 解决方案
You need to disable this setting in Android Studio:
```groovy
File > Settings(or Preferences for previous versions) > Experimental > Only sync active variant
```
