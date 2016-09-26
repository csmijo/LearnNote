#Activity状态恢复的方法

`Android`

--

## 1. Activity状态恢复

重写Activity的三个方法：
*  onSaveInstanceState(Bundle outState)   
*  onCreate()
*  onRestoreInstanceState()

### 1. onSaveInstanceState(Bundle outState)

1. 方法调用的时机：
    * 系统即将触发Activity重构前调用
    * 资源不足，系统认为这个Activity有可能被回收时会被调用
    * 参数outState，由系统负责维护和传入，用来存放需要保持的数据
    
2. 要点
    * outState是一种key，value的形式

##2. 禁止Activity状态恢复

修改android:configChanges属性：
*  如果在manifest中为activity指定了该属性，那么属性中罗列的配置变化将不再导致activity的重构
* 会触发activity的**`onConfigurationChanged()`**方法的回调
* 但是这也会使得在语言切换等情况下activity并不主动改变自己的语言

##3. 做好重构的准备
* 即便通过configChanges属性禁止了重构，也无法阻止在内存不足时activity被回收
* 所以必须考虑被重构时的逻辑
* 可以通过config改变来模拟这种重构

##4. Android横竖屏切换小节
结合网上的整理，小结跟这几配置相关的情景：

1. **不设置Activity的android:configChanges时**，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次（我在三星4.0设备上发现切横屏和竖屏都是执行一次，而并非这里说的有执行两次的情况，不知道是否以前版本手机会这样？）；
2. **设置Activity的android:configChanges="orientation"时**，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次；
3. **设置Activity的android:configChanges="orientation|keyboardHidden"时**，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法。

**注意：** 、
* **上述描述是在Android3.2以前**，如果缺少了**keyboardHidden**选项，不能防止Activity的销毁重启，也就不能执行onConfigurationChanged方法了。
* **在Android3.2之后**，必须加上**screenSize**属性才可以屏蔽调用Activity的生命周期（我在一些设备上亲测可以不需要keyboardHidden，只要screenSize就可以了，但是保险起见还是继续保留keyboardHidden吧）。



##5. android:screenOrientation属性值

　　缺省情况下，当手机没有关闭横竖屏切换功能时，系统一旦触发横竖屏切换，当前活动的App界面就会进行横竖屏切换，由于横竖屏界面的尺寸等参数不同，所以就要对横竖屏进行不同的适配。有的开发者为了避免不必要的麻烦，就让App禁止横竖屏，这就需要在`AndroidManifest.xml`中设置`activity`的`android:screenOrientation`属性来实现。

　　下面来介绍一下`android:screenOrientation`的属性值：

|属性值|说明|
|:--|:--|
|**unspecified**|默认值，由系统来判断显示方向。判定的策略和设备有关，所以不同的设备会有不同的显示方向|
|**landscape**|横屏显示(宽度 > 高度)|
|**portrait**|竖屏显示(宽度 < 高度)|
|**user**|用户当前首选的方向|
|**behind**|和该Activity下的那个Activity的方向保持一致(在Activity堆栈中)|
|**sensor**|由物理的感应器来决定。如果用户旋转设备将会导致屏幕横竖屏切换|
|**nosensor**|忽略无力感应器，这样就不会随着用户旋转设备二更改了("unspecified"除外)|

##6. 重启Activity的横竖屏切换

　　缺省情況下，Activity每次横竖屏切换都会重新调用一轮`onPause--> onStop--> onDestory--> onCreate--> onStart--> onResume`操作，从而销毁原来的Activity对象，创建新的Activity对象。这是因为通常情况下，横竖屏之间切换，界面的宽高会发生转换，从而可能要求不同的布局。

　　具体的布局切换可以通过如下两种方式来实现：
* 在`res`目录下建立`layout-land`和`land-port`目录，相应的layout文件名不变。`layout-land`是横屏的layout；`layout-port`是竖屏的layout。横竖屏切换时系统自己会调用Activity的`onCreate`方法，从而根据当前的屏幕方向自动加载相应的布局文件。
* 也可以在代码中判断当前的横竖屏情况，然后加载相应的布局文件。比如：
```java
@Override
protected void onCreate(Bundle icicle) {
 super.onCreate(icicle);

 int mCurrentOrientation = getResources().getConfiguration().orientation;
 if ( mCurrentOrientation == Configuration.ORIENTATION_PORTRAIT ) {
         // If current screen is portrait
         Log.i("info", "portrait"); /* 竖屏 */
         setContentView(R.layout.mainP);
 } else if ( mCurrentOrientation == Configuration.ORIENTATION_LANDSCAPE ) {
         //If current screen is landscape
         Log.i("info", "landscape"); // 横屏
         setContentView(R.layout.mainL);
 }
 init();//初始化，赋值等操作
 findViews();//获得控件
}
``` 

　　重启Activity在未加处理的情况下必然导致数据的丢失和重新获取，这样用户体验非常差。为此就需要在切换前对数据进行保存，切换重启后对数据进行恢复。
###1. onSaveInstanceState(Bundle outState)
　　当您的Activity开始停止时，系统会调用`onSaveInstanceState()`以便您的Activity可以保存带有键值对集合的状态信息。 此方法的默认实现保存有关Activity视图层次的状态信息，例如 EditText 小工具中的文本或ListView 的滚动位置。

　　要保存Activity的更多状态信息，您必须实现`onSaveInstanceState()`并将键值对添加至 Bundle 对象。 例如：
```java
static final String STATE_SCORE = "playerScore";
static final String STATE_LEVEL = "playerLevel";
...

@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // Save the user's current game state
    savedInstanceState.putInt(STATE_SCORE, mCurrentScore);
    savedInstanceState.putInt(STATE_LEVEL, mCurrentLevel);

    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(savedInstanceState);
}
```
　　当您的Activity在先前销毁之后重新创建时，您可以从系统向Activity传递的 Bundle 恢复已保存的状态。`onCreate()` 和 `onRestoreInstanceState()` 回调方法均接收包含实例状态信息的相同 Bundle。

`onCreate()`中恢复数据：
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState); // Always call the superclass first

    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        // Restore value of members from saved state
        mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
        mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
    } else {
        // Probably initialize members with default values for a new instance
    }
    ...
}
```

　　您可以选择实现系统在 `onStart()` 方法之后调用的 `onRestoreInstanceState()` ，而不是在onCreate() 期间恢复状态。 **系统只在存在要恢复的已保存状态时调用`onRestoreInstanceState()` ，因此您无需检查 Bundle 是否为 null：**
```java
public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);

    // Restore state members from saved instance
    mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
    mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
}
```
**要特别强调的是**，`onSaveInstanceState` 并不是生命周期方法，Android 系统**并不保证在杀死 `activity`前一定调用它**：

当 `activity` 进入后台或者被销毁时 `onPause()` 总会被调到；当 `activity` 被销毁时 `onStop()` 总会被调到；而 `onSaveInstanceState()` 不会被调到的两种情况：
* 当 `activity B` 被调到 `activity A` 上面，并且在B的生命周期内A未被杀掉，系统不会为A调用这个方法。因为A仍保持它原来的用户交互的状态。
* 当用户按返回键从 `activity B` 返回到 `activity A`，系统不会为B调用这个方法。因为B的实例已被销毁，所以没有必要恢复状态。

**旋转时方法的调用流程:**
```
旋转发生了，activity 实例被销毁：
onPause() // 存储永久数据
onSaveInstanceState(Bundle) // 存储暂时数据，在onStop前被调用
onStop()
onDestroy()

重建 activity 实例：
onCreate(Bundle) // 获取暂存数据
onStart()
onRestoreInstanceState(Bundle) // 也可在这获取暂存数据，在onStart后被调用
onResume() // 做对应于onPause的恢复的工作
```
###2. onRetainNonConfigurationInstance

　　如果Activity重建需要耗费大量资源或需要访问网络导致时间很长，可以实现`onRetainNonConfigurationInstance()`方法将所需数据先保存到一个对象里，像下面这样：
```java
@Override 

public Object onRetainNonConfigurationInstance() { 
    final MyDataObject data = collectMyLoadedData(); 
    return data; 
}
```
在onCreate()函数中调用`getLastNonConfigurationInstance()`，获取`onRetainNonConfigurationInstance()`保存的数据:
```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    final MyDataObject data = (MyDataObject) getLastNonConfigurationInstance();
    if (data == null) {//表示不是由于Configuration改变触发的onCreate()
        data = loadMyData();
    }
    ...
}
```
**需要注意的是:**

　　我们不应该保存那些依赖Activity的数据，比如Drawable，Adapter，View或者任何与Context相关联的数据，这样会导致内存泄露。

##7. 非重启Activity的横竖屏切换

　　虽然重启Activity为我们提供了保存数据和读取数据的方式，但是我们也可以通过非重启Activity的方式来应对横竖屏的切换。Android也为我们提供了解决方案，就是通过onConfigurationChanged拦截横竖屏变换，从而进行必要的重新布局和切换操作。操作步骤如下：

1. 在AndroidManifest中为相应的Activity设置`android:configChanges`属性，指定需要忽略的Configuration属性，从而使Activity不进行重建流程，具体如下：
    * **Android3.2以前**
    
        ```
        android:configChanges="orientation|keyboardHidden"
        ```

##8. setRequestOrientation()

--
[参考]
* [Android横竖屏切换小节](http://www.cnblogs.com/franksunny/p/3714442.html)
* [重新创建Activity](https://developer.android.com/training/basics/activity-lifecycle/recreating.html#SaveState)
* [如何在Android设备旋转时暂存数据以保护当前的交互状态？](https://segmentfault.com/a/1190000003965285#articleHeader7)
* [Android 处理横竖屏切换事件](http://ipjmc.iteye.com/blog/1265991)

