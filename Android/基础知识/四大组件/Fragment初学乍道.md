#Fragment初学乍道

`Android` `Fragment`

--

##1. 基本概念

###1) Fragment的概述

　　Fragment是Android3.0后引入的一个新的API，他出现的初衷是为了适应大屏幕的平板电脑，当然现在他仍然是平板APP UI设计的宠儿，而且我们普通手机开发也会加入这个`Fragment`， 我们可以把他**看成一个小型的Activity，又称`Activity`片段**！

　　想想，如果一个很大的界面，我 就一个布局，写起界面来会有多麻烦，而且如果组件多的话是管理起来也很麻烦！而使用`Fragment` 我们可以把**屏幕划分成几块，然后进行分组，进行一个模块化的管理**！从而可以更加方便的在运行过程中动态地更新`Activity`的用户界面！

　　另外**`Fragment`并不能单独使用，他需要嵌套在`Activity`中使用**，尽管他拥有自己的生命周期，但是还是会受到宿主Activity的生命周期的影响，比如Activity 被destory销毁了，他也会跟着销毁！

###2) Fragment的生命周期
![Fragment的生命周期](http://www.runoob.com/wp-content/uploads/2015/08/31722863.jpg "")

###3) 核心要点

1. 3.0版本后引入,即minSdk要大于11
2. `Fragment`需要嵌套在`Activity`中使用,当然也可以嵌套到另外一个`Fragment`中,但这个被嵌套的`Fragment`也是需要嵌套在`Activity`中的,间接地说，**Fragment还是需要嵌套在Activity中**!! 受寄主Activity的生命周期影响,当然他也有自己的生命周期!另外**不建议在Fragment里面 嵌套Fragment因为嵌套在里面的Fragment生命周期不可控**!!!
3. 官方文档说创建Fragment时至少需要实现三个方法:`onCreate()`,`onCreateView()`,`OnPause()`; 不过**貌似只写一个`onCreateView()`也是可以的**...
4. Fragment的生命周期和Activity有点类似，有**三种状态**:
    * Resumed:在允许中的Fragment可见；
    * Paused:所在Activity可见,但是得不到焦点；
    * Stoped: 
        * 调用addToBackStack(),Fragment被添加到Bcak栈 
        * 该Activity转向后台,或者该Fragment被替换/删除
5. 停止状态的fragment仍然活着(所有状态和成员信息被系统保持着),然而,它对用户不再可见,并且如果`activity`被干掉,他也会被干掉.

###4) `android.app.Fragment` Vs `andrioid.support.v4.app.Fragment`

在导入Fragment类是具体是使用哪个包下的Fragment类呢？？

　　其实，前面说过Fragment是`Android 3.0(API 11)`后引入的，那么如果开发的app需要在3.0以下的版本运行，那么就使用`andrioid.support.v4.app.Fragment`，因为v4包最低可以兼容到1.6版本。如果不关心android 3.0以下的版本，则使用`android.app.Fragment`即可。

**使用V4包下的Fragment时注意事项：**
* 如果你使用了v4包下的Fragment,那么所在的那个`Activity`就要继承`FragmentActivity`哦! 
* `Fragment`,`FragmentManager`,`FragmentTransaction`都是用的v4包啊
* `getFragmentManager()`改成`getSupportFragmentManager()`

##2. 创建一个Fragment

###1) 静态加载Fragment

![静态加载Fragment](http://www.runoob.com/wp-content/uploads/2015/08/14443487.jpg "")

###2) 动态加载Fragment

![动态加载Fragment](http://www.runoob.com/wp-content/uploads/2015/08/29546434.jpg "")

##3. Fragment管理与Fragment事务

![Fragment管理与Fragment事务](http://www.runoob.com/wp-content/uploads/2015/08/97188171.jpg "")

##4. Fragment与Activity的交互

![Fragment与Activity的交互](http://www.runoob.com/wp-content/uploads/2015/08/45971686.jpg "")

###1) Fragment传递数据给Activity
* **Step1：定义一个回调接口(Fragment中)**

    ```java
    /*接口*/  
    public interface CallBack{  
        /*定义一个获取信息的方法*/  
        public void getResult(String result);  
    }  
    ```
* **Step2:接口回调(Fragment中)**

    ```java
    /*接口回调*/  
    public void getData(CallBack callBack){  
        /*获取文本框的信息,当然你也可以传其他类型的参数,看需求咯*/  
        String msg = editText.getText().toString();  
        callBack.getResult(msg);  
    }  
    ```
* **Step3:使用接口回调方法获取数据(Activity中)**

    ```java
    /* 使用接口回调的方法获取数据 */  
    leftFragment.getData(new CallBack() {  
     @Override  
           public void getResult(String result) {              /*打印信息*/  
                Toast.makeText(MainActivity.this, "-->>" + result, 1).show();  
                }
            }); 
    ```
    
###2) Fragment与Fragment之间的数据互传

　　其实这很简单,找到要接受数据的`fragment对象`,直接调用`setArguments`传数据进去就可以了。
* 通常的话是replace时,即fragment跳转的时候传数据的,那么只需要在初始化要跳转的`Fragment`后调用他的`setArguments()`方法传入数据即可!
* 如果是两个`Fragment`需要即时传数据,而非跳转的话,就需要先在`Activity`获得f1传过来的数据, 再传到f2了,就是以**Activity为媒介**~

**示例代码如下：**

```java
FragmentManager fManager = getSupportFragmentManager( );
FragmentTransaction fTransaction = fManager.beginTransaction();
Fragment t1 = new Fragment();
Fragment t2 = new Fragment();
Bundle bundle = new Bundle();
bundle.putString("key",id);
t2.setArguments(bundle); 
fTransaction.add(R.id.fragmentRoot, t2, "~~~");  
fTransaction.addToBackStack(t1);  
fTransaction.commit(); 
```

##5. 走一次生命周期

1. `Activity`加载`Fragment`的时候,依次调用下面的方法:**onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart ->onResume**
2. 当我们弄出一个悬浮的对话框风格的`Activity`,或者其他,就是**让Fragment所在的Activity可见,但不获得焦点**，此时的状态为：**onPause**
3. 当对话框关闭,`Activity`又获得了焦点,状态又变为**onResume**
4. 当我们替换`Fragment`,并调用`addToBackStack()`将他添加到Back栈中 **onPause -> onStop -> onDestoryView** ！！**注意**，此时的`Fragment`还没有被销毁哦!!!
5. 当我们按下键盘的回退键，`Fragment`会再次显示出来: **onCreateView -> onActivityCreated -> onStart -> onResume**
6. 如果我们替换后,在事务`commit`之前没有调用`addToBackStack()`方法将 `Fragment`添加到back栈中的话;又或者退出了`Activity`的话,那么`Fragment`将会被完全结束, `Fragment`会进入销毁状态**onPause -> onStop -> onDestoryView -> onDestory -> onDetach**

--
**参考文献：**

* [Fragment基本概述](http://www.runoob.com/w3cnote/android-tutorial-fragment-base.html "")