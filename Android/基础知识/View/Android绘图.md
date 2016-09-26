#Android绘图

`Android` `绘图`

--

##1. Drawable资源使用注意事项

1. Drawable分为两种：**一种**是我们**普通的图片资源**，在Android Studio中我们一般放到`res/mipmap`目录下， 和以前的Eclipse不一样哦！另外我们如果把工程切换成Android项目模式，我们直接往`mipmap`目录下丢图片即可，AS会自动分hdpi，xhdpi...!**另一种**是我们编写的**XML形式的Drawable资源**，我们一般把他们放到`res/drawable`目录 下，比如最常见的按钮点击背景切换的Selctor！
2. 在XML我们直接通过`@mipmap`或者`@drawable`设置Drawable即可。比如: `android:background = "@mipmap/iv_icon_zhu",   "@drawable/btn_back_selctor"` ，而在Java代码中我们可以通过Resource的`getDrawable(R.mipmap.xxx)`可以获得drawable资源 如果是为某个控件设置背景，比如ImageView，我们可以直接调用控件`.getDrawale()`同样 可以获得drawable对象！
3. Android中drawable中的资源名称有约束，必须是：**[a-z0-9_.]（即：只能是字母数字及和.）**， 而且不能以数字开头，否则编译会报错： `Invalid file name: must contain only [a-z0-9.]`！ 小写啊！！！！小写！！！小写！——重要事情说三遍~

##2. Drwable,Bitmap,Canvas,Matrix

1. **Drawable**:通用的图形对象，用于装载常用格式的图像，既可以是PNG，JPG这样的图像，也是前面学的那13种Drawable类型的可视化对象！我们可以理解成**一个用来放画的——画框**！
2. **Bitmap(位图)**：我们可以**把他看作一个画架**，我们先把画放到上面，然后我们可以进行一些处理，比如获取图像文件信息，做旋转切割，放大缩小等操作！
3. **Canvas(画布)**：如其名，**画布**，我们可以在上面作画(绘制)，你既可以用**Paint(画笔)**，来画各种形状或者写字，又可以用**Path(路径)**来绘制多个点，然后连接成各种图形！
4. **Matrix(矩阵)**：**用于图形特效处理的**，颜色矩阵(ColorMatrix)，还有使用Matrix进行图像的 平移，缩放，旋转，倾斜等！

##3. Paint，Canvas，Path详解

具体设置方法查看：[三个绘图工具类详解](http://www.runoob.com/w3cnote/android-tutorial-drawable-tool.html)

###1) Paint(画笔)

　　**用于设置绘制风格**,如:线宽(笔触粗细),颜色,透明度和填充风格等 直接使用无参构造方法就可以创建Paint实例: `Paint paint = new Paint( );`

###2) Canvas(画布)

1. Canvas的**构造方法**有两种：
    * Canvas(): 创建一个空的画布，可以使用`setBitmap()`方法来设置绘制具体的画布。
    * Canvas(Bitmap bitmap): 以bitmap对象创建一个画布，将**内容都绘制在bitmap**上，因此bitmap不得为null。
2. **drawXXX()方法族**：以一定的坐标值在当前画图区域画图，另外**图层会叠加， 即后面绘画的图层会覆盖前面绘画的图层**。
    * `drawPath(Path path, Paint paint)`：绘制一个路径，参数一为Path路径对象
    * `drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)` ： 贴图，参数一就是我们常规的Bitmap对象，参数二是源区域(这里是bitmap)， 参数三是目标区域(应该在canvas的位置和大小)，参数四是Paint画刷对象， 因为用到了缩放和拉伸的可能，当原始Rect不等于目标Rect时性能将会有大幅损失。

3. **save()和restore()方法**： 
    * save( )：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作！ 
    * restore（）：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。 
    * save()和restore()要**配对使用**(restore可以比save少,但不能多)，若restore调用次数比save多,会报错！

【参考】
* [Canvas的save和restore](http://www.cnblogs.com/xirihanlin/archive/2009/07/24/1530246.html)

###3) Path(路径)

　　简单点说就是描点，连线~在创建好我们的Path路径后，可以调用Canvas的`drawPath(path,paint)` 将图形绘制出来。

##4. 双缓冲技术

　　先概述一下，双缓冲的核心技术就是先通过`setBitmap()`方法将要绘制的所有的图形绘制到一个`Bitmap`上，然后再来调用`drawBitmap()`方法绘制出这个`Bitmap`，显示在屏幕上。

具体实现步骤如下：
```java
/*1. 定义*/
Canvas bufferCanvas;
Bitmap bufferBitmap;
Paint paint;

/*2. 创建对象*/
bufferCanvas = new Canvas(bufferBitmap);
paint = new Paint();

/*3. 在缓存中绘图*/
bufferCanvas.drawCircle(x,y,radius,paint);

/*4. 在屏幕上绘图*/
canvas.drawBitmap(bufferBitmap,0,0,paint);
```

##5. 最小滑动距离

　　判断`view`的`click` 和`move`操作时，经常会判断滑动的最小距离，标准写法： 
```java
int touchSlop = ViewConfigureation.get(context).getScaledTouchSlop();```