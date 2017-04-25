#Android SurfaceView 

`Android` `SufaceView`

--

## 1. 简述

###1. 普通 view 的缺陷：

1. View 缺乏双缓冲机制
2. 当程序需要更新view 上的图片时，程序必须重绘 View 上显示的整张图片
3. 新线程无法直接更新 view 组件
4. 刷新时需要我们主动调用 invalidate() 或者 postInvalidate()方法，view 比较被动

###2. SurfaceView 的主要优势：

1. SurfaceView 的刷新处于主动，有利于频繁的更新画面
2. SurfaceView 的绘制在子线程进行，避免了UI 线程的阻塞
3. SurfaceView 在底层实现了一个双缓冲机制，效率大大提升

###3. SurfaceView 的使用

1. 自定义 SurfaceView 类必须继承 SurfaceView 实现 SurfaceHolder.Callback 接口
2. 实现 SurfaceHolder.Callback 中的三个 SurfaceView 生命周期，分别为：、
    * surfaceCreated(SurfaceHolder holder)： 当Surface第一次创建后会立即调用该函数。程序可以在该函数中做些和绘制界面相关的初始化工作，一般情况下都是在另外的线程来绘制界面，所以不要在这个函数中绘制Surface。
    * surfaceChanged(SurfaceHolder holder, int format, int width, int height)：当Surface的状态（大小和格式）发生变化的时候会调用该函数，在surfaceCreated调用后该函数至少会被调用一次。 
    * surfaceDestroyed(SurfaceHolder holder)

3. SurfaceView 通常与 SurfaceHolder 结合使用，SurfaceHolder 用于向与之关联的 SurfaceView 上绘图，调用 SurfaceView 的 getHolder() 方法即可获取 SurfaceView 关联的 SurfaceHolder。

顾明思议，不多解释被回调的时机。
1. 在surfaceCreated方法里开启一个子线程。
2. 在这个子线程中开启一个由Flag控制的While循环，用于不断地绘制。
3. 在循环中通过SurfaceHolder对象的**lockCanvas方法**获得一个Canvas对象用于绘制。
4. 每次绘制完成通过SurfaceHolder对象**unlockCanvasAndPost方法**传入Canvas对象完成更新。
5. 最后要在surfaceDestroyed方法中去改变while循环的Flag为false，结束子线程的绘制。

【注意】： 当调用 SurfaceHolder 的 unlockCanvasAndPost() 方法之后，该方法之前所绘制的图形还处于缓冲区中，下一次 lockCanvas() 方法锁定的区域可能会"遮挡"它。

##2. 源码分析

###1. SurfaceView 的 MVC 架构

1. Surface，SurfaceHolder 和 SurfaceView 三者之间的关系实质上就是广为人知的 MVC。其中：
    * Surface——Model：数据模型，数据
    * SurfaceView——View：视图，交互界面
    * SurfaceHolder——Controller：控制器

###2. Surface

1. **android.view.SurfaceView**，Surface 实现了 Parcelable 接口进行序列化（这里主要用来在进程间传递 surface 对象），用来处理屏幕显示缓冲区的数据，源码中对它的注释为：** Handle onto a raw buffer that is being managed by the screen compositor**。 Surface是原始图像缓冲区（raw buffer）的一个句柄，而原始图像缓冲区是由屏幕图像合成器（screen compositor）管理的。
2. 虽然Surface保存了当前窗口的像素数据，但是在使用过程中是**不直接和Surface打交道**的，由SurfaceHolder的Canvas lockCanvas()或则Canvas lockCanvas(Rect dirty)函数来获取Canvas对象，通过在Canvas上绘制内容来修改Surface中的数据。

###3. SurfaceHolder

1. **android.view.SurfaceHolder**， SurfaceHolder 实际上是一个接口，它充当的是 Controller 的角色。
2. Callback 是 SurfaceHolder 内部的一个接口，接口中有以下三个方法：
    * **public void surfaceCreated(SurfaceHolder holder)**; 
        * Surface 第一次被创建时被调用，例如 SurfaceView 从不可见状态到可见状态时
        * 在这个方法被调用到 surfaceDestroyed 方法被调用之前的这段时间，Surface 对象是可以被操作的，拿 SurfaceView 来说就是如果 SurfaceView 只要是在界面上可见的情况下，就可以对它进行绘图和绘制动画 
        * **这里还有一点需要注意**，Surface 在一个线程中处理需要渲染的图像数据，如果你已经在另一个线程里面处理了数据渲染，就不需要在这里开启线程对 Surface 进行绘制了
    * **public void surfaceChanged(SurfaceHolder holder, int format, int width, int height)**; 
        * Surface 大小和格式改变时会被调用，例如横竖屏切换时如果需要对 Sufface 的图像和动画进行处理，就需要在这里实现 
        * 这个方法在 surfaceCreated 之后至少会被调用一次
    * **public void surfaceDestroyed(SurfaceHolder holder)**; 
        * Surface 被销毁时被调用，例如 SurfaceView 从可见到不可见状态时
        * 在这个方法被调用过之后，就不能够再对 Surface 对象进行任何操作，所以需要保证绘图的线程在这个方法调用之后不再对 Surface 进行操作，否则会报错

###4. SurfaceView

1. **android.view.SurfaceView**，SurfaceView，就是用来显示 Surface 数据的 View，通过 SurfaceView 来看到 Surface 的数据。
2. Provides a dedicated drawing surface embedded inside of a view hierarchy. 

    在屏幕显示的视图层中嵌入了一块用做图像绘制的 Surface 视图
    
3. the SurfaceView punches a hole in its window to allow its surface to be displayed. 

    SurfaceView 在屏幕上挖了个洞来来世它所绘制的图像

4. 挖洞是什么鬼？
    * **这里引入一个Z轴的概念，SurfaceView 视图所在层级的Z轴位置是小于用来其宿主 Activity 窗口的 Layer 的 Z 轴的，就是说其实 SurfaceView 实际是显示在 Activity 所在的视图层下方的**
    * 那么问题就来了，为什么还是能看到 SurfaceView？形象一点的说法就是你在墙上凿了一个方形的洞，然后在洞上装了块玻璃，你就能看到墙后面的东西了。SurfaceView 就做了这样的事情，它把 Activity 所在的层当作了墙
    
5. The Surface will be created for you while the SurfaceView's window is visible. 

    这里说明了动画是什么时候开始的，当 SurfaceView 可见时，就可以开始在 Canvas 上绘制图像，并把图像数据传递给 Surface 用来显示在 SurfaceView 上

6. you should implement SurfaceHolder.Callback#surfaceCreated and SurfaceHolder.Callback#surfaceDestroyed to discover when the Surface is created and destroyed as the window is shown and hidden. 

    在使用 SurfaceView 的地方需要实现 SurfaceHolder.CallBack 回调，来对 Surface 的创建和销毁进行监听以及做响应的处理，这里的处理指的是开始对 Canvas 进行绘制并把数据传递给 Surface 来做显示



--

[参考文献]

* [Android SurfaceView的基本使用](http://www.jianshu.com/p/07fa4180f0a5)
* [Android SurfaceView 源码分析及使用](http://tech.youzan.com/surfaceview-sourcecode/)