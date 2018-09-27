<br>&ensp;&ensp;Jetpack已经出了很久很久了，近几年的GDD几乎每次都会介绍新的组件，说来惭愧，一直没有好好学习，看近年的Google 的很多Demo中其实都有所体现，之前都是大概的了解了一遍。最近决定，好好梳理一遍，既学习其用法，也尝试学习下其设计思想。也是时候该补充一下了。进入正题--ViewModel</br>
<br>&ensp;&ensp;首先都是看官方的例子，[ViewModel][1]官方的的例子是会和另一个架构库LiveData写在一起，很多的博客也是照官方的例子来说明，开始接触时甚至给了我一种假象：ViewModel都是和LiveData一起使用的。后来阅读才了解，ViewModel和LiveData职责分工还是很明显的，使用LiveData Demo主要使用其observe功能，LiveDate的使用及原理之后再分析，甚至在appcompat-v7:27.1.1中直接单独集成了ViewModel.所以，故为排除干扰，今天不会使用官方的主流Demo用法，先来看ViewModel。</br>
<br>&ensp;&ensp;Android的UI控制器（Activity和Fragment）从创建到销毁拥有自己完整的生命周期，当系统配置发生改变时（(Configuration changes)），系统就会销毁Activity和与之关联的Fragment然后再次重建<font color=#FFA500>（可通过在AndroidManifast.xml中配置android:configChanges修改某些行为，这里不讨论）</font>,那么存储在当前UI中的临时数据也会被清空，例如，登陆输入框，输入账号或密码后旋转屏幕，视图被重建，输入过的数据也清空了，这无疑是一种不友好的用户体验。对于少量的可序列化数据可以使用onSaveInstanceState()方法保存然后在onCreate()方法中重新恢复，正如所说onSaveInstanceState有一定的局限性，大量的数据缓存则可以使用[Fragment][2].setRetainInstance(true)来保存数据。ViewModel也是提供了相同的功能，用来存储和管理与UI相关的数据，允许数据在系统配置变化后存活，其内部原理其实也是使用Fragment.setRetainInstance(true)来实现，这个后面会分析</br>

<br>首先先看下使用方式，先上效果图</br>
![ViewMode Sample](http://omv6ufghj.bkt.clouddn.com/device-2018-09-27-224647_20180927224809.gif)
<pre>
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
</pre>
&nbsp;
<pre>

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
</pre>

&nbsp;

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

<br>非常简单的一个例子，这就是ViewModel最简单的使用了，就是TextView中显示ViewModel的数据。ViewModel需要由ViewModelProvider.get(Class<T>)来取得，旋转屏幕销毁后，之前改变的数据还在。  
接下来就是进入主题分析下ViewModel到底是怎么实现的呢？  
带着问题看源码：  

- ViewModelProvider是干啥的？
- AndroidViewModelFactory 这命名一看就是应该是工厂模式，工厂创建了什么？
- provider.get(MyViewModel.class) 这里直接使用的get命名就得到了需要的唯一数据
- 注释中ViewModelStoreOwner又是什么角色？
</br>

[1]: https://developer.android.google.cn/topic/libraries/architecture/viewmodel        "Jetpack-ViewModel" 
[2]: https://developer.android.com/reference/android/support/v4/app/Fragment           "Fragment-reference"
