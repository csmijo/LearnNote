#公共技术点之View绘制流程

`Android` `View` `笔记`

--

##1. View树的绘制流程

当`Activity`接收到焦点的时候，它会被请求绘制布局,该请求由 `Android framework` 处理.**绘制是从根节点开始，对布局树进行 `measure` 和 `draw`**。整个 `View` 树的绘图流程在`ViewRoot.java`类的`performTraversals()`函数展开，该函数所做的工作可**简单概况**为:
* 是否需要重新计算视图大小(measure)
* 是否需要重新安置视图的位置(layout)
* 以及是否需要重绘(draw)

**流程图如下:**
![view_mechanism_flow.png](pic/view_mechanism_flow.png "")

**View 绘制流程函数调用链**

![view_draw_method_chain.png](pic/view_draw_method_chain.png "")

**注意：**

　　用户主动调用`request`，**只会执行**`measure`和`layout`过程，而**不会执行**`draw`过程。

##2. 概念（measure和layout）

从整体来看Measure和Layout这两个步骤的执行如下图所示：
![measure_layout.png](pic/measure_layout.png "")

###具体分析
####1. measure过程

　　measure过程由`measure(int,int)`方法发起，从上到下有序的测量View，在measure过程的最后，每个视图存储自己的尺寸大小和测量规格。

　　measure过程会为一个View及所有子节点的`mMeasuredWidth`和`mMeasuredHeight`变量赋值，该值可以通过`getMeasuredWidth()`和`getMeasuredHeight()`方法获得。

**measure过程传递尺寸的两个类**
* `ViewGroup.LayoutParams` (View自身的布局参数)
* `MeasureSpecs` (父视图对子视图的测量要求)

1. **ViewGroup.LayoutParams**

    这个类用来指定**视图的高度和宽度等参数**。对于每个视图的height和width，可以有以下几个选择值：
    * **具体值**
    * **MATCH_PARENT** 表示子视图希望和父视图一样大(**不包含padding值**)
    * **WRAP_CONTENT** 表示子视图大小为正好能包裹其内容大小(**包含padding值**)

2. **MeasureSpecs**
    
    测量规格，包含**测量要求和尺寸的信息**，有三种模式：
    * **UNSPECIFIED**： 父视图不对子视图有任何约束，它可以达到所期望的任意尺寸。(**一般自定义View中用不到**)
    * **EXACTLY**： 父视图为子视图指定一个确切的尺寸，而且无论子视图期望多大，它都必须在该指定大小的边界内，**对应的属性为 match_parent 或具体值**。
    * **AT_MOST**：父视图为子视图指定一个最大尺寸。子视图必须确保它自己所有子视图可以适应在该尺寸范围内，**对应的属性为 wrap_content**，这种模式下，父控件无法确定子 View 的尺寸，只能由子控件自己根据需求去计算自己的尺寸，**这种模式就是我们自定义视图需要实现测量逻辑的情况**。
    
3. **measure核心方法**
    * `measure(int widthMeasureSpec,int heightMeasureSpec)`
        
        该方法不可覆写，它最终会调用view/viewGroup的`onMeasure()`方法
    * `onMeasure(int widthMeasureSpec,int heightMeasureSpec)`

        **该方法就是我们自定义视图中实现测量逻辑的方法**，该方法的参数是父视图对子视图的 `width` 和` height` 的测量要求。在我们自身的自定义视图中，要做的就是根据该 `widthMeasureSpec` 和 `heightMeasureSpec` 计算视图的` width` 和 `height`，不同的模式处理方式不同。
    * `setMeasureDimension()`
    
         **测量阶段终极方法**，在`onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 方法中调用，将计算得到的尺寸，传递给该方法，测量阶段即结束。**该方法也是必须要调用的方法，否则会报异常**。在我们在自定义视图的时候，不需要关心系统复杂的 Measure 过程的，**只需调用**`setMeasuredDimension()`设置根据 `MeasureSpec` 计算得到的尺寸即可.
         
![measurechildflow.png](pic/measurechildflow.png "")
         
####2. layout过程
    
　　layout过程由`layout(int,int,int,int)方法发起，也是自上而下进行遍历。在该过程中，每个父视图会根据`measure`过程得到的尺寸来摆放自己的子视图。

　　首先要明确的是，**子视图的具体位置都是相对于父视图而言的**。View 的 `onLayout`方法为空实现，而 `ViewGroup` 的 `onLayout` 为 `abstract` 的，因此，**如果自定义的 View 要继承 ViewGroup 时，必须实现 onLayout 函数**。

　　在 layout 过程中，子视图会调用`getMeasuredWidth()`和`getMeasuredHeight()`方法获取到 `measure` 过程得到的 `mMeasuredWidth` 和 `mMeasuredHeight`，作为自己的 `width` 和 `height`。然后**调用每一个子视图的`layout(l, t, r, b)`函数，来确定每个子视图在父视图中的位置**。

####3. draw过程

先看一些与draw过程相关的函数：
* `View.draw(Canvas canvas)`: 由于 ViewGroup 并没有复写此方法，因此，所有的视图最终都是调用 `View` 的 `draw` 方法进行绘制的。**在自定义的视图中，也不应该复写该方法，而是复写 onDraw(Canvas) 方法进行绘制**，如果自定义的视图确实要复写该方法，那么请先调用 super.draw(canvas)完成系统的绘制，然后再进行自定义的绘制。
* `View.onDraw()`: View 的onDraw（Canvas）默认是空实现，**自定义绘制过程需要复写的方法**，绘制自身的内容。
* `dispatchDraw()`: 发起对子视图的绘制。View 中默认是空实现，**ViewGroup 复写了`dispatchDraw()`来对其子视图进行绘制**。该方法我们**不用去管**，自定义的 `ViewGroup` **不应该**对dispatchDraw()进行复写。
* `drawChild(canvas, this, drawingTime)`
直接调用了 `View` 的 `child.draw(canvas, this,drawingTime)` 方法，文档中也说明了，除了被`ViewGroup.drawChild()` 方法外，你不应该在其它任何地方去复写或调用该方法，它属于 `ViewGroup`。而`View.draw(Canvas)`方法是我们自定义控件中可以复写的方法，具体可以参考上述对`view.draw(Canvas)`的说明。从参数中可以看到，`child.draw(canvas, this, drawingTime)` 肯定是处理了和父视图相关的逻辑，但 `View` 的最终绘制，还是 `View.draw(Canvas)`方法。
* `invalidate()`: 请求重绘View树，即draw过程，假如视图大小没有发生变化就不会调用`layout()`过程，并且只绘制哪些调用了`invalidate()`方法的view
* `requestLayout()`: 当布局变化的时候，比如方向，尺寸变化时会调用该方法，在自定义的视图中，**如果某些情况下希望重新测量尺寸大小，应该手动去调用该方法，它会触发`measure()`和`layout()`过程，但不会进行 `draw`**。

![draw_method_flow.png](pic/draw_method_flow.png "")
--
【参考】
* [公共技术点之 View 绘制流程](http://b.codekk.com/blogs/detail/54cfab086c4761e5001b253f)