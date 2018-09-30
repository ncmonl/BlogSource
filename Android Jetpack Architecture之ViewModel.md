---
title: Android Jetpack Architecture之ViewModel
date: 2018-09-26 20:40:11
categories: "技术栈"
tags:
  - Android
  - Jetpack
---
<br>&ensp;&ensp;Jetpack已经出了很久很久了，近几年的GDD几乎每次都会介绍新的组件，说来惭愧，一直没有好好学习，看近年的Google 的很多Demo中其实都有所体现，之前都是大概的了解了一遍。最近决定，好好梳理一遍，既学习其用法，也尝试学习下其设计思想。也是时候该补充一下了。进入正题--ViewModel</br>
<br>&ensp;&ensp;首先都是看官方的例子，[ViewModel][1]官方的的例子是会和另一个架构库LiveData写在一起，很多的博客也是照官方的例子来说明，开始接触时甚至给了我一种假象：ViewModel都是和LiveData一起使用的。后来阅读才了解，ViewModel和LiveData职责分工还是很明显的，使用LiveData Demo主要使用其observe功能，LiveDate的使用及原理之后再分析，甚至在appcompat-v7:27.1.1中直接单独集成了ViewModel.所以，故为排除干扰，今天不会使用官方的主流Demo用法，先来看ViewModel。</br>
<br>&ensp;&ensp;Android的UI控制器（Activity和Fragment）从创建到销毁拥有自己完整的生命周期，当系统配置发生改变时（(Configuration changes)），系统就会销毁Activity和与之关联的Fragment然后再次重建<font color=#FFA500>（可通过在AndroidManifast.xml中配置android:configChanges修改某些行为，这里不讨论）</font>,那么存储在当前UI中的临时数据也会被清空，例如，登陆输入框，输入账号或密码后旋转屏幕，视图被重建，输入过的数据也清空了，这无疑是一种不友好的用户体验。对于少量的可序列化数据可以使用onSaveInstanceState()方法保存然后在onCreate()方法中重新恢复，正如所说onSaveInstanceState对于大量的数据缓存有一定的局限性，大量的数据缓存则可以使用[Fragment][2].setRetainInstance(true)来保存数据。ViewModel也是提供了相同的功能，用来存储和管理与UI相关的数据，允许数据在系统配置变化后存活，我们一起看一下这个ViewModel的缓存是怎么实现的呢？</br>

<br>首先先看下使用方式，先上效果图</br>
![ViewMode Sample](http://omv6ufghj.bkt.clouddn.com/device-2018-09-27-224647_20180927224809.gif)
```java
public class MyViewModel extends ViewModel {
   	String name;
    public String getName(){
    	return name;
    }
    public void setName(String name){
    	this.name = name;
    }
    @Override
    protected void onCleared() {
    	super.onCleared();
    	name = null;
    }
}
```
&nbsp;
```java
public class ViewModelActivity extends AppCompatActivity implements View.OnClickListener {
    private static final String TAG = "ViewModelActivity";
    TextView textView;
    private MyViewModel myViewModel;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_viewmodel);
        textView = findViewById(R.id.textView);
        textView.setOnClickListener(this);
        ViewModelProvider.Factory factory = ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());
	/*
	*这里的this是ViewModelStoreOwner接口在appcompat-v7:27.1.1支持库中AppCompatActivity已经实现了，		
	*如果是较低版本，需要更新支持包或者参考其实现对本来继承的Activity做对应实现。
	*/
        ViewModelProvider provider = new ViewModelProvider(this, factory);//
        myViewModel = provider.get(MyViewModel.class);
        Log.e(TAG, "onCreate: " + myViewModel.getName() );
        if (myViewModel.getName() != null) {
            textView.setText(myViewModel.getName());
        }
    }
    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.textView:
                myViewModel.setName("MyViewModel Test");
                textView.setText(myViewModel.getName());
                break;
        }
    }
}
```
&nbsp;
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginBottom="8dp"
        android:text="default"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```
非常简单的一个例子，这就是ViewModel最简单的使用了，就是TextView中显示ViewModel的数据。ViewModel需要由ViewModelProvider.get(Class<T>)来取得，旋转屏幕销毁后，之前改变的数据还在。  
接下来就是进入主题分析下ViewModel到底是怎么实现的呢？  
带着问题看源码：  

- ViewModelProvider是干啥的？
- AndroidViewModelFactory 这命名一看就是应该是工厂模式，工厂创建了什么？
- provider.get(MyViewModel.class) 这里直接使用的get命名就得到了需要的唯一数据
- 注释中ViewModelStoreOwner又是什么角色？  
先看ViewModel类，没什么说的，就是一个么有任何真正实现的抽象类，只有一个抽象方法onCleared()

```java
  public abstract class ViewModel {
      /**
       * This method will be called when this ViewModel is no longer used and will be destroyed.
       * <p>
       * It is useful when ViewModel observes some data and you need to clear this subscription to
       * prevent a leak of this ViewModel.
       */
      @SuppressWarnings("WeakerAccess")
      protected void onCleared() {
      }
  }
```
接着看下ViewModelFactory，顾名思义就是制造ViewModel的。  
AndroidViewModelFactory的继承关系如下:
>android.arch.lifecycle.ViewModelProvider.Factory
>>android.arch.lifecycle.ViewModelProvider.NewInstanceFactory
>>>android.arch.lifecycle.ViewModelProvider.AndroidViewModelFactory

Factory是一个只包含一个create的interface，NewInstanceFactory实现了该方法传入Class<T>会利用ViewModel的默认无参构造器创建一个对应ViewModel的实例，而AndroidViewModelFactory增加了一个属性就是应用的Applicaion,同时重写create方法，查看ViewModel是否有包含Applicaion参数的构造方法从而使用，对应的其实还有一个AndroidViewModel是ViewModel的子类，默认已经实现了带有Application参数的构造方法，需要使用在ViewModel中使用application的直接继承AndroidViewModel就可以，看到这里其实最上面的例子有个不是问题的问题，其实上面的Factory直接使用NewInstanceFactory就可以创建出对应的ViewModel实例了。
```java
	/**
	 * Implementations of {@code Factory} interface are responsible to instantiate ViewModels.
	 */
	public interface Factory {
	    /**
	     * Creates a new instance of the given {@code Class}.
	     * <p>
	     *
	     * @param modelClass a {@code Class} whose instance is requested
	     * @param <T>        The type parameter for the ViewModel.
	     * @return a newly created ViewModel
	     */
	    @NonNull
	    <T extends ViewModel> T create(@NonNull Class<T> modelClass);
	}

```
```java
/**
 * Simple factory, which calls empty constructor on the give class.
 */
public static class NewInstanceFactory implements Factory {

    @SuppressWarnings("ClassNewInstance")
    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
        //noinspection TryWithIdenticalCatches
        try {
            return modelClass.newInstance();
        } catch (InstantiationException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        }
    }
}
```
```java
/**
 * {@link Factory} which may create {@link AndroidViewModel} and
 * {@link ViewModel}, which have an empty constructor.
 */
public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

    private static AndroidViewModelFactory sInstance;

    /**
     * Retrieve a singleton instance of AndroidViewModelFactory.
     *
     * @param application an application to pass in {@link AndroidViewModel}
     * @return A valid {@link AndroidViewModelFactory}
     */
    @NonNull
    public static AndroidViewModelFactory getInstance(@NonNull Application application) {
        if (sInstance == null) {
            sInstance = new AndroidViewModelFactory(application);
        }
        return sInstance;
    }

    private Application mApplication;

    /**
     * Creates a {@code AndroidViewModelFactory}
     *
     * @param application an application to pass in {@link AndroidViewModel}
     */
    public AndroidViewModelFactory(@NonNull Application application) {
        mApplication = application;
    }

    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
        if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
            //noinspection TryWithIdenticalCatches
            try {
                return modelClass.getConstructor(Application.class).newInstance(mApplication);
            } catch (NoSuchMethodException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InvocationTargetException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
        return super.create(modelClass);
    }
}
```

之后通过ViewModelStoreOwner和刚刚创建的Factory创建出ViewModelPrivider实例
```java
/**
 * Creates {@code ViewModelProvider}, which will create {@code ViewModels} via the given
 * {@code Factory} and retain them in a store of the given {@code ViewModelStoreOwner}.
 *
 * @param owner   a {@code ViewModelStoreOwner} whose {@link ViewModelStore} will be used to
 *                retain {@code ViewModels}
 * @param factory a {@code Factory} which will be used to instantiate
 *                new {@code ViewModels}
 */
public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
    this(owner.getViewModelStore(), factory);
}

/**
 * Creates {@code ViewModelProvider}, which will create {@code ViewModels} via the given
 * {@code Factory} and retain them in the given {@code store}.
 *
 * @param store   {@code ViewModelStore} where ViewModels will be stored.
 * @param factory factory a {@code Factory} which will be used to instantiate
 *                new {@code ViewModels}
 */
public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
    mFactory = factory;
    this.mViewModelStore = store;
}
```
```java
/**
 * A scope that owns {@link ViewModelStore}.
 * <p>
 * A responsibility of an implementation of this interface is to retain owned ViewModelStore
 * during the configuration changes and call {@link ViewModelStore#clear()}, when this scope is
 * going to be destroyed.
 */
@SuppressWarnings("WeakerAccess")
public interface ViewModelStoreOwner {
    /**
     * Returns owned {@link ViewModelStore}
     *
     * @return a {@code ViewModelStore}
     */
    @NonNull
    ViewModelStore getViewModelStore();
}
```
ViewModelStoreOwner 也是一个接口是FragmentActivity实现了该接口并实现了其中的getViewModelStore()方法
```java
public class FragmentActivity extends BaseFragmentActivityApi16 implements
        ViewModelStoreOwner...{
    private ViewModelStore mViewModelStore;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        mFragments.attachHost(null /*parent*/);

        super.onCreate(savedInstanceState);

        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            mViewModelStore = nc.viewModelStore;
    }
    /**
     * Returns the {@link ViewModelStore} associated with this activity
     *
     * @return a {@code ViewModelStore}
     */
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
        return mViewModelStore;
    }
}
```
这个ViewModelStore又是什么呢，其实就是真正利用HashMap存储ViewModel的地方了，看下代码在存储和clear同时会调用ViewModel需要实现的抽象方法onClear()
```java
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.onCleared();
        }
        mMap.clear();
    }
}
```
这样ViewModelProvider就是有了一个ViewModel的容器，这时去调用ViewModelProvider的get(Class<T>)方法就是去调用mViewModelStore
的get()方法取出对应的ViewModel所以这里只要持有的ViewModelStore是有缓存的，那么取出的ViewModel就是相同的缓存了。
```java
/**
 * Returns an existing ViewModel or creates a new one in the scope (usually, a fragment or
 * an activity), associated with this {@code ViewModelProvider}.
 * <p>
 * The created ViewModel is associated with the given scope and will be retained
 * as long as the scope is alive (e.g. if it is an activity, until it is
 * finished or process is killed).
 *
 * @param modelClass The class of the ViewModel to create an instance of it if it is not
 *                   present.
 * @param <T>        The type parameter for the ViewModel.
 * @return A ViewModel that is an instance of the given type {@code T}.
 */
@NonNull
@MainThread
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}

@NonNull
@MainThread
public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
    ViewModel viewModel = mViewModelStore.get(key);

    if (modelClass.isInstance(viewModel)) {
        //noinspection unchecked
        return (T) viewModel;
    } else {
        //noinspection StatementWithEmptyBody
        if (viewModel != null) {
            // TODO: log a warning.
        }
    }

    viewModel = mFactory.create(modelClass);
    mViewModelStore.put(key, viewModel);
    //noinspection unchecked
    return (T) viewModel;
}

```
看到这里就会发现ViewModelStore的缓存其实是通过NonConfigurationInstances的缓存来实现的，这样就完成了Activity销毁重建后ViewModel还保存原来的数据的过程，那么NonConfigurationInstances 是什么呢？如果有了解过使用在Activity中使用onRetainNonConfigurationInstance()保存缓存数据，在onCreate()中通过getLastNonConfigurationInstance()恢复之前的数据状态的同学可能会很熟悉这里的写法，是的，这里FragmentActivity就是使用的这种方式来保存之前的ViewModelStore,看下FragmentActivity的onRetainNonConfigurationInstance()方法。

```java
    /**
     * Retain all appropriate fragment state.  You can NOT
     * override this yourself!  Use {@link #onRetainCustomNonConfigurationInstance()}
     * if you want to retain your own state.
     */
    @Override
    public final Object onRetainNonConfigurationInstance() {
        if (mStopped) {
            doReallyStop(true);
        }

        Object custom = onRetainCustomNonConfigurationInstance();

        FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

        if (fragments == null && mViewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        nci.viewModelStore = mViewModelStore;//就是这里了，会把之前的VeiwmodelStroe存储到NonConfigurationInstances中以供后续恢复使用
        nci.fragments = fragments;
        return nci;
    }
```

这里其实再次出现了一个问题  onRetainNonConfigurationInstance()和getLastNonConfigurationInstance()又是怎么恢复数据呢?...这个其实和Activity的启动流程相关，这里也介绍一下吧，之后的内容其实是Activity的内容了，趁这次看ViwModel也跟着看了一遍，有了解过Activity启动流程的同学更容易理解的多，大家酌情观看。

也不能从头开始说起，再从头就要越扯越远了，就从ActivityThread.java中的scheduleLaunchActivity开始
```java
        @Override
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
                ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
                CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
                int procState, Bundle state, PersistableBundle persistentState,
                List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
                boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

            updateProcessState(procState, false);

            ActivityClientRecord r = new ActivityClientRecord();

            r.token = token;
            r.ident = ident;
            r.intent = intent;
            r.referrer = referrer;
            r.voiceInteractor = voiceInteractor;
            r.activityInfo = info;
            r.compatInfo = compatInfo;
            r.state = state;
            r.persistentState = persistentState;

            r.pendingResults = pendingResults;
            r.pendingIntents = pendingNewIntents;

            r.startsNotResumed = notResumed;
            r.isForward = isForward;

            r.profilerInfo = profilerInfo;

            r.overrideConfig = overrideConfig;
            updatePendingConfiguration(curConfig);

            sendMessage(H.LAUNCH_ACTIVITY, r);
        }
```
从ActivityThread.java中H（extents Handler）接收到LAUNCH_ACTIVITY，并且会接收ActivityClientRecord，其中会调用ActivityThread的handleLaunchActivity方法：
```java
	//ActivityThread.java
	//没有前后文的H中的handleMessage~~~
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;
			//ActivityClientRecord 是apk进程中一个Activity的代表，这个对象的activity成员引用真正的Activity组件,后面的都和它有关系
                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");///这里~这里~
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
```
```java
    private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
            ...
            Activity a = performLaunchActivity(r, customIntent);
	    ...
    }

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    ...
        if (activity != null) {
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (r.overrideConfig != null) {
                    config.updateFrom(r.overrideConfig);
                }
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config, //看到这个r.lastNonConfigurationInstances 就是在Activity方法中调用getLastNonConfigurationInstance()获取到的Object了。
                        r.referrer, r.voiceInteractor, window, r.configCallback);
    ...
    }
```
注释中的地方就是lastNonConfigurationInstances的赋值的地方，可能会发现在scheduleLaunchActivity并没有对lastNonConfigurationInstances赋值，因为第一次启动Activity时，这里其实就是null的，那么赋值的地方在哪里呢，既然是销毁后会恢复数据，追踪发现在performDestroyActivity()也就是在调用onDestroy生命周期之前有这样一段代码
```java
private ActivityClientRecord performDestroyActivity(IBinder token, boolean finishing,
            int configChanges, boolean getNonConfigInstance) {
        ActivityClientRecord r = mActivities.get(token);
        ...无关代码省略
            if (getNonConfigInstance) {
                try {
                    r.lastNonConfigurationInstances
                            = r.activity.retainNonConfigurationInstances();///就是这里出现了想要找的NonConfigurationInstances
                } catch (Exception e) {
                    if (!mInstrumentation.onException(r.activity, e)) {
                        throw new RuntimeException(
                                "Unable to retain activity "
                                + r.intent.getComponent().toShortString()
                                + ": " + e.toString(), e);
                    }
                }
            }
            try {
                r.activity.mCalled = false;
                mInstrumentation.callActivityOnDestroy(r.activity);
	...无关代码省略
        return r;
    }
```
在performDestroyActivity()调用了Activity.retainNonConfigurationInstances()方法了，所以逻辑切换回Activity中...
```java
     /**
     * This method is similar to {@link #onRetainNonConfigurationInstance()} except that
     * it should return either a mapping from  child activity id strings to arbitrary objects,
     * or null.  This method is intended to be used by Activity framework subclasses that control a
     * set of child activities, such as ActivityGroup.  The same guarantees and restrictions apply
     * as for {@link #onRetainNonConfigurationInstance()}.  The default implementation returns null.
     */
    @Nullable
    HashMap<String,Object> onRetainNonConfigurationChildInstances() {
        return null;
    }

    NonConfigurationInstances retainNonConfigurationInstances() {
        Object activity = onRetainNonConfigurationInstance();///熟悉的代码，原来的配方，和分析ActivityThread之前联系起来了，在Activity中是空实现，这里就是获取子类的NonConfigurationInstance()，之前的例子就是的得FragmentActivity中的具体实现，上文中已经在分析ActivityThread.java已经指出。
        HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
        FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

        // We're already stopped but we've been asked to retain.
        // Our fragments are taken care of but we need to mark the loaders for retention.
        // In order to do this correctly we need to restart the loaders first before
        // handing them off to the next activity.
        mFragments.doLoaderStart();
        mFragments.doLoaderStop(true);
        ArrayMap<String, LoaderManager> loaders = mFragments.retainLoaderNonConfig();

        if (activity == null && children == null && fragments == null && loaders == null
                && mVoiceInteractor == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.activity = activity;
        nci.children = children;
        nci.fragments = fragments;
        nci.loaders = loaders;
        if (mVoiceInteractor != null) {
            mVoiceInteractor.retainInstance();
            nci.voiceInteractor = mVoiceInteractor;
        }
        return nci;//这里返回的是Activity中的NonConfigurationInstances就保存在了ActivityClientRecord中了
    }
```
至此，ActivityClientRecord就不再深入了，可以看到在Activity中是以一个ArrayMap来保存Activity的记录，记录的就是Activity的状态，所以这里就实现了对NonConfigurationInstances的保存。

---

**结语：至此就基本看完了ViewModel在Activity中的使用和原理，在Fragment中的实现主要是使用setRetainInstance(true)的方式去保存，跟今天的分析也有关联，分析源码的过程总是看着就有新的问题，再次带着问题去解决会再次有不同的收获，本文的理解也可能有偏差，如有错误和想要交流的也欢迎指正沟通。**  

[1]: https://developer.android.google.cn/topic/libraries/architecture/viewmodel        "Jetpack-ViewModel"
[2]: https://developer.android.com/reference/android/support/v4/app/Fragment           "Fragment-reference"
