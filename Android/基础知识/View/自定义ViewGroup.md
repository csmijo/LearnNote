#自定义ViewGroup

`Android` `ViewGroup` `笔记`

--

##1. ViewGroup，View职责概述

从Api的角度对ViewGroup和View进行浅析：

1. View的职责是根据ViewGroup传入的测量值和测量模式，对自己的宽高进行确定(`onMeasure()`中完成)，然后在`onDraw()`中完成对自己的绘制
2. ViewGroup则需要给View传入View的测量值和测量模式(`onMeasure()`中完成)，而且对于此ViewGroup的父布局，自己也需要在`onMeasure()`完成对自己宽高的确定。此外，还需要在`onLayout()`中完成对其childView的位置指定。

![measurechildflow.png](pic/measurechildflow.png "")

##2. 自定义ViewGroup本质就干一件事——layout

ViewGroup**最最根本的职责**就是在自己的内部，给每一个view/viewGroup找一个合适的位置，也就是调用他们的`layout()`方法：

```java
public void layout(int left, int top, int right, int bottom)
```

**注意：**

1. **关于`left`,`right`,`top`,`bottom`**。 他们都是坐标值，而他们参照的**坐标系是由ViewGroup设定的**。这个坐标系是以ViewGroup左上角为原点，向右 X， 向下 Y构建起来的。
2. ViewGroup在自己的layout方法中，获得了parent给自己设定的尺寸大小，即**availableWidth**和**availableHeight**，这个值相当于parent告诉ViewGroup：“请以你的左上角为圆点，向右为x，向下为y的坐标系，给你的每一个子View确定位置和大小。我可以向你保证，这个坐标系中的点P1（0，0）、点P2（availableWidth，0）、点P3（0，availableHeight）、点P4（availableWidth,availableHeight）组成的方框区域内的子View都可以获得在手机屏幕（这里指硬件意义上的屏幕）上展示自己的机会。这个方框之外的子View，能不能在手机屏幕上展示自己，我就管不了了。”从这里我们看到，**parent给我们的ViewGroup设定的尺寸，并不一定就完全对应着手机屏幕上的一块相同大小的区域**，在有些情况下，parent给我们的ViewGroup设定的这个尺寸可能比整个手机屏幕还大。但是，parent仍然向我们保证，在该区域内layout的子View，都能获得在手机屏幕上展示自己的机会，parent是如何做到这一点的呢？答案是：**通过parent的scroll功能。**
3. 假如我是一个ViewGroup，我把一个子View的一部分layout在了parent给定的区域内，**另一部分超出了该区域**，这个子View是不是最多只能获得部分展示自己的机会？”不用怀疑，答案是：**Yes！**
4. 如何解决layout在parent限定区域外的view的显示？有三个选择：
    * **选择一：** 不要将子View 放到这个区域之外
    * **选择二：** 让你的ViewGroup实现scroll功能，从而确保parent限定区域外的子View也能够有机会展示自己。
    * **选择三：** 将你的ViewGroup的parent换成ScrollView。这样你的ViewGroup就不用自己实现scroll功能了。但是ScrollView只能允许子View的高度超过自己，不允许子View的宽度超过自己。所以，作为ViewGroup，可以在不超过availableWidth的情况下，将子View layout 到任意的高度上。
    * **原则就是：** **宁超高度，不超宽度！**


##3. 疑问*

### 1. getWith(),getMeasuredWidth(),getIntricWidth()??

源码中定义：

|方法|返回值|说明|
|:---:|:---:|:---|
|`getWidth()`|**mRight-mLeft**|return the width of you view, in pixels.|
|`getMeasuredWidth()`|**mMeasuredWidth & MEASURED_SIZE_MASK**|like `getMeasuredWidthAndState()`,but only returns the **raw width** component(that is the result is masked by `MASURED_SIZE_MASK`)|
|`getMeasuredWidthAndState()`||1. Return the full width measurement information for this view as computed by the most recent call to `#measure(int, int)`. <br>2. This result is a bit mask as defined by  `#MEASURED_SIZE_MASK` and `#MEASURED_STATE_TOO_SMALL`<br> 3. This should be used during measurement and layout calculations only.<br> 4. Use `#getWidth()` to see how wide a view is after layout.|
   
**总结：**

1. `getMeasuredWidth()`： 获得的值是`setMeasuredDimension()`方法设置的值，它的值在`measure()`方法运行后就会确定；
2. `getWidth()`： 获得的值是`layout`方法中传递的四个参数中的`mRight-mLeft`，它的值在`layout()`方法运行后确定；
3. 一般情况下在`onLayout`方法中使用`getMeasure  dWidth()`方法，而在除`onLayout`方法之外的地方用`getWidth`方法

**getMeasuredWidth():**
![getMeasuredWidth.png](pic/getMeasuredWidth.png "") 

**getWidth():**
![getWidth.png](pic/getWidth.png "") 


【參考】
* [Android开发之getMeasuredWidth和getWidth区别从源码分析](http://blog.csdn.net/dmk877/article/details/49734869)

###2. getPaddingLeft??  padding?? margin???
 
　　查看笔记 **#padding和layout_margin的区别**。
###3. getLeft()???

源码中定义：

|方法|返回值|说明|
|:---:|:---:|:---|
|`getLeft()`|**mLeft**|1.Left position of this view relative to its parent <br> 2. return the left edge of this view, in pixels.|

参看上面的**`getWidth()`**

--
【參考】
* [教你步步为营掌握自定义ViewGroup](http://www.jianshu.com/p/5e61b6af4e4c)
* [Android 手把手教您自定义ViewGroup（一）](http://blog.csdn.net/lmj623565791/article/details/38339817)