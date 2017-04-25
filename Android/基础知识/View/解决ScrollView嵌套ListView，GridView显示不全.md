#解决ScrollView嵌套ListView，GridView显示不全

`Android`

--

很多开发人员都遇到过这种需求，就是ScrollView内部嵌套ListView，而该ListView数据条数是不确定的
，所以需要设置为包裹内容，然后就会发现ListView就会显示第一行出来。然后就会百度到一条解决方案，继承ListView，覆写onMeasure方法。

##1. 解决方法


可以通过重写ListView 或者 GridView 的 onMeasure() 方法来解决显示不全问题。

```java
@Override  
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
    int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);  
    super.onMeasure(widthMeasureSpec, expandSpec);  
}  
```

* [解决ScrollView下嵌套ListView、GridView显示不全的问题(冲突)](http://blog.csdn.net/cs_li1126/article/details/12906203)
* [详解嵌套ListView、ScrollView布局显示不全的问题](http://blog.csdn.net/hanhailong726188/article/details/46136569)

##2. 原理

ListView 的 onMeasure() 方法：

```java
if (heightMode == MeasureSpec.UNSPECIFIED) {
      heightSize = mListPadding.top + mListPadding.bottom + childHeight +
                   getVerticalFadingEdgeLength() * 2;
}

if (heightMode == MeasureSpec.AT_MOST) {
     // TODO: after first layout we should maybe start at the first visible position, not 0
     heightSize = measureHeightOfChildren(widthMeasureSpec, 0, NO_POSITION, heightSize, -1);
}
```

* 当 `heightMode` 是 `MeasureSpec.UNSPECIFIED` 时，高度就等于第一行的高度值；
* 当 `heightMode` 是 `MeasureSpec.AT_MOST` 时，高度就等于所有子view的高度
* GridView 的 onMeasure() 方法的效果一样

所以：我们猜测可能是因为`ScrollView`传递了一个`UNSPECIFIED`限制给`ListView`。

通过`ScrollView` 的`onMeasure()` 源码可以发现，`ScrollView` 在 `measureChildWithMargins()` 中向
`ListView` 的 `onMeasure()` 方法传递了一个`UNSPECIFIED` 的限制。

为什么呢，想想，**因为ScrollView，本来就是可以在竖直方向滚动的布局**，所以，它对它所有的子View的高度就是UNSPECIFIED，
意思就是，**不限制子View有多高**，因为我本来就是需要竖直滑动的，它的本意就是如此，所以它对子View高度不做任何限制。


* [Android View框架的measure机制](http://blog.csdn.net/hjf_huangjinfu/article/details/51147636)

