## 应用弹窗“此应用专为旧版Android打造，因此可能无法正常运行

安装在新设备上的应用出现了这个警告弹窗：

![targetsdk_error_dialog](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/targetsdk_error_dialog.png)

显然是个系统弹窗。看一下原因

### Android源码分析

追踪应用启动代码，忽略过程，本文不是重点，经过层层调用会进入到AppWarnings方法中，直接去看关键信息：

```java
//com/android/server/wm/AppWarnings.java
    /**
     * Shows the "deprecated target sdk" warning, if necessary.
     *
     * @param r activity record for which the warning may be displayed
     */
    public void showDeprecatedTargetDialogIfNeeded(ActivityRecord r) {
        if (r.appInfo.targetSdkVersion < Build.VERSION.MIN_SUPPORTED_TARGET_SDK_INT) {
            mUiHandler.showDeprecatedTargetDialog(r);
        }
    }
    /**
     * Called when an activity is being started.
     *
     * @param r record for the activity being started
     */
    public void onStartActivity(ActivityRecord r) {
        showUnsupportedCompileSdkDialogIfNeeded(r);
        showUnsupportedDisplaySizeDialogIfNeeded(r);
        showDeprecatedTargetDialogIfNeeded(r);
    }
```
可以看到该弹窗的显示位置showDeprecatedTargetDialogIfNeeded(ActivityRecord)只有一个条件：targetSdkVersion<Build.VERSION.MIN_SUPPORTED_TARGET_SDK_INT
继续追踪Build.java

```java
//android/os/Build.java
        /**
         * The current lowest supported value of app target SDK. Applications targeting
         * lower values may not function on devices running this SDK version. Its possible
         * values are defined in {@link Build.VERSION_CODES}.
         *
         * @hide
         */
        public static final int MIN_SUPPORTED_TARGET_SDK_INT = SystemProperties.getInt(
                "ro.build.version.min_supported_target_sdk", 0);
```
是设备中配置信息ro.build.version.min_supported_target_sdk
目前发现在Android P 和Android Q中会出现该问题，查看测试机型中该配置的值
Android Q设备中该值：

![targetsdk_error_Q](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/targetsdk_error_Q.png)

Android P设备中该值：

![targetsdk_error_P](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/targetsdk_error_P.png)

这里有一个问题，既然是在每个Activity启动时会调用AppWarnings.java的onStartActivity方法，**那会不会每次打开新Activity，都弹此对话框？**

答案：不会的，这里有一个小技巧，第一次弹出对话框后，用户如果选择“确定”，AMS会给此应用设置一个Flag标识：FLAG_HIDE_DEPRECATED_SDK。每次准备弹窗时，会先判断此标识值是否为true，如果是，说明已经提示过用户，无需再弹窗。代码如下：

```java
public DeprecatedTargetSdkVersionDialog(final AppWarnings manager, Context context, ApplicationInfo appInfo) {
	// ...
	final AlertDialog.Builder builder = new AlertDialog.Builder(context)
	.setPositiveButton(R.string.ok, (dialog, which) ->
	manager.setPackageFlag(mPackageName, AppWarnings.FLAG_HIDE_DEPRECATED_SDK, true))
	.setMessage(message)
	.setTitle(label);
	// ...
}
```

### 问题结论

**结论很清晰：文章开头中提到的弹窗，在Android P设备中targetSdkVersion<17的应用会弹出该弹窗，在Android Q设备中targetSdkVersion<23的应用会弹出该弹窗。**
