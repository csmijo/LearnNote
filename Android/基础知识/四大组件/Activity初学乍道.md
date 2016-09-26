#Activity初学乍道

`Android` `Activity`

--

##1. Activity的概念和生命周期图

![Activity的概念和生命周期图](http://www.runoob.com/wp-content/uploads/2015/08/18364230.jpg "Activity的概念和生命周期图")

>`注意：`AlertDialog和PopWindow 不会触发onPause()和onStop()方法

##2. Activity/ActionBarActivity/AppCompatActivity的区别：
　　
`Activity`就不用说啦，后面这两个都是为了低版本兼容而提出的提出来的，他们都在v7包下， ~~ActionBarActivity~~已被废弃，从名字就知道，ActionBar~，而在`5.0`后，被Google弃用了，现在用 ToolBar...而我们现在在Android Studio创建一个`Activity`默认继承的会是：**`AppCompatActivity`**! 当然你也可以只写`Activity`，不过AppCompatActivity给我们提供了一些新的东西而已！ 

##3. 横竖屏切换与状态保存的问题

　　App横竖屏切换的时候会销毁当前的`Activity`然后重新创建一个，你可以自行在生命周期 的每个方法里都添加打印Log的语句，来进行判断，又或者设一个按钮一个TextView点击按钮后，修改TextView 文本，然后横竖屏切换，会神奇的发现TextView文本变回之前的内容了！ 横竖屏切换时Act走下述生命周期：
>**onPause-> onStop-> onDestory-> onCreate->onStart->onResume**

###关于横竖屏切换可能遇到下述问题：

**1. 禁止屏幕横竖屏自动切换**

　　很简单，在`AndroidManifest.xml`中为`Activity`添加一个属性： `android:screenOrientation`， 有下述可选值：

>* `unspecified`:**默认值**, 由系统来判断显示方向.判定的策略是和设备相关的，所以不同的设备会有不同的显示方向。
* `landscape`:横屏显示（宽比高要长）
* `portrait`:竖屏显示(高比宽要长)
* `user`:用户当前首选的方向
* `behind`:和该Activity下面的那个Activity的方向一致(在Activity堆栈中的)
* `sensor`:有物理的感应器来决定。如果用户旋转设备这屏幕会横竖屏切换。
* `nosensor`:忽略物理感应器，这样就不会随着用户旋转设备而更改了（"unspecified"设置除外）。

**2. 横竖屏时想加载不同的布局**

　　**1）准备两套不同的布局**

　　Android会自己根据横竖屏加载不同布局： 创建两个布局文件夹：layout-land横屏,layout-port竖屏 然后把这两套布局文件丢这两文件夹里，文件名一样，Android就会自行判断，然后加载相应布局了！

　　**2）自己在代码中进行判断**

　　自己想加载什么就加载什么：我们一般是在onCreate()方法中加载布局文件的，我们可以在这里对横竖屏的状态做下判断，关键代码如下：
>```android
if (this.getResources().getConfiguration().orientation == Configuration.ORIENTATION_LANDSCAPE){  
     setContentView(R.layout.横屏);
}else if (this.getResources().getConfiguration().orientation ==Configuration.ORIENTATION_PORTRAIT) {  
    setContentView(R.layout.竖屏);
}
```

**3. 状态保存问题**

　　通过一个`Bundle savedInstanceState`参数即可完成！ 三个核心方法：
>```android
onCreate(Bundle savedInstanceState);
onSaveInstanceState(Bundle outState);
onRestoreInstanceState(Bundle savedInstanceState);
```

　　你只重写`onSaveInstanceState()`方法，往这个bundle中写入数据，比如：
> savedInstanceState.putInt("num",1);

这样，然后你在`onCreate`或者`onRestoreInstanceState`中就可以拿出里面存储的数据，不过拿之前要判断下是否为`null`哦！
> savedInstanceState.getInt("num");

##4. Activity件的数据传递

![Activity件的数据传递](http://www.runoob.com/wp-content/uploads/2015/08/7185831.jpg "Activity件的数据传递")

**`注意`： 使用Bundle传递数据时，Bundle的大小是有限制的，要求`< 0.5MB`，如果大于这个值，就会报`TransactionTooLargeException`异常的。**

##5. 多个Activity之间的交互

1. 使用`startActivityForResult(Intent intent,int requestCode)`来启动一个`Activity`，例如在A中使用该方法启动B
2. 在启动的`Activity`中重写`onActivityResult(int requestCode,int resultCode,Intent data)`,例如在A中重写该方法。其中`requestCode`是用于区分在该`Activity`中不同的启动方式，比如两个不同按钮启动同一个`Activity`,传递的数据可能不同，这里就可以用这个`requestCode`来区别`resultCode`；子`Activity`通过`setResult()`返回的码
3. 在子Activity中重写 `setResult(int resultCode,Intent data)`

##6. 设置Activity全屏的方法：

**1.代码隐藏ActionBar**

　　在`Activity`的`onCreate()`方法中调用`getActionBar.hide()`即可

**2.通过`requestWindowFeature`设置**
　　
>`requestWindowFeature(Window.FEATURE_NO_TITLE); `

　　**该代码需要在setContentView ()之前调用，不然会报错！！！**

**3.通过AndroidManifest.xml的theme**

　　在需要全屏的Activity的标签内设置
>`theme = @android:style/Theme.NoTitleBar.FullScreen`

##7. onWindowFocusChanged方法妙用：

我们先来看下官方对这个方法的介绍：

![onWindowFocusChanged](http://www.runoob.com/wp-content/uploads/2015/08/17084157.jpg "")

　　就是，当`Activity`得到或者失去焦点的时候，就会回调该方法！如果我们想监控`Activity`是否加载完毕，就可以用到这个方法了~ 想深入了解的可移步到这篇文章： [onWindowFocusChanged触发简介](http://blog.csdn.net/yueqinglkong/article/details/44981449)

##8. Activity，Window与View的关系

![Activity，Window与View的关系](http://www.runoob.com/wp-content/uploads/2015/08/93497523.jpg "")

**流程解析：** Activity调用startActivity后最后会调用attach方法，然后在PolicyManager实现一个Ipolicy接口，接着实现一个Policy对象，接着调用makenewwindow(Context)方法，该方法会返回一个PhoneWindow对象，而PhoneWindow 是Window的子类，在这个PhoneWindow中有一个DecorView的内部类，是所有应用窗口的根View，即View的老大， 直接控制Activity是否显示(引用老司机原话..)，好吧，接着里面有一个LinearLayout，里面又有两个FrameLayout他们分别拿来装ActionBar和CustomView，**而我们setContentView()加载的布局就放到这个CustomView中**！

##9. Activity的四种加载模式详解

![Activity四种加载模式](http://www.runoob.com/wp-content/uploads/2015/08/50179298.jpg "")

**1. standard模式**

　　标准启动模式，也是activity的默认启动模式。在这种模式下启动的activity可以被多次实例化，即在同一个任务中可以存在多个activity的实例，每个实例都会处理一个Intent对象。如果Activity A的启动模式为standard，并且A已经启动，在A中再次启动Activity A，即调用startActivity（new Intent（this，A.class）），会在A的上面再次启动一个A的实例，即当前的桟中的状态为A-->A。

![standard-1](http://www.jcodecraeer.com/uploads/20150520/1432087372621766.jpg "")

![standard-2](http://www.jcodecraeer.com/uploads/20150520/1432087374604123.jpg "")

**2. singleTop模式**

　　如果一个以singleTop模式启动的Activity的实例已经存在于任务栈的栈顶， 那么再启动这个Activity时，不会创建新的实例，而是重用位于栈顶的那个实例， 并且会调用该实例的**onNewIntent()**方法将Intent对象传递到这个实例中。 举例来说，如果A的启动模式为singleTop，并且A的一个实例已经存在于栈顶中， 那么再调用startActivity（new Intent（this，A.class））启动A时， 不会再次创建A的实例，而是重用原来的实例，并且调用原来实例的onNewIntent()方法。 这时任务栈中还是这有一个A的实例。如果以singleTop模式启动的activity的一个实例 已经存在与任务栈中，但是不在栈顶，那么它的行为和standard模式相同，也会创建多个实例。

![singleTop](http://www.jcodecraeer.com/uploads/20150520/1432087389219419.jpg "")

**3. singleTask模式**

　　只允许在系统中有一个Activity实例。如果系统中已经有了一个实例， 持有这个实例的任务将移动到顶部，同时intent将被通过**onNewIntent()**发送。 如果没有，则会创建一个新的Activity并置放在合适的任务中。

![singleTask](http://www.jcodecraeer.com/uploads/20150520/1432087394416117.jpg "")

**4. singleInstance模式**

　　保证系统无论从哪个Task启动Activity都只会创建一个Activity实例,并将它加入新的Task栈顶，也就是说被该实例启动的其他activity会自动运行于另一个Task中。 当再次启动该activity的实例时，会重用已存在的任务和实例。并且会调用这个实例的onNewIntent()方法，将Intent实例传递到该实例中。和singleTask相同，同一时刻在系统中只会存在一个这样的Activity实例。

![singleInstance](http://www.jcodecraeer.com/uploads/20150520/1432087655129646.jpg "")

--
参考文档：
* [Activity初学乍练](http://www.runoob.com/w3cnote/android-tutorial-activity.html)
* [Activity初窥门径](http://www.runoob.com/w3cnote/android-tutorial-activity-start.html "")
* [Activity登堂入室](http://www.runoob.com/w3cnote/android-tutorial-activity-intro.html "")