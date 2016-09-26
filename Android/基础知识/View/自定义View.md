#自定义View

`Android` `view` `笔记`

--
    
##1. 问题引出

* **问题一：** 从Android系统设计者的角度，View这个概念究竟是做什么的？
* **问题二：** Android系统中这个view类，它有哪些默认功能和行为，能做什么，不能做什么？
* **问题三：** 如果要改变这个view的行为，外观，怎么覆写view中的方法？

##2. 问题一：从Android系统设计者的角度，View这个概念究竟是做什么的？

###1. 官方定义：
> This class represents the **basic building block for user interface components**. A view **occupies a rectangular area** on the screen and is **responsible for drawing and event handing**.

###2. 解读

1. **View是用户接口组件的基本构建块。**  在Android中，用户与一个应用交互，其实就是和一个个view在进行交互。这些view可以是简单的view，也可以是若干view合成的复合view——即ViewGroup。
2. **View在屏幕上占据一个矩形区域。** **这是解决在哪里交互的问题。** 占据矩形区域是为了简单。通过这个区域，可以实现和用户的交互(点击，滑动，拖动等)。
3. **View通过绘制自己与事件处理两种方式与用户进行交互。** **这是解决如何交互的问题。** 
    * **通过绘制自己实现和用户交互。** 比如，用户点击view时，改变背景颜色等。
    * **通过事件处理实现和用户交互。** 比如，用户点击view时，弹出Toast告诉用户完成点击。

##3. 问题2：Android系统中这个view类，它有哪些默认功能和行为，能做什么，不能做什么？

###1. view如何被显示到屏幕

* 一套完整的用户界面用一个Window来表示，**Window负责管理所有的View们。**
* Window首先加载一个viewGroup，用它来包含所有的其他view，这个ViewGroup叫做**DecorView**。
* **注意**，这个DecorView除了包含用户界面上的那些view，还包含了作为**Window特有的view——titleBar**。


###2. 确定每个view的位置
　　DecorView知道，不同的View为了完成自己的交互任务所需要的屏幕区域大小是不同的，所以：
* DecorView在确定给每个View**分配的屏幕区域大小**时，**是允许View参与进来**，与它一起商量的。————**measure过程view和包含他的viewGroup协商**
* 但是每个View**在屏幕区域中的位置**就不能让View自己来决定了，而是**由DecorView一手操办**。————**layout过程由包含view的viewGroup决定**

![自定义view_1.png](pic/自定义view_1.png "")

　　虽然View无法决定自己在ViewGroup中的位置，但是开发者使用View时可以向ViewGroup表达自己所用的View要放在哪里。以vertical LinearLayout为例，view在布局文件出现的位置决定view在LinearLayout中出现的位置，还可以通过Layout_margin，Layout_gravity等进行进一步调整。所以：
* **Layout_* 之类的配置并不是view的属性**，它们只是使用该View的使用者用来细化调整该View在ViewGroup中位置的。
* 这些Layout_* 值**在Inflate时，是由ViewGroup读取**，然后生成一个**ViewGroup特定的LayoutParams**。这样ViewGroup在为该子View安排位置时，就可以参考这个layoutParams中的信息了。

　　不同的ViewGroup拥有不同的LayoutParams内部类，这是因为它们允许的子View调整自己位置的方式是不一样的。


　　这些确定View位置的过程，被包装在View的Layout方法中。所以：
* 对于基本view而言，layout方法是没有用的，所以都是空的。————**自定义View的时候不用覆写layout方法**
* 对于符合View——即ViewGroup而言，需要用layout方法来为自己的子view安排位置。————**自定义ViewGroup需要覆写layout方法**

###3. 确定View的大小

　　要确定View的大小，这是一个**开发者、View、ViewGroup三方协商**的过程。

1. 开发者在书写布局文件时，会为view写上layout_width,layout_height这两个配置。这是开发者向ViewGroup表达的，我希望这个View的大小是多少。这两个配置可能的取值有三种：
    * **具体值**，比如50dp
    * **match_parent**，表示开发者向ViewGroup说，把你所有的屏幕区域都给这个View
    * **wrap_content**，表示开发者向ViewGroup说，只要给这个View够他展示自己的空间即可，**置于具体给多少，就需要ViewGroup和View沟通了**

2. ViewGroup收到了开发者对View大小的说明，然后**ViewGroup会综合考虑自己的空间大小以及开发者的请求，然后生成两个MeasureSpec对象（width与height）传给View，这两个对象是ViewGroup向子View提出的要求**，就相当于告诉子View：“我已经与你的使用者（开发者）商量过了，现在把我们商量确定的结果告诉你，你的宽度不能违反width MeasureSpec对象的要求，你的高度不能违反height MeasureSpec对象的要求，现在，你赶紧根据这个要求确定下自己要多大空间，**只许少，不许多哦**。”

    　　然后，**这两个对象将会传到子View的`protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)`方法中**。子View能怎么办呢？它肯定是要先看看ViewGroup的要求是什么吧，于是，它从传入的两个对象中解译出如下信息：
    
    ```java
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize =  MeasureSpec.getSize(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize =  MeasureSpec.getSize(heightMeasureSpec);
    ```
    > **Mode与Size一起，准确表达出了ViewGroup的要求**
    
    **Mode的取值有三种，他们代表着ViewGroup的总体态度：**
    * `EXACTLY`: 比如ViewGroup对View说，你**只能用100dp**，原因是多样的，你不能不理不顾，不然你达不到他的要求，他可能就不用你了。
    * `AT_MOST`: 比如**你最多只能用100dp**，这是因为你的使用者说让你占据wrap_content的大小，让我跟你商量，我又不知道你到底要占多大区域，但是我告诉你，我只有100dp，你最多也只能用这么多哈。
    * `UNSPCIFIED`: 你自己看着办，把你最理想的大小告诉我，我考虑考虑

3. 子View已经清楚地理解了ViewGroup和它的使用者对它的大小的期望和要求了。下步就要在该要求下来确定自己的大小并告诉ViewGroup了。

    关于子view怎么确定自己的大小，不同的view有不同的态度，但是**有几点基本的规矩是要遵守的**：
    + **规矩一：** **不要违反ViewGroup的规定**，最后设置的尺寸一定要在ViewGroup要求的范围内（不论是宽度还是高度）。
    + **规矩二：** **就是要在该方法中调整自己的绘制参数**，这一点很好理解，毕竟ViewGroup提出了尺寸要求，要及时根据这一要求调整自己的绘制，比如，如果自己的背景图片太大，那就算算要缩放多少才合适，并且设置一个合理的缩放值。
    + **规矩三：** **就是一定要设置自己考虑后的尺寸**，如果不设置就相当于没有告诉ViewGroup自己想要的大小，这会导致ViewGroup无法正常工作，设置的办法就是在`onMeasure()`方法的最后，调用`setMeasuredDimension()`方法。

    |childLayoutParams\parentSpecMode|EXACTLY|AT_MOST|UNSPECIFIED|
    |:---:|:---:|:---:|:---:|
    |dp/px|EXACTLY<br>childSize|EXACTLY<br>childSize|EXACTLY<br>childSize|
    |match_parent|EXACTLY<br>parentSize|AT_MOST<br>parentSize|UNSPECIFIED<br>0|
    |wrap_content|AT_MOST<br>parentSize|AT_MOST<br>parentSize|UNSPECIFIED<br>0|
    
###4. 绘制view

　　关于View的绘制，非常简单，就是一个方法`onDraw()`。

##4. 问题3：如果要改变这个view的行为，外观，怎么覆写view中的方法？

![view-method-for-override.png](pic/view-method-for-override.png "")

* 比如View被`inflated`出来后，系统会回调该View的**`onFinishInflate()`**方法，你的View可以在这个方法中，做一些准备工作。
* 如果你的View所属的**Window可见性发生了变化**，系统会回调该View的**`onWindowVisibilityChanged()`**方法，你也可以根据需要，在该方法中完成一定的工作，比如，当Window显示时，注册一个监听器，根据监听到的广播事件改变自己的绘制，当Window不可见时，解除注册，因为此时改变自己的绘制已经没有意义了，自己也要跟着Window变成不可见了。
* 当ViewGroup中的**子View数量增加或者减少**，导致ViewGroup给自己分配的屏幕区域大小发生变化时，系统会回调View的**`onSizeChanged()`**方法，该方法中，View可以获取自己最新的尺寸，然后根据这个尺寸相应调整自己的绘制。
* 当用户在View所占据的**屏幕区域发生了触摸交互**，系统会将用户的交互动作分解成如DOWN、MOVE、UP等一系列的MotionEvent，并且把这些事件传递给View的**`onTouchEvent()`**方法，View可以在这个方法中进行与用户的交互处理。
* 除了这些方法，View还实现了**三个接口**，如下：
![自定义view_2.png](pic/自定义view_2.png "")

    三个接口是：
    * Drawable.Callback: 用来让View中的Drawable能够与View通信，尤其是AnimationDrawable，更是必须依赖该回调才能实现对话效果。
    * KeyEvent.Callback: 用来处理键盘事件的，这与`onTouchEvent`用来处理触摸事件是相对的
    * AccessibilityEventSource


--
【参考】

* [教你步步为营掌握自定义View](http://www.jianshu.com/p/d507e3514b65)
* [android的逐帧动画](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1106/1915.html)