---
title: Android WebView必知必会(2)-WebSettings
categories: 技术栈
tags:
  - Android
date: 2019-03-07 23:29:48
---
## 简介 
WebSettings 看名称就明显了，是对WebView的一些基础设置，网页中需要的一些支持均需要在WebSettings 中设置。获取方式也很简单,通过WebView的方法获得即可。
```java
    public WebSettings getSettings() {
        checkThread();
        return mProvider.getSettings();
    }
```
<!--more-->
记录下对应的API含义，基本是直接找来的，大部分在项目用有验证，如果有使用偏差会随时回来修改，大家参考下。
```java
/**
 * 是否支持缩放，配合方法setBuiltInZoomControls使用，默认true
 */
setSupportZoom(boolean support)
 
/**
 * 是否需要用户手势来播放Media，默认true
 */
setMediaPlaybackRequiresUserGesture(boolean require)
 
/**
 * 是否使用WebView内置的缩放组件，由浮动在窗口上的缩放控制和手势缩放控制组成，默认false
 */
setBuiltInZoomControls(boolean enabled)
 
/**
 * 是否显示窗口悬浮的缩放控制，默认true
 */
setDisplayZoomControls(boolean enabled)
 
/**
 * 是否允许访问WebView内部文件，默认true
 */
setAllowFileAccess(boolean allow)
 
/**
 * 是否允许获取WebView的内容URL ，可以让WebView访问ContentPrivider存储的内容。 默认true
 */
setAllowContentAccess(boolean allow)
 
/**
 * 是否启动概述模式浏览界面，当页面宽度超过WebView显示宽度时，缩小页面适应WebView。默认false
 */
setLoadWithOverviewMode(boolean overview)
 
/**
 * 是否保存表单数据，默认false
 */
setSaveFormData(boolean save)
 
/**
 * 设置页面文字缩放百分比，默认100%
 */
setTextZoom(int textZoom)
 
/**
 * 是否支持ViewPort的meta tag属性，如果页面有ViewPort meta tag 指定的宽度，则使用meta tag指定的值，否则默认使用宽屏的视图窗口
 */
setUseWideViewPort(boolean use)
 
 
/**
 * 是否支持多窗口，如果设置为true ，WebChromeClient#onCreateWindow方法必须被主程序实现，默认false
 */
setSupportMultipleWindows(boolean support)
 
/**
 * 指定WebView的页面布局显示形式，调用该方法会引起页面重绘。默认LayoutAlgorithm#NARROW_COLUMNS
 */
setLayoutAlgorithm(LayoutAlgorithm l)
 
/**
 * 设置标准的字体族，默认”sans-serif”。font-family 规定元素的字体系列。
 * font-family 可以把多个字体名称作为一个“回退”系统来保存。如果浏览器不支持第一个字体，
 * 则会尝试下一个。也就是说，font-family 属性的值是用于某个元素的字体族名称或/及类族名称的一个
 * 优先表。浏览器会使用它可识别的第一个值。
 */
setStandardFontFamily(String font)
 
/**
 * 设置混合字体族。默认”monospace”
 */
setFixedFontFamily(String font)
 
/**
 * 设置SansSerif字体族。默认”sans-serif”
 */
setSansSerifFontFamily(String font)
 
/**
 * 设置SerifFont字体族，默认”sans-serif”
 */
setSerifFontFamily(String font)
 
/**
 * 设置CursiveFont字体族，默认”cursive”
 */
setCursiveFontFamily(String font)
 
/**
 * 设置FantasyFont字体族，默认”fantasy”
 */
setFantasyFontFamily(String font)
 
/**
 * 设置最小字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。
 */
setMinimumFontSize(int size)
 
/**
 * 设置最小逻辑字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。
 */
setMinimumLogicalFontSize(int size)
 
/**
 * 设置默认字体大小，默认16，取值区间[1-72]，超过范围，使用其上限值。
 */
setDefaultFontSize(int size)
 
/**
 * 设置默认填充字体大小，默认16，取值区间[1-72]，超过范围，使用其上限值。
 */
setDefaultFixedFontSize(int size)
 
/**
 * 设置是否加载图片资源，注意：方法控制所有的资源图片显示，包括嵌入的本地图片资源。
 * 使用方法setBlockNetworkImage则只限制网络资源图片的显示。值设置为true后，
 * webview会自动加载网络图片。默认true
 */
setLoadsImagesAutomatically(boolean flag)
 
/**
 * 是否加载网络图片资源。注意如果getLoadsImagesAutomatically返回false，则该方法没有效果。
 * 如果使用setBlockNetworkLoads设置为false，该方法设置为false，也不会显示网络图片。
 * 当值从true改为false时。WebView会自动加载网络图片。
 */
setBlockNetworkImage(boolean flag)
 
/**
 * 设置是否加载网络资源。注意如果值从true切换为false后，WebView不会自动加载，
 * 除非调用WebView#reload().如果没有android.Manifest.permission#INTERNET权限，
 * 值设为false，则会抛出java.lang.SecurityException异常。
 * 默认值：有android.Manifest.permission#INTERNET权限时为false，其他为true。
 */
setBlockNetworkLoads(boolean flag)
 
/**
 * 设置是否允许执行JS。
 */
setJavaScriptEnabled(boolean flag)
 
/**
 * 是否允许Js访问任何来源的内容。包括访问file scheme的URLs。考虑到安全性，
 * 限制Js访问范围默认禁用。注意：该方法只影响file scheme类型的资源，其他类型资源如图片类型的，
 * 不会受到影响。ICE_CREAM_SANDWICH_MR1版本以及以下默认为true，JELLY_BEAN版本
 * 以上默认为false
 */
setAllowUniversalAccessFromFileURLs(boolean flag)
 
 
/**
 * 是否允许Js访问其他file scheme的URLs。包括访问file scheme的资源。考虑到安全性，
 * 限制Js访问范围默认禁用。注意：该方法只影响file scheme类型的资源，其他类型资源如图片类型的，
 * 不会受到影响。如果getAllowUniversalAccessFromFileURLs为true，则该方法被忽略。
 * ICE_CREAM_SANDWICH_MR1版本以及以下默认为true，JELLY_BEAN版本以上默认为false
 */
setAllowFileAccessFromFileURLs(boolean flag)
 
/**
 * 设置存储定位数据库的位置，考虑到位置权限和持久化Cache缓存，Application需要拥有指定路径的
 * write权限
 */
setGeolocationDatabasePath(String databasePath)
 
/**
 * 是否允许Cache，默认false。考虑需要存储缓存，应该为缓存指定存储路径setAppCachePath
 */
setAppCacheEnabled(boolean flag)
 
/**
 * 设置Cache API缓存路径。为了保证可以访问Cache，Application需要拥有指定路径的write权限。
 * 该方法应该只调用一次，多次调用自动忽略。
 */
setAppCachePath(String appCachePath)
 
/**
 * 是否允许数据库存储。默认false。查看setDatabasePath API 如何正确设置数据库存储。
 * 该设置拥有全局特性，同一进程所有WebView实例共用同一配置。注意：保证在同一进程的任一WebView
 * 加载页面之前修改该属性，因为在这之后设置WebView可能会忽略该配置
 */
setDatabaseEnabled(boolean flag)
 
/**
 * 是否存储页面DOM结构，默认false。
 */
setDomStorageEnabled(boolean flag)
 
/**
 * 是否允许定位，默认true。注意：为了保证定位可以使用，要保证以下几点：
 * Application 需要有android.Manifest.permission#ACCESS_COARSE_LOCATION的权限
 * Application 需要实现WebChromeClient#onGeolocationPermissionsShowPrompt的回调，
 * 接收Js定位请求访问地理位置的通知
 */
setGeolocationEnabled(boolean flag)
 
/**
 * 是否允许JS自动打开窗口。默认false
 */
setJavaScriptCanOpenWindowsAutomatically(boolean flag)
 
/**
 * 设置页面的编码格式，默认UTF-8
 */
setDefaultTextEncodingName(String encoding)
 
/**
 * 设置WebView代理，默认使用默认值
 */
setUserAgentString(String ua)
 
/**
 * 通知WebView是否需要设置一个节点获取焦点当
 * WebView#requestFocus(int,android.graphics.Rect)被调用的时候，默认true
 */
setNeedInitialFocus(boolean flag)
 
/**
 * 基于WebView导航的类型使用缓存：正常页面加载会加载缓存并按需判断内容是否需要重新验证。
 * 如果是页面返回，页面内容不会重新加载，直接从缓存中恢复。setCacheMode允许客户端根据指定的模式来
 * 使用缓存。
 * LOAD_DEFAULT 默认加载方式
 * LOAD_CACHE_ELSE_NETWORK 按网络情况使用缓存
 * LOAD_NO_CACHE 不使用缓存
 * LOAD_CACHE_ONLY 只使用缓存
 */
setCacheMode(int mode)
 
/**
 * 设置加载不安全资源的WebView加载行为。KITKAT版本以及以下默认为MIXED_CONTENT_ALWAYS_ALLOW方
 * 式，LOLLIPOP默认MIXED_CONTENT_NEVER_ALLOW。强烈建议：使用MIXED_CONTENT_NEVER_ALLOW
 */
setMixedContentMode(int mode)
```

