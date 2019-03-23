---
title: Android WebView必知必会(3)-WebChromeClient
categories: 技术栈
tags:
  - Android
date: 2019-03-20 22:35:47
---

## 简介

😀WebChromeClient：当影响"浏览器"的事件到来时，就会通过WebChromeClient中的方法回调通知用法,所以WebChromeClient都是和我们平时使用浏览器需要的一些交互有密切的联系。

<!--more-->

## 获取网页的加载进度

- void  onProgressChanged(WebView view, int newProgress)  ；

  Tell the host application the current progress of loading a page. 

  newProgress: Current page loading progress, represented by an integer between 0 and 100.

  大家一定要注意，底层实现时，是利用handler来定时轮循当前进度的，每隔一定时间查询一次，所以每次拿到的进度数据是不一样的。也就是说如果页面较简单，可能会直接返回100，而跳过中间的各个数据。也就是说，除了100，其它任何一个数值不是一定返回的。所以大家如果要用到进度，除了数值100可以用等号来判断，其它一定要用大于号或小于号，如果用了等号，可能永远也不会执行到。



## 获取网页中的基本信息 

- void  onReceivedIcon(WebView view, Bitmap icon) 

  icon A Bitmap containing the favicon for the current page.网页中首部图标

- void  onReceivedTitle(WebView view, String title) 

  title: A String containing the new title of the document. 网页中标题中的更改，首部标题

  获取标题的时间主要取决于网页前端设置标题的位置，JS中的函数设置也在这里接收

- void  onReceivedTouchIconUrl(WebView view, String url, boolean precomposed)  Notify the host application of the url for an apple-touch-icon(苹果图标). 

- 苹果为iOS设备配备了apple-touch-icon私有属性，添加该属性，在iPhone,iPad,iTouch的safari浏览器上可以使用添加到主屏按钮将网站添加到主屏幕上，方便用户以后访问。apple-touch-icon 标签支持sizes属性，可以用来放置对应不同的设备。

  url: The icon url.图片地址

  precomposed: True if the url is for a precomposed touch icon. 标记为True的话使用原图



## 拦截网页中JS控制台消息 

当html中调用console相关输出的时候，就会通过onConsoleMessage进行通知

和alert,prompt,confirm不同，我们不需要强制设置WebChromeClient（但是仍需要setJavaScriptEnabled为true），当点击log按钮时，也会调用console相应的函数把日志打印出来。

可以获取到的信息有：

```java
@Override
public boolean onConsoleMessage(ConsoleMessage consoleMessage) {
	Log.e("ncmon", "onConsoleMessage : "
          + "\nmessage=" + consoleMessage.message()
          + "\nlineNumber=" + consoleMessage.lineNumber()
          + "\nmessageLevel=" + consoleMessage.messageLevel()
          + "\nsourceId=" + consoleMessage.sourceId());
    return super.onConsoleMessage(consoleMessage);
}
ncmon : onConsoleMessage : 
    message=Uncaught TypeError: Cannot read property 'getItem' of null
    lineNumber=2
    messageLevel=ERROR
    sourceId=https://m.baidu.com/?from=844b&vit=fps
chromium: [INFO:CONSOLE(2)] "Uncaught TypeError: Cannot read property 'getItem' of null", source: https://m.baidu.com/?from=844b&vit=fps (2)
```

可以看到除了楼主输出的日志外，还有chromium的日志，这里也就要体现返回值的作用，如果返回true时，就表示拦截了console输出，系统就不再通过console输出出来了；如果返回false(默认值)，则表示没有拦截console输出，调用系统默认处理

```java
public class ConsoleMessage {
    private MessageLevel mLevel;
    private String mMessage;
    private String mSourceId;
    private int mLineNumber;
}
```

## 拦截网页中JS弹框

三种类型的弹框

```java
public boolean onJsAlert(WebView view, String url, String message,
        JsResult result) {
    return false;
}
```

当网页调用alert()来弹出alert弹出框前回调，用以拦截alert()函数

```java
public boolean onJsConfirm(WebView view, String url, String message,
        JsResult result) {
    return false;
}
```

带有确定取消提示，当网页调用confirm()来弹出confirm弹出框前回调，用以拦截confirm()函数

```java
public boolean onJsPrompt(WebView view, String url, String message,
        String defaultValue, JsPromptResult result) {
    return false;
}
```

带有输入框的弹窗，当网页调用prompt()来弹出prompt弹出框前回调，用以拦截prompt()函数

- 处理JS弹框首先需要设置JavaScript支持，也必须设置WebChromeClient.

- 参数

  url:page_with_curl:：弹出对话框的网页网址

  message:speech_balloon::对话框的内容

  result:closed_umbrella:: 返回给JavaScript的响应，JsResult.confirm()表示点击了弹出框的确定按钮，JsResult.cancel()则表示点击了弹出框的取消按钮。<font color=red>其中JsPromptResult也是继承JsResult的，只是多了输入的内容变量 (String mStringResult)</font>

- 返回值 

  true Native 自行处理弹窗逻辑，不弹出alert弹窗了

  false WebView 自行弹出alert对话框处理

  如果是return true，此时我们必须手动调用JsResult的.confirm()或.cancel()方法，因为如果没有调用JsResult的confirm()或cancel()来告诉WebView你的处理结果，则WebView就会认为这个弹出框还一直弹在那里（虽然此时根本没有弹框弹出），所以之后你再点击alert按钮时，将会无效。这一点一定要注意。

## 打开和关闭Window

```java
public boolean onCreateWindow(WebView view, boolean isDialog,
        boolean isUserGesture, Message resultMsg) {
    return false;
}
```

这个方法回调的前提需要设置WebView多窗口支持，就是浏览器的多个tab

~~~ java
webSettings.setSupportMultipleWindows(true);//支持多窗口。如果设置为true
~~~

在Html中，超级链接标签a里有一个target属性，其意义是决定"是否在新窗口/标签页中打开链接"，如果不写target=”_blank”那么就是在相同的标签页打开，如果写了，就是在新的空白标签页中打开。比如：

```html
<a href="https://www.taobao.com/" title="淘宝" target="_blank">新窗口打开链接</a>
```

WebView默认是不支持target，点击上面的链接会在当前WebView中打开此链接。

完整逻辑如下

- 没有设置setSupportMultipleWindows属性为true

  - 没有setWebChromeClient：点击此链接会在当前WebView中打开此链接
  - 有setWebChromeClient：点击此链接会在当前WebView中打开此链接，不会回调onCreateWindow方法

- 有设置setSupportMultipleWindows属性为true

  - 没有setWebChromeClient：点击此链接不会在当前WebView中打开此链接

  - 有setWebChromeClient

    - 没有重写onCreateWindow方法：点击此链接不会在当前WebView中打开此链接，会回调onCreateWindow方法
    - 有重写onCreateWindow方法：点击此链接不会在当前WebView中打开此链接，会回调onCreateWindow方法，会在你新创建的WebView中打开此链接

```java
@Override
public boolean onCreateWindow(WebView webView, boolean isDialog, boolean isUserGesture, Message resultMsg) {
    Log.i("bqt", "【onCreateWindow】 " + isDialog + "  " + isUserGesture + "\n详细信息" + resultMsg.toString());
    if (activity != null) {
        WebView childView = new WebView(activity);//Parent WebView cannot host it's own popup window.
        childView.setBackgroundColor(Color.GREEN);
        childView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Log.i("bqt", "【shouldOverrideUrlLoading-子】");
                activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse(url)));
                return true;
            }
        });
        WebView.WebViewTransport transport = (WebView.WebViewTransport) resultMsg.obj;
        transport.setWebView(childView);//setWebView和getWebView两个方法
        resultMsg.sendToTarget();
        return true;
    } else return super.onCreateWindow(webView, isDialog, isUserGesture, resultMsg);//默认是returns false
}
```

感觉浏览器才会用到这个API,常规应用基本都是打开自己定制的Html页面

对应的还有一个关闭Window的API，在JS调用window.close()方法时会回调此方法【<button onclick="window.close()">关闭窗口</button>】

```java
public void onCloseWindow(WebView window) {}
```

## 文件选择器

这个方法在项目中很常用，Html中如果需要选择本机文件就需要把文件传过去。基本操作为打开文件管理器再把选择的文件传到网页中。我也有用到，就贴一下完整的代码片段。

```java
//For Android  >= 4.1
public void openFileChooser(ValueCallback<Uri> valueCallback, String acceptType, String capture) {
    uploadMessage = valueCallback;
    openImageChooserActivity();
}

// For Android >= 5.0
@Override
public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
    uploadMessageAboveL = filePathCallback;
    Intent i = new Intent(Intent.ACTION_GET_CONTENT);
    i.addCategory(Intent.CATEGORY_OPENABLE);
    if (fileChooserParams != null && fileChooserParams.getAcceptTypes() != null
                && fileChooserParams.getAcceptTypes().length > 0) {
		i.setType(fileChooserParams.getAcceptTypes()[0]);
    } else {
		i.setType("*/*");
    }
    startActivityForResult(Intent.createChooser(i, "File Chooser"), FILECHOOSER_RESULTCODE);
    return true;
}

 @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == FILE_CHOOSER_RESULT_CODE) {
            if (null == uploadMessage && null == uploadMessageAboveL) {
                return;
            }
            Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
            if (uploadMessageAboveL != null) {
                onActivityResultAboveL(requestCode, resultCode, data);
            } else if (uploadMessage != null) {
                uploadMessage.onReceiveValue(result);
                uploadMessage = null;
            }
        }
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void onActivityResultAboveL(int requestCode, int resultCode, Intent intent) {
        if (requestCode != FILE_CHOOSER_RESULT_CODE || uploadMessageAboveL == null) {
            return;
        }
        Uri[] results = null;
        if (resultCode == Activity.RESULT_OK) {
            if (intent != null) {
                String dataString = intent.getDataString();
                ClipData clipData = intent.getClipData();
                if (clipData != null) {
                    results = new Uri[clipData.getItemCount()];
                    for (int i = 0; i < clipData.getItemCount(); i++) {
                        ClipData.Item item = clipData.getItemAt(i);
                        results[i] = item.getUri();
                    }
                }
                if (dataString != null) {
                    results = new Uri[]{Uri.parse(dataString)};
                }
            }
        }
        uploadMessageAboveL.onReceiveValue(results);
        uploadMessageAboveL = null;
    }
```

@params filePathCallback:  提供要上传的文件的路径列表, or NULL to cancel. Must only be called if the showFileChooser implementations returns true.

@params fileChooserParams:  描述要打开的文件选择器的模式，以及与之一起使用的选项。

## 视频(全屏)播放

```java
public Bitmap getDefaultVideoPoster() {
    return null;
}
```

Html中，视频（video）控件在没有播放的时候将给用户展示一张“海报”图片（预览图）。其预览图是由Html中video标签的[poster](http://www.w3school.com.cn/tags/att_video_poster.asp)属性来指定的。如果开发者没有设置poster属性, 则可以通过这个方法来设置默认的预览图。

```html
<video controls poster="/images/w3school.gif">
   <source src="movie.mp4" type="video/mp4">
   <source src="movie.ogg" type="video/ogg">
   Your browser does not support the video tag.
</video> 
```

```java
public View getVideoLoadingProgressView() {
    return null;
}
```

播放视频时，在第一帧呈现之前，需要花一定的时间来进行数据缓冲。ChromeClient可以使用这个函数来提供一个在数据缓冲时显示的视图。 例如,ChromeClient可以在缓冲时显示一个转轮动画。

```java
public void onShowCustomView(View view, CustomViewCallback callback) {};
public void onHideCustomView() {}
```

通知主机应用webview需要显示一个custom view，主要是用在视频全屏 HTML5 Video support。

网页中有H5播放flash video的时候按下全屏按钮将会调用到这个方法，一般用作设置网页播放全屏操作。
## 获取地理位置 

```java
public void onGeolocationPermissionsShowPrompt(String origin, GeolocationPermissions.Callback callback) {}

public void onGeolocationPermissionsHidePrompt() {}
```

```java
public class GeolocationPermissions {
    /**
     * A callback interface used by the host application to set the Geolocation
     * permission state for an origin.
     */
    public interface Callback {
        /**
         * Sets the Geolocation permission state for the supplied origin.
         *
         * @param origin the origin for which permissions are set
         * @param allow whether or not the origin should be allowed to use the
         *              Geolocation API
         * @param retain whether the permission should be retained beyond the
         *               lifetime of a page currently being displayed by a
         *               WebView
         */
        public void invoke(String origin, boolean allow, boolean retain);
    };
}
```
JavaScript中有调用定位的API时会调用本方法

@param origin 来源，谁调用的，传回给谁

@param allow 是否允许本次定位

@param retain 是否保持允许定位状态，传递了true后，本WebView之后都可以获得当前位置信息。

对应的onGeolocationPermissionsHidePrompt（）就是通知应用程序，地理位置权限请求已被取消
## 请求权限

```java
    /**
     * Notify the host application that web content is requesting permission to
     * access the specified resources and the permission currently isn't granted
     * or denied. The host application must invoke {@link PermissionRequest#grant(String[])}
     * or {@link PermissionRequest#deny()}.
     *
     * If this method isn't overridden, the permission is denied.
     *
     * @param request the PermissionRequest from current web content.
     */
public void onPermissionRequest(PermissionRequest request) {
    request.deny();
}

/**
 * Notify the host application that the given permission request
 * has been canceled. Any related UI should therefore be hidden.
 *
 * @param request the PermissionRequest that needs be canceled.
 */
public void onPermissionRequestCanceled(PermissionRequest request) {}
```

```java
public abstract class PermissionRequest {
    public final static String RESOURCE_VIDEO_CAPTURE = "android.webkit.resource.VIDEO_CAPTURE";
    public final static String RESOURCE_AUDIO_CAPTURE = "android.webkit.resource.AUDIO_CAPTURE";
    public final static String RESOURCE_PROTECTED_MEDIA_ID =
            "android.webkit.resource.PROTECTED_MEDIA_ID";
    public final static String RESOURCE_MIDI_SYSEX = "android.webkit.resource.MIDI_SYSEX";
    public abstract Uri getOrigin();
    public abstract String[] getResources();
    public abstract void grant(String[] resources);
    public abstract void deny();
}
```
@method getorigin()获取来源

@method getResources()获取请求的权限列表

@method grant ()允许的权限列表

@method deny()拒绝了权限

需要的就是PermissRequest中定义的四个常亮代表的权限，关于权限的使用是其他知识点，大家可以看看这个[栗子](https://codeday.me/bug/20181125/420409.html)。
##  其他方法
```java
/**
 *获得所有访问历史项目的列表，用于链接着色。
 */
public void getVisitedHistory(ValueCallback<String[]> callback) {
}
/**
 *webview请求得到焦点，发生这个主要是当前webview不是前台状态，是后台webview。
 */
public void onRequestFocus(WebView view) {}
```
