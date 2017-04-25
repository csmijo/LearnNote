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


##4. 图解 Android 事件分发机制

###1. Android 事件分发流

 ![Android 事件分发流](pic/Android 事件分发流.png "")
 
注意：

1. 仔细看的话，图分为3层，从上往下依次是Activity、ViewGroup、View
2. 事件从左上角那个白色箭头开始，由Activity的dispatchTouchEvent做分发
3. 箭头的上面字代表方法返回值，（return true、return false、return super.xxxxx(),super 的意思是调用父类实现。
4. dispatchTouchEvent和 onTouchEvent的框里有个【true—->消费】的字，表示的意思是如果方法返回true，那么代表事件就此消费，不会继续往别的地方传了，事件终止。
5. Activity 的dispatchTouchEvent 无论返回什么，都会调用ViewGroup 的dispatchTouchEvent（自行看源码）


###2. 详细分析

1. **如果事件不被中断，整个事件流向是一个类U型图**

    所以如果我们没有对控件里面的方法进行重写或更改返回值，而直接用super调用父类的默认实现，那么整个事件流向应该是从Activity—->ViewGroup—>View 从上往下调用dispatchTouchEvent方法，一直到叶子节点（View）的时候，再由View—>ViewGroup—>Activity从下往上调用onTouchEvent方法。
    
2. **dispatchTouchEvent 和 onTouchEvent 一旦return true,事件就停止传递了（到达终点）（没有谁能再收到这个事件）。**

    如Android事件分发流中的图片所示：只要 return true，就表示事件就没再往下继续传递了。
    
    【疑问】：dispatchTouchEvent return true 此时调用该层控件的 onTouchEvent() 来消费这个事件？？？
    
    【回答】：dispatchTouchEvent return true 只是终止传递，具体查看下面标题6 的内容
    
3. **dispatchTouchEvent 和 onTouchEvent return false的时候事件都回传给父控件的onTouchEvent处理。**
    * **对于dispatchTouchEvent 返回 false 的含义应该是**：事件停止往子View传递和分发同时开始往父控件回溯（父控件的onTouchEvent开始从下往上回传直到某个onTouchEvent return true），事件分发机制就像递归，return false 的意义就是递归停止然后开始回溯。
    * **对于onTouchEvent return false 就比较简单了**，就是它不消费这个事件，并让事件继续往父控件的方向从下往上流动。
    

4. **dispatchTouchEvent、onTouchEvent、onInterceptTouchEvent**

    ViewGroup 和View的这些方法的默认实现就是会让整个事件安装U型完整走完，所以 return super.xxxxxx() 就会让事件依照U型的方向的完整走完整个事件流动路径），中间不做任何改动，不回溯、不终止，每个环节都走到。
    
5. **onInterceptTouchEvent 的作用**

    Intercept 的意思就拦截，每个ViewGroup每次在做分发的时候，问一问拦截器要不要拦截（也就是问问自己这个事件要不要自己来处理）如果要自己处理那就在onInterceptTouchEvent方法中 return true就会交给自己的onTouchEvent的处理，如果不拦截就是继续往子控件往下传。默认是不会去拦截的，因为子View也需要这个事件，所以onInterceptTouchEvent拦截器return super.onInterceptTouchEvent()和return false是一样的，是不会拦截的，事件会继续往子View的dispatchTouchEvent传递。
    
6. **ViewGroup 和View 的dispatchTouchEvent方法返回super.dispatchTouchEvent()的时候事件流走向。**

    首先看下**ViewGroup 的dispatchTouchEvent**，之前说的return true是终结传递。return false 是回溯到父View的onTouchEvent，然后ViewGroup怎样通过dispatchTouchEvent方法能把事件分发到自己的onTouchEvent处理呢，return true和false 都不行，那么只能通过Interceptor把事件拦截下来给自己的onTouchEvent，所以ViewGroup dispatchTouchEvent方法的super默认实现就是去调用onInterceptTouchEvent，记住这一点。
    
    那么对于**View的dispatchTouchEvent** return super.dispatchTouchEvent()的时候呢事件会传到哪里呢，很遗憾View没有拦截器。但是同样的道理return true是终结。return false 是回溯会父类的onTouchEvent，怎样把事件分发给自己的onTouchEvent 处理呢，那只能return super.dispatchTouchEvent,View类的dispatchTouchEvent（）方法默认实现就是能帮你调用View自己的onTouchEvent方法的。
    
7. **小结：**
    * 对于 dispatchTouchEvent，onTouchEvent，return true是终结事件传递。return false 是回溯到父View的onTouchEvent方法。
    * ViewGroup 想把自己分发给自己的onTouchEvent，需要拦截器onInterceptTouchEvent方法return true 把事件拦截下来。
    * ViewGroup 的拦截器onInterceptTouchEvent 默认是不拦截的，所以return super.onInterceptTouchEvent()=return false；
    * View 没有拦截器，为了让View可以把事件分发给自己的onTouchEvent，View的dispatchTouchEvent默认实现（super）就是把事件分发给自己的onTouchEvent。

###3. ACTION_DOWN 事件传递总结


1. **ViewGroup和View 的dispatchTouchEvent 是做事件分发，那么这个事件可能分发出去的四个目标**

    注：——> 后面代表事件目标需要怎么做。
    1. 自己消费，终结传递。——->return true ；
    2. 给自己的onTouchEvent处理——-> 调用super.dispatchTouchEvent()系统默认会去调用 onInterceptTouchEvent，在onInterceptTouchEvent return true就会去把事件分给自己的onTouchEvent处理。
    3. 传给子View——>调用super.dispatchTouchEvent()默认实现会去调用 onInterceptTouchEvent 在onInterceptTouchEvent return false，就会把事件传给子类。
    4. 不传给子View，事件终止往下传递，事件开始回溯，从父View的onTouchEvent开始事件从下到上回归执行每个控件的onTouchEvent——->return false；
    
    **注：** 由于View没有子View所以不需要onInterceptTouchEvent 来控件是否把事件传递给子View还是拦截，所以View的事件分发调用super.dispatchTouchEvent()的时候默认把事件传给自己的onTouchEvent处理（相当于拦截），对比ViewGroup的dispatchTouchEvent 事件分发，View的事件分发没有上面提到的4个目标的第3点。

2. **ViewGroup和View的onTouchEvent方法是做事件处理的，那么这个事件只能有两个处理方式：**

    1. 自己消费掉，事件终结，不再传给谁—–>return true;
    2. 继续从下往上传，不消费事件，让父View也能收到到这个事件—–>return false;**View的默认实现是不消费的。所以super==false。**

3. **ViewGroup的onInterceptTouchEvent方法对于事件有两种情况：**

    1. 拦截下来，给自己的onTouchEvent处理—>return true;
    2. 不拦截，把事件往下传给子View—->return false,**ViewGroup默认是不拦截的，所以super==false；**

###4. 关于 ACTION_MOVE 和 ACTION_UP

1. 简单的说，就是当dispatchTouchEvent在进行事件分发的时候，只有前一个事件（如ACTION_DOWN）返回true，才会收到ACTION_MOVE和ACTION_UP的事件。
2.  **如果在某个控件的dispatchTouchEvent 返回true消费终结事件**，那么收到ACTION_DOWN 的函数也能收到 ACTION_MOVE和ACTION_UP。
3. **对于在onTouchEvent消费事件的情况**：在哪个View的onTouchEvent 返回true，那么ACTION_MOVE和ACTION_UP的事件从上往下传到这个View后就不再往下传递了，而直接传给自己的onTouchEvent 并结束本次事件传递过程。



--
【参考】
* [公共技术点之 View 事件传递](http://b.codekk.com/blogs/detail/54cfab086c4761e5001b253e)
* [可能是讲解Android事件分发最好的文章](http://www.jianshu.com/p/2be492c1df96)
* [图解 Android 事件分发机制](http://mp.weixin.qq.com/s/BJT0qAN_i0lXsJFnTrCJSg)