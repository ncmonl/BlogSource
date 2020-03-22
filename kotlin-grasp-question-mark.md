---
title: Kotlin学习笔记-较难理解的知识点？
date: 2020-02-28 22:01:52
categories: "技术栈" 
tags:
  - Android
  - Kotlin
---

## 前言

对我们来说，需求是学习的最大动力，比兴趣动力都足，之前对Kotlin一直是一知半解，没有系统性的学习，最近开始看很多open source都开始使用Kotlin语言来写了，很多社区的大神们也极力在推荐学习，再不学习都要跟不上了，刚好最近有时间学习一下，找了一本书《Kotlin核心编程》感觉讲的还比较容易理解，成体系，所以要集中系统的学习一下，为以后阅读代码，甚至转向Kotlin项目做些准备。

本系列主要是记录一下自己学习中感觉与Java相比，思想上比较难转换或者说难记忆的点，也有一些是觉得会很常用的知识点，加深记忆可供以后查询，同时也想分享给大家一下，好比上学时期的老师划重点，话不多说，看好小黑板~。

## 一. 极有特点的？

#### 1.1 ？

单独的？代表可以为空@Nullable ，Kotlin默认定义属性时不能为null,写在定义属性时，表示该属性可空

```kotlin
val type:String = null //这样定义会报错，默认不允许为null
val type:String? = null //增加？代表可以定义为null
//扩展一下
val type = null //这样也不会报错，定义了一个无意义的属性Kotlin中的Nothing也不能修改为其他值
```

#### 1.2 ?. 和!!.

```kotlin
seat?.student
```
直接对应Java中的Nullcheck,先判断是否为null，然后再调用，如果为空就返回null

同Java代码表示

```java
if(seat != null){
    return seat.student
}else{
    return null
}
```

!!.是和?.对应的，

**?.程序允许为空**

**!!.程序不允许为空**

```kotlin
seat!!.student
```

直接对应Java中的Nullcheck,先判断是否为null，然后再调用，如果为空就抛出异常

对应的Java代码表示

```java
if(seat != null){
    return seat.student
}else{
    throw new NullPointerException();
}
```



#### 1.3 ?:

```kotlin
val c = a ?: b // 
```

判断a是否为null，如果不为null返回a，如果为null返回b

同Java代码表示

```java
a != null ? c = a : c = b 
```

#### 1.4 as?

```kotlin
val x = y as? String
```

安全的类型转换运算符，默认不加？时，判断如果x不能转换为String类型是会跑出java.lang.ClassCastException

增加？后，x如果不能转换为String，会返回一个null值。

同Java代码

```java
if (y instenceof String){
    x = (String)y;
}else{
    x = null
}
```
