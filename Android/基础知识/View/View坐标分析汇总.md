#View坐标分析汇总

`Android` `View`

--

##1. 概述

获取坐标的一些方法，大体如下图所示：

![view坐标系](http://jbcdn2.b0.upaiyun.com/2016/08/37af6dff6bb770ac33823a9127b2700c.png "")

![屏幕view坐标系](http://jbcdn2.b0.upaiyun.com/2016/08/f86ba7108f5389b0f0e6f43a6696a247.png "")

###1. view提供的方法
* **getTop**: view自身的顶边到其父布局顶边的距离
* **getBottom**: view自身的底边到其父布局顶边的距离
* **getLeft**: view自身的左边到其父布局左边的距离
* **getRight**: view自身的右边到其父布局左边的距离

###2. MotionEvent提供的方法
* **getX()**: 获取点击事件相对控件左边的x轴坐标，即点击事件距离控件左边的距离
* **getY()**: 获取点击事件相对控件顶边的y轴坐标，即点击事件距离控件顶边的距离
* **getRawX()**: 获取点击事件相对整个屏幕左边的x轴坐标，即点击事件距离整个屏幕左边的距离
* **getRawY()**: 获取点击事件相对整个屏幕顶边的y轴坐标，即点击事件距离这个屏幕顶边的距离

--
**[参考]**
* [android之View坐标系(view获取自身坐标的方法和点击事件中坐标的获取)](http://blog.csdn.net/jason0539/article/details/42743531)

##2. 获取

![Android4.0窗口机制](http://jbcdn2.b0.upaiyun.com/2016/08/24eac3a9409b8a95190c2fe170393658.png "")


　　对于View，ViewGroup来说，width、height、top、left等属性值是在Measure与Layout过程完成之后才开始正确赋值的，而**Measure与Layout却都晚于onCreate方法执行，所以onCreate中getLeft根本就取不到值**！

　　那如何在onCreate方法中取到width，height的值呢？参考一下博客：
* [解决在onCreate()过程中获取View的width和Height为0的4种方法](http://www.cnblogs.com/kissazi2/p/4133927.html)
* [ onCreate中View的width,height为0的问题](http://blog.csdn.net/codezjx/article/details/45341309)、

总结如下：

1. 监听Draw/Layout事件：ViewTreeObserver
2. **将一个runnable添加到Layout队列中：View.post()**
3. 重写View的onLayout方法
4. **重写Activity的onWindowFocusChanged方法，在该方法中获取**

--

**[参考]**
* [Android4.0窗口机制和创建过程分析](http://bbs.51cto.com/thread-1072344-1.html)
* [解决在onCreate()过程中获取View的width和Height为0的4种方法](http://www.cnblogs.com/kissazi2/p/4133927.html)
* [ onCreate中View的width,height为0的问题](http://blog.csdn.net/codezjx/article/details/45341309)

##3. 计算

###1. Android的坐标轴：

![Android坐标系](http://jbcdn2.b0.upaiyun.com/2016/08/72351a14d893c8da0311a817fd5e85bd.png "")

###2. 实现View移动的方法大致如下图所示：
![View移动的方法简介](http://jbcdn2.b0.upaiyun.com/2016/08/bac0e3eab708d9d498b73eabedf4efb8.png "")


###3. 对于scrollTo、scrollBy需要注意的两个问题
**问题一：**


**移动计算值 = 最开始点坐标 – 最后移动到的坐标**

原因是因为最终会调用这个方法
—— invalidateInternal(l – scrollX, t – scrollY, r – scrollX, b – scrollY, true, false);

**其中l,t,r,b为原来坐标点，scrollX,scrollY为目标坐标点，只有当目标坐标点值是负数时，移动到的位置才为正数！**

例如scrollTo ，我们要从（0,0）移动到（200,400）这个点，根据上面的公式可知为负值


**问题二：**

**为什么需要加上 ((View)getParent())？**

TestTextView本身是View，**scrollTo、scrollBy移动的都是View的Content**，如果不加的话，使用的效果则是TestTextView的文字位置变化，而TestTextView本身不会变化。

**如果在ViewGroup中使用scrollTo、scrollBy，则移动的是ViewGroup中的View**.我们这里需要让TestTextView移动，则需要先 ((View)getParent())，然后再((View)getParent()).scrollTo…


--

**[参考]**
* [那些你应该知道却不一定知道的—View坐标分析汇总](http://android.jobbole.com/84285/)