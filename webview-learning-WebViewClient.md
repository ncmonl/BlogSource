---
title: Android WebView必知必会(1)-WebViewClient
date: 2019-02-23 14:27:48
categories: 技术栈
tags:
  - Android
  - WebView
---

项目中遇到了混合开发怎么办，不需要慌，Android提供的WebView已经非常强大，可以和前端同学配合好完成非常多的事情。配合之前呢就需要对WebView做一个全面的了解，看看WebView能做些什么，又怎么去做这些事情，又会遇到哪些问题。

## WebView简介 ##
从Android4.4系统开始，Chromium内核取代了Webkit内核，正式地接管了WebView的渲染工作。Chromium是一个开源的浏览器内核项目，基于Chromium开源项目修改实现的浏览器非常多，包括最著名的Chrome浏览器，以及一众国内浏览器（360浏览器、QQ浏览器等）。其中Chromium在Android上面的实现是`Android System WebView`。

<!--more-->

从Android5.0系统开始，WebView移植成了一个独立的apk，可以不依赖系统而独立存在和更新，我们可以在`系统->设置->Android System WebView`看到WebView的当前版本。

从Android7.0系统开始，如果系统安装了Chrome (version>51)，那么Chrome将会直接为应用的WebView提供渲染，WebView版本会随着Chrome的更新而更新，用户也可以选择WebView的服务提供方（在开发者选项->WebView Implementation里），WebView可以脱离应用，在一个独立的沙盒进程中渲染页面（需要在开发者选项里打开）。

从Android8.0系统开始，默认开启WebView多进程模式，即WebView运行在独立的沙盒进程中。

## WebViewClient ##
### 抛砖 ###
WebView加载在线网页必备，如果不设置，会直接跳转打开浏览器应用加载网页！同时包含很多有用的回调函数。

举一个小栗子：

通常加载一个网页都需要一定的时间，这时候就需要一个loading友好提示一下用户，正在加载，加载完成后隐藏它。这个怎么实现呢，就需要用到WebViewClient了，WebViewClient包含了很多网页加载需要用到的回调。

```java
mWebView.setWebViewClient(new WebViewClient(){
    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        super.onPageStarted(view, url, favicon);
        Log.d(TAG,"onPageStarted");
    }
 
    @Override
    public void onPageFinished(WebView view, String url) {
        super.onPageFinished(view, url);
        Log.d(TAG,"onPageFinished");
    }
});
```
WebView.setWebViewClient方法设置WebViewClient回调，重写了两个方法，onPageStarted会在加载网页时调用，onPageFinished就代表加载完成了。利用这两个方法就能完成loading的控制。在onPageFinished中也可以通过WebView.getTitle（）可获得当前网页加载完时的title(这里指Html文件中head中的title)等，当然对某些网页时不准的，因为网页中可能会通过JS代码动态的修改title,这个操作是在onPageFinish()之后进行的，这个title在另外的函数可以获取到，总结WebChromeClient的时候会讲到。
### 常用函数简介

不一定全都会用到，先学习一下，以备不时之需，会挑几个非常常用的总结下使用场景。

```java
/**
 * 在开始加载网页时会回调
 */
public void onPageStarted(WebView view, String url, Bitmap favicon) 
/**
 * 在结束加载网页时会回调
 */
public void onPageFinished(WebView view, String url)
/**
 * 拦截 url 跳转,在里边添加点击链接跳转或者操作
 */
@Deprecated
public boolean shouldOverrideUrlLoading(WebView view, String url)
//sdk>=24    
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request)
/**
 * 在每一次请求资源时，都会通过这个函数来回调,请求资源的主机应用程序允许应用程序返回数据，如果该方法返回
 * null，WebView将会按照平常一样继续加载；否则，返回的内容将会被使用。该方法在独立线程而非UI线程中回
 * 调，因此访问私有数据或系统视图时应该谨慎行事。
 */
@Deprecated
public WebResourceResponse shouldInterceptRequest(WebView view,String url) {
    return null;
}
public WebResourceResponse shouldInterceptRequest(WebView view,WebResourceRequest request) {
	return shouldInterceptRequest(view, request.getUrl().toString());
}
/**
 *加载资源完成（与shouldInterceptRequest对应）
 */
 public void onLoadResource(WebView view, String url) 
/**
 * 加载错误的时候会回调，在其中可做错误处理，比如再请求加载一次，或者提示自定义的网路问题页面
 */
@Deprecated
public void onReceivedError(WebView view, int errorCode,String description, String failingUrl)
//sdk>=23 
public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error)
/**
 * 当接收到https错误时，会回调此函数，在其中可以做错误处理
 */
public void onReceivedSslError(WebView view, SslErrorHandler handler,SslError error)
/**
 * 回调该方法，处理SSL认证请求
 */
public void onReceivedClientCertRequest(WebView view, ClientCertRequest request)
/**
 * 页面大小改变后回调该方法，获取缩放前后大小
 */
public void onScaleChanged(WebView view, float oldScale, float newScale)
/**
 * 更新历史记录
 * @param isReload 代表Url是重新载入的
 */
public void doUpdateVisitedHistory(WebView view, String url, boolean isReload)
/**
 * 账户自动登录
 */
public void onReceivedLoginRequest(WebView view, String realm,String account, String args) 
/**
 * 通知主程序输入事件不是由WebView调用。是否让主程序处理WebView未处理的Input Event。
 * 除了系统按键，WebView总是消耗掉输入事件或shouldOverrideKeyEvent返回true。
 * 该方法由event 分发异步调用。注意：如果事件为MotionEvent，则事件的生命周期只存在方法调用过程中，
 * 如果WebViewClient想要使用这个Event，则需要复制Event对象。
 */
onUnhandledInputEvent(WebView view, InputEvent event)
/**
 * 如果浏览器需要重新发送POST请求，可以通过这个时机来处理。默认是不重新发送数据。 参数说明
 * @param dontResend 浏览器不需要重新发送的参数
 * @param resend浏览器需要重新发送的参数
 */
@Override
public void onFormResubmission(WebView view, Message dontResend,Message resend)！
    
```

#### shouldOverrideUrlLoading

非常重要的方法，第一个就需要说明，重定向，超链接都会用的方法，这个函数在加载超链接时会回调，由于这个函数是在加载Url之前调用，返回值true代表拦截处理就不再继续加载了，返回false则继续加载，所以如果需要做一些白名单黑名单操作可以在这里处理它们。

```java
public boolean shouldOverrideUrlLoading(WebView view, String url) {
	return false;
}
```

默认既返回false。所以如果没有其他设置需求可以采用设置一个默认的WebViewClient既能保持正常加载网页。

```java
mWebView.setWebViewClient(new WebViewClient());
```

有重载方法，在SDK>=24调用，所以如果需要处理，需要两个函数都重写

```java
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request)
```

#### onReceivedError

接收网络错误，典型使用场景是加载404页面

```java
public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
        //如果是主frame的话就执行方法，要是在其他比如iframes就会不执行
	if (request.isForMainFrame()) {
		onReceivedError(view,error.getErrorCode(), rror.getDescription().toString(),request.getUrl().toString());
	}
    //处理加载本地错误页面
    mWebView.loadUrl("file:///android_asset/http_error.html")
}
```

WebResourceError包含了错误信息,其中的ErrorCode对应WebViewClient中的常量ERROR_*可以做对应处理

```java
public abstract class WebResourceError {
    public abstract int getErrorCode();
    public abstract CharSequence getDescription();
    @SystemApi
    public WebResourceError() {}
}
```

#### onReceivedSslError

很多帖子都介绍了这个方法，顾名思义与处理SSL错误有关，也就是处理https的请求时验证出错，看到的很多例子都是以以前的12306网站为例，以前的12306是自签名证书，所以用chrome打开时会出现警告，如果用WebView加载这里就会回调onReceivedSslError函数，笔者写这篇笔记时12306已经是公共证书了，也就没有这个错误验证了。暂时使用自己的理解介绍一下，以后发现可用网站再验证：

该函数的默认实现为

```java
public void onReceivedSslError(WebView view, SslErrorHandler handler,SslError error) {
	handler.cancel();
}
```

SslErrorHandler.cancel()就是结束了Ssl处理，也就不会继续加载了，这里子类可以重写处理逻辑为：

```java
public void onReceivedSslError(WebView view, SslErrorHandler handler,SslError error) {
    //super.onReceivedSslError(view, handler, error);
	handler.proceed();
}
```

改变了SslErrorHandler的处理，使其能继续验证证书，正常打开该网页。

`需要注意的是如果有回调了onReceivedSslError，是不会同时回调onReceivedError的`

WebViewClient还有很多没有介绍到的方法，有机会还是找一些应用场景试验下再行补充，这篇文章先到这里吧，之后再总结下常用设置和WebChromeClient相关。