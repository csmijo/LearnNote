#在onTouchEvent中处理任意时间的长按事件

`Android`

--
##A. 方法一 （不是最优）
###1. 欲实现的效果

　　Android提供了GestureDetector类来处理一些常用的手势操作，比如说 onLongPress,onFling 等。但这里不使用GestureDetector，而是直接在自定义View重写的onTouchEvent中进行处理。

　　欲实现的效果是：当手机按住屏幕时，如果在指定的时间内没有移动（如500毫秒）,那么进入长按模式，此时手指在屏幕上移动都算作长按模式。如果手机按住屏幕就立马移动，那么就算作移动模式。


###2. 使用的方法

　　MotionEvent 类提供了记录当前坐标的函数（**getX()**,**getY()**）和当前事件产生的时间的函数（**getEventTime()**）以及按下时间（**getDowntime()**）。MotionEvent同时也提供了当前的操作类型，按下（ACTION_DOWN）、 移动 （ACTION_MOVE）、弹起 （ACTION_UP）。有了这些参数，我们便可以轻易的实现想要的效果了。

###3. 实现思路

　　在按下时记录x,y坐标以及按下时间，当第一次移动的时候获取移动的时间，如果大于指定的长按时间，那么进入长按模式，否则就是普通的移动模式。很容易，在模拟器里面实现了这个效果，但是当在真机里面运行时，却无法实现这样的效果。原来模拟器点击的时候能够保证在不移动鼠标的情况下不触发ACTION_MOVE，但是真机却很敏感，几乎在ACTION_DOWN后的几毫秒之后就立马不停的ACTION_MOVE了。想了一下，其实只要稍微变通下变可以在真机上也实现相同的效果了。那就是判断ACTION_MOVE后的坐标和ACTION_DOWN的坐标的偏移值是否小于我们指定的偏移像素，如果在指定值内，那么认为没有移动。于是有了如下这个函数。

```java
/**
	 * * 判断是否有长按动作发生 * @param lastX 按下时X坐标 * @param lastY 按下时Y坐标 *
	 * 
	 * @param thisX
	 *            移动时X坐标 *
	 * @param thisY
	 *            移动时Y坐标 *
	 * @param lastDownTime
	 *            按下时间 *
	 * @param thisEventTime
	 *            移动时间 *
	 * @param longPressTime
	 *            判断长按时间的阀值
	 */
	static boolean isLongPressed(float lastX, float lastY, float thisX,
			float thisY, long lastDownTime, long thisEventTime,
			long longPressTime) {
		float offsetX = Math.abs(thisX - lastX);
		float offsetY = Math.abs(thisY - lastY);
		long intervalTime = thisEventTime - lastDownTime;
		if (offsetX <= 10 && offsetY <= 10 && intervalTime >= longPressTime) {
			return true;
		}
		return false;
	}
```

　　在ACTION_DOWN的时候，记录下lastX，lastY和lastDownTime，使用getEventTime()，在ACTION_MOVE的时候判断当前是否为长按模式(类标志变量的方式)，如果不是，那么获取当前的thisX，thisY和thisEventTime调用函数进行判断。最后别忘记在ACTION_UP里将长按标志值为FALSE。ACTION_DOWN里面这样处理:

```java
//检测是否长按,在非长按时检测 
	if(!mIsLongPressed){ 
		mIsLongPressed = isLongPressed(mLastMotionX, mLastMotionY, x, y, lastDownTime,eventTime,500); 
		} 
	if(mIsLongPressed){  
		//长按模式所做的事 
		}else{  
			//移动模式所做的事
	}
}
```


##B. 方法二（最优）

###1. 方法一缺点

　　在使用方法一实现长按弹出Dialog的过程中，总是会出现弹出多个Dialog的情况，但是Debug时正确。具体原因还需进一步寻找。。。。。
###2. 实现思路

　　该方法在ActionDown的时候，postDelay一个Runnable出去，delay的时间就是长按的时间，然后在move超过一定距离或者up 的时候，把这个delay的runnable移除掉，就可以达到长按的效果。

###3. 实现代码

```java
@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                touchView = null;

                startX = (int) event.getX();
                startY = (int) event.getY();
                startTime = event.getDownTime();

                if (hasView(startX, startY)) {
                   
                    /* 长按操作 */ 
                    mHandler.postDelayed(new Runnable() {
                        @Override
                        public void run() {
                            longPressedAlertDialog();
                        }
                    }, PRESSREANGE);
                } 
                break;
            case MotionEvent.ACTION_MOVE:
                int lastX = (int) event.getX();
                int lastY = (int) event.getY();
                //移动
                moveView(lastX, lastY);

                if (Math.abs(lastX - startX) > CLICKRANGE || Math.abs(lastY - startY) > CLICKRANGE) {
                    this.mHandler.removeCallbacksAndMessages(null);
                }

                break;
            case MotionEvent.ACTION_UP:
                this.mHandler.removeCallbacksAndMessages(null);
                break;
        }
        return true;
    }
```

--
**参考文献**
* [Android开发：在onTouchEvent中处理任意时间的长按事件](http://blog.csdn.net/eoeandroida/article/details/8308877)