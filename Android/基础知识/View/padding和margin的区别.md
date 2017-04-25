#padding和layout_margin的区别

`Android` 

--

##1. 描述

Android的Padding和layout_margin和html中的是一样的。如下图所示：

![padding和layout_margin](http://images.cnblogs.com/cnblogs_com/ghj1976/201104/201104261840454974.png "")
![padding&margin.jpg](pic/padding&margin.jpg "")

1. 描述一：
    * Padding 为内边框，指控件内容相对控件的边框的距离
    * layout_margin  为外边框，指控件相对父控件边框的距离
    
2. 描述二：
    * padding是以**所被定义的控件A** 为parent控件，而内部的内容物与控件A的间距。**padding是站在父view的角度描述问题**，它表示的是它里面的内容与这个父view边界的距离。
    * layout_margin是以A控件**所在的控件**为parent控件，是A与其的间距。**margin是站在自己的角度描述问题的**，它表示自己与其他（上下左右）view之间的距离，如果同一级只有一个view，那么它的效果基本上和padding一样。

3. 描述三：

 
To help me remember the meaning of padding, I think of a big coat with lots of thick cotton padding. I'm inside my coat, **but me and my padded coat are together. We're a unit.**

But to remember margin, I think of, "Hey, give me some **margin**!" **It's the empty space between me and you**. Don't come inside my comfort zone -- my margin.
 

![padding&margin_2.PNG](pic/padding&margin_2.PNG "")

##2. 常用方法信息总结

1. **view 中的 width，height 源码定义**

    |方法|返回值|说明|
    |:--|:--:|:--|
    |view.getWidth()|mRight-mLeft|The width of your view, in pixels.|
    |view.getHeight()|mBottom-mTop|The height of your view, in pixels.|
    
2. **view 中的 left, top, right, bottom 源码定义**

    |方法|返回值|说明|
    |:--|:--:|:--|
    |view.getLeft()|mLeft|1. Left position of this view relative to its parent. <br/>2. The left edge of this view, in pixels.|
    |view.getRight()|mRight|1. Right position of this view relative to its parent.<br/>2. The right edge of this view, in pixels.|
    |view.getTop()|mTop|1. Top position of this view relative to its parent. <br/>2. The top of this view, in pixels.|
    |view.getBottom()|mBottom|1. Bottom position of this view relative to its parent.<br/>2. The bottom of this view, in pixels.|

   
3. **ViewGroup中 LayoutParams 的 leftMargin，topMargin，rightMargin，bottomMargin 源码定义**

    LayoutParams 获取方式： **view.getLayoutParams()**

    |方法|返回值|说明|
    |:--|:--:|:--|
    |params.leftMargin|leftMargin|1. The left margin in pixels of the child.<br/>2. Margin values should be positive.<br/>3. Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value to this field.|
    |params.topMargin|topMargin|1. The top margin in pixels of the child.<br/>2. Margin values should be positive.<br/>3. Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value to this field.|
    |params.rightMargin|rightMargin|1. The right margin in pixels of the child.<br/>2. Margin values should be positive.<br/>3. Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value to this field.|
    |params.bottomMargin|bottomMargin|1. The bottom margin in pixels of the child.<br/>2. Margin values should be positive.<br/>3. Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value to this field.|
    
4. **view 中的 leftPadding, topPadding, rightPadding, rightPadding 源码定义**

    |方法|返回值|说明|
    |:--|:--:|:--|
    |view.getPaddingLeft()|mPaddingLeft|1. Returns the left padding of this view. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the left padding in pixels|
    |view.getPaddingRight()|mPaddingRight|1. Returns the right padding of this view. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the right padding in pixels|
    |view.getPaddingTop()|mPaddingTop|1. Returns the top padding of this view. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the top padding in pixels|
    |view.getPaddingBottom()|mPaddingBottom|1. Returns the bottom padding of this view. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the bottom padding in pixels|
    |view.getPaddingStart()|(getLayoutDirection() == LAYOUT_DIRECTION_RTL) ? mPaddingRight : mPaddingLeft;|1. Returns the start padding of this view **depending on its resolved layout direction**. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the start padding in pixels|
    |view.getPaddingEnd()|(getLayoutDirection() == LAYOUT_DIRECTION_RTL) ? mPaddingLeft : mPaddingRight;|1. Returns the end  padding of this view  **depending on its resolved layout direction**. If there are inset and enabled scrollbars, this value may include the space required to display the scrollbars as well.<br/>2. the end padding in pixels|
    
    其中：LAYOUT_DIRECTION_RTL 表示 Horizontal layout direction of this view is from Right to Left.
    
5. **【重要】**
    * **To measure its(view) dimensions, a view takes into account its padding.** The padding is expressed in pixels for the left, top, right and bottom parts of the view. **Padding can be used to offset the content of the view by a specific amount of pixels.** For instance, a left padding of 2 will push the view's content by 2 pixels to the right of the left edge. Padding can be set using the **setPadding(int, int, int, int)** or** setPaddingRelative(int, int, int, int)** method and queried by calling getPaddingLeft(), getPaddingTop(), getPaddingRight(), getPaddingBottom(), getPaddingStart(), getPaddingEnd().
        * 这段话说明：view 在计算它的大小，也就是 getWidth(),getHeight() 时会**将 padding 的值也计算在内**。
    * **Even though a view can define a padding, it does not provide any support for margins.** **However**, view groups provide such a support. Refer to **ViewGroup** and **ViewGroup.MarginLayoutParams** for further information.
        * 这段话说明：view 在计算它的大小时，**不将 margin 的值计算在内**。 
    
    
    
    



--
**参考文献**
* [ android:padding和android:margin的区别](http://blog.csdn.net/xxdbupt/article/details/20450915)
* [Android中padding与layout_margin的区别与用法](http://www.it165.net/pro/html/201503/35285.html)
* [Difference between a View's Padding and Margin](http://stackoverflow.com/questions/4619899/difference-between-a-views-padding-and-margin)
* [View](https://developer.android.com/reference/android/view/View.html)