---
title: Android Studio 更新后出现Error:No service of type Factory  available in ProjectScopeServices.解决办法
date: 2017-03-16 01:01:52
categories: "错误志" 
tags:
  - Android
  - IDEError
---
## 问题描述 ##
今天更新Android Studio后 之前能运行的项目出现错误:
![](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image1.png)

## 解决办法 ##
查找到解决方案在此记录一下：
是github的 自动化打包插件maven 版本需要更新导致
![](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image2.png)
![](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image3.png)
