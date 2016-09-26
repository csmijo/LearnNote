#View事件传递

`Android` `View` `笔记`

--

##1. 基础知识

(1) 所有 `Touch`事件都被封装成 `MotionEvent` 对象，包括 Touch的位置、时间、历史记录以及第几个手指(多指触摸)等。

(2) 事件类型分为: `ACTION_DOWN`,`ACTION_UP`,`ACTION_MOVE`,`ACTION_POINTER_DOWN`,`ACTION_POINTER_UP`,`ACTION_CANCEL`，每个事件都是以`ACTION_DOWN`开始，`ACTION_UP`结束的。
(3) 对事件的处理包括三类：
* 传递——`dispatchTouchEvent()`
* 拦截——`onInterceptTouchEvent()`
* 消费——`onTouchEvent()`和`onTouchListener()`

##2. 传递流程
(1) 事件从`Activity.dispatchTouchEvent()`开始传递，只要没有被停止或者拦截，从最上层的View/ViewGroup开始一直往下(子View)传递。子View可以通过`onTouchEvent()`对事件进行处理。

(2) 事件从父View/ViewGroup 传递给子View， ViewGroup通过 `onInterceptTouchEvent()`对事件做拦截，停止其往下传递。**`onInterceptTouchEvent()`只存在于ViewGroup中**，普通的View中没有这个方法。

(3) 如果事件从上往下传递过程中一直没有停止，且最底层子View没有消费该事件，该事件就会反向往上传递，这时父View/ViewGroup可以进行消费，如果还是该事件还是没有被消费的话，最后会到Activity的`onTouchEvent()`函数

(4) 如果View没有对 `ACTION_DOWN`进行消费，**之后的其他事件不会传递过来**。

(5) `onTouchListener()`优先于 `onTouchEvent()`对事件进行消费。

##3. 拦截事件
![view-dispatchTouch.png](pic/view-dispatchTouch.png "")

**说明：**

　　ViewGroup A包含 多个View和ViewGroup，其中就包含有ViewGroup B。ViewGroup B也包含多个View 和ViewGroup，其中就包含有View C。View C是一个普通的View。

　　现在假设用户首先触摸到的屏幕上的点是C上的某个点，该点被标记为触摸点（`touch point`），DOWN事件就在该点产生。然后用户移动手指并最后离开屏幕，此过程中手指是否离开C的区域无关紧要，关键是手势（gesture）是从哪里开始的。

　　现在假设ViewGroup B没有拦截 `DOWN`事件，但是它拦截了 `MOVE`事件。原因可能是ViewGroup B是一个scrolling view。当用户仅仅在它的区域内点击（tap）时，被点击到的元素应当能处理该点击事件。但是当用户手指移动了一定的距离后，就不能再视该手势（gesture）为点击了——很明显，用户是想scroll。这就是为什么B要接管该手势（gesture）。

**下面是事件被处理的顺序：**

1. `DOWN` 事件被依次传到 A 和 B的 `onInterceptTouchEvent()`方法，他们都返回 false， 因为他们目前还不想拦截。
2. `DOWN`事件传递到C的onTouchEvent方法，返回了true。
3. 在后续到来`MOVE`事件时，A 的`onInterceptTouchEvent()`方法仍然返回false。
4. B 的`onInterceptTouchEvent()`方法收到了该 `MOVE` 事件，此时B 注意到用户手指移动距离已经超过了一定的threshold（或者称为slop）。因此，B的`onInterceptTouchEventz()` 方法决定返回true，从而**接管该手势（gesture）后续的处理**。
5. 然后，这个 `MOVE` 事件将会被系统变成一个 `CANCEL`事件，这个`CANCEL`事件将会传递给C 的`onTouchEvent()`方法。
6. 现在，又来了一个`MOVE 事件，它被传递给A的 `onInterceptTouchEvent()`方法，A还是不关心该事件，因此`onInterceptTouchEvent()`方法继续返回false。
7. 此时，该 `MOVE`事件将不会再传递给B 的`onInterceptTouchEvent()`方法，该方法**一旦返回一次`true`，就再也不会被调用了**。事实上，该MOVE以及“手势剩余部分”都将传递给**B的 `onTouchEvent()` 方法**（除非A决定拦截“手势剩余部分”）。
8. C **再也不会**收到该手势（gesture）产生的任何事件了。

**总结：**
* 如果一个`ViewGroup` 拦截了**最初的`DOWN`**事件，该事件**仍然会**传递到该ViewGroup的`onTouchEvent()`方法中。
* 另一方面，如果ViewGroup拦截了一个**半路的事件**（比如，MOVE），这个事件将会被系统变成一个 `CANCEL` 事件，并传递给之前处理该手势（gesture）的子View，而且**不会再传递**（无论是被拦截的MOVE还是系统生成的CANCEL）给 ViewGroup的`onTouchEvent()`方法。只有再到来的事件才会传递到ViewGroup的`onTouchEvent()`方法中。
* 如果你想更加疯狂一点，你可以在你自己的ViewGroup中直接覆写 `dispatchTouchEvent()`方法，并对传递进来的事件做任何你想做的处理。但这样的话你**可能会破坏一些约定，所以应当小心。**

--
【参考】
* [公共技术点之 View 事件传递](http://b.codekk.com/blogs/detail/54cfab086c4761e5001b253e)
* [可能是讲解Android事件分发最好的文章](http://www.jianshu.com/p/2be492c1df96)