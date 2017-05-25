# UIAutomator框架及实战

`测试` `UIAutomator`

--

## 1. UIAutomator 简介

`UIAutomator` 是 `Android` 官方提供的一套**黑盒自动化测试框架**。

### 1.1 适用范围

目前 `UIAutomator` **仅支持API Level 17 及以上**，而且控件解析**仅支持 Android 原生控件，对于 WebView 则无法解析。**

`UIAutomator` 可以在**无源码** 的情况下和竞品进行对比测试，还可以进行 **单元测试、性能测试、压力测试**。`UIAutomator` 可以进行 ROM 层级的测试，或是 APP 间协作需跨进程的测试。


## 2. UIAutomator 快速上手

UIAutomator 测试先忙的整体流程大体上可以分为以下的步骤：

1. 分析应用 UI 界面元素，获取元素属性
2. 创建测试用例，编码模拟用户操作过程
3. 编译测试代码为测试 jar 包，push 到终端
4. 在设备上运行测试，查看测试结果
5. 修改发现的 BUG， 修复并重新测试

### 2.1 界面元素获取

使用 `uiautomatorviewer.bat` 查看和获取界面元素信息, 一般位于 `Android SDK\tools\` 目录下

【疑问】

界面中有两个按钮，
* 一个的提示是 "`Device Screenshot(uiautomator dump)`" ，
* 一个的提示是 "`Device Screenshot with compressed Hierarchy(uiautomator dump --compressed)`"

这两个的区别是什么？？ 

uiautomator dump --compressed 是简洁版信息

http://blog.csdn.net/wanglha/article/details/42969439


### 2.2 项目环境配置

1. 创建 Java 项目，非 Android 项目
2. 添加依赖的 jar 包， `android.jar` 和 `uiautomator.jar` ，这两个 jar 包一般位于 `\Android SDK\platforms\android-xx` 目录下，其中 xx 表示 `API　Level`, 建议使用 `API 19` 以上（新增了 `UiSelector.resourceId` 系列方法）

![uiautomator编译与运行测试代码](http://on9czqsf5.bkt.clouddn.com/uiautomator%E7%BC%96%E8%AF%91%E4%B8%8E%E8%BF%90%E8%A1%8C%E6%B5%8B%E8%AF%95%E4%BB%A3%E7%A0%81.PNG)

**注意：** 创建 build 文件的命令中的 `-t` 参数，表示编译的 Android 版本在本机（PC）中的序号，如图：

![uiautomator-anroid-list.PNG](http://on9czqsf5.bkt.clouddn.com/uiautomator-anroid-list.PNG?2017-05-16-09-56-46)

## 3. UiDevice API

参考文献：
* https://wenku.baidu.com/view/7fe534bfa32d7375a517804b.html###
* 官方文档: https://developer.android.com/reference/android/support/test/uiautomator/UiDevice.html



有两种方式可以调用 UIDevice：
* `UiDevice.getInstance()`
* `getUiDevice()`

**推荐使用 `UiDevice.getInstance()` 进行书写**。

因为如果使用 `getUiDevice()` 进行调用，在同一个类中不会有问题，但是如果要在其他类中的方法使用了 `getDevcice()` ，调用到本类中不会有语法错误，但是执行的时候会出错。

### 3.1 click

`boolean click(int x,int y)`

### 3.2 freezeRotation

`void freezeRotation()`

禁用传感器和设备的旋转且在当前的旋转状态冻结

### 3.3 getCurrentPackageName

`String getCurrentPackageName()`

返回当前界面的包名的字符串

### 3.4 getDisplayHeight 与 getDisplayWidth

* `int getDisplayHeight()`
* `int getDisplayWidth()`

获取显示器的高度、宽度，以像素为单位

### 3.5 getDisplayRotation

`int getDisplayRotation()`

返回当前的显示旋转，0,90,180,270；分别用 0,1,2,3 来代表

### 3.6 getDisplaySizeDp

`Point getDisplaySizeDp()`

返回显示 DP 大小(设备独立的像素)返回的显示大小根据每个屏幕旋转

### 3.7 getProductName 

`String getProductName()`

返回当前设备的产品名

### 3.8 监听器

1. `void registerWatcher(String name, UiWatcher watcher)`

    注册一个监听器，当前运行指定步骤被打断的时候，处理中断异常
    
2. `void removeWatcher(String name)`
3. `void resetWatcherTriggers()`
4. `void runWatchers()`

    强制运行所有的监听器
    
5. `boolean hasAnyWatcherTriggered()`

    检查是否有监听器触发过
    
6. `boolean hasWatcherTriggered(String watcherName)`

    检查某个特定的监听器是否触发过
    
### 3.9 按键事件

1. `boolean pressBack()`   模拟短按返回键. 
2. `boolean pressDPadCenter()`   轨迹球 
3. `boolean pressDPadDown()`    轨迹球 
4. `boolean pressDPadLeft()`    轨迹球 
5. `boolean pressDPadRight()`   轨迹球
6. `boolean pressDPadUp()`      轨迹球
7. `boolean pressDelete()`    **模拟短按删除键**. 
8. `boolean pressEnter()`   **模拟短按回车键**. 
9. `boolean pressHome()`    **模拟短按HOME键**.  
10. `boolean  pressKeyCode(int keyCode, int metaState)`   模拟短按键盘代码.  键盘代码可以参考谷歌官网： https://developer.android.com/reference/android/view/KeyEvent.html
11. `boolean pressKeyCode(int keyCode)`   模拟短按键盘代码
12. `boolean pressMenu()`     **模拟短按MENU键**
13. `boolean pressRecentApps()`     **模拟短按最近应用程序按键**
14. `boolean  pressSearch()`   **模拟短按搜索键**
 
### 3.10 旋转

1. `void freezeRotation()`  禁用传感器和冻结装置物理旋转在其当前旋转状态 
2. `void unfreezeRotation()`    重新启用传感器和允许物理旋转 
3. `boolean isNaturalOrientation()`    检查设置是否是在其自然旋转竖屏的位置上 
4. `void setOrientationLeft()`    通过禁用传感器，然后模拟设备向左转，并且固定位置 
5. `void setOrientationNatural()`    通过禁用传感器，然后模拟设备转到其自然默认的方向，并且固定位置 
6. `void  setOrientationRight()` 
通过禁用传感器，然后模拟设置向右转，并且固定冻结在那
 
### 3.11 锁屏与唤醒

1. `void sleep()`   **锁屏**  模拟按电源键，如果屏幕已经是关闭的则没有任何作用 
2. `void wakeUp()`  **唤醒**  模拟按电源键，如果屏幕是唤醒的没有任何作用 
3. `boolean isScreenOn()`   检查屏幕是否唤醒

### 3.12 等待对象

1. `void waitForIdle(long timeout)`  等待当前应用程序处于空闲状态 
2. `void  waitForIdle()`  等待当前的应用程序处于空闲状态 
3. `boolean  waitForWindowUpdate(String packageName, long timeout)`    等待窗口内容更新事件的发生 

### 3.13 截图

1. `boolean  takeScreenshot(File storePath)`     把当前窗口截图并将其存储为png默认1.0f的规模(原尺寸)和90%质量， 参数为file类的文件路径  
2. `boolean  takeScreenshot(File storePath, float scale, int quality)`      把当前窗口截图为png格式图片，可以自定义缩放比例与图片质量
 
    参数：  
    * storePath：存储路径，必须为png格式 
    * Scale：缩放比例，1.0为原图  
    * Quality：图片压缩质量，范围为0-100

    返回：  
    * True：截图成功 
    * False：截图失败

### 3.14 拖拽与滑动

1. `boolean swipe(Point[] segments, int segmentSteps)`    在点阵列中滑动，5ms一步  
2. `boolean swipe(int startX, int startY, int endX, int endY, int steps)`     通过坐标**滑动屏幕**  
3. `boolean  drag(int startX, int startY, int endX, int endY, int steps)`   从一个坐标到另一个坐标进行**拖拽** 

    参数：
    * segments：Poing[]      点阵列，可以多个点 
    * segmentSteps：滑动步长
    * StartX-StartY： 具体坐标值 
    
    返回：  
    * True：滑动成功 
    * False：滑动

### 3.15 通知栏·快速设置

1. `boolean openNotification()`   打开通知栏  
2. `boolean openQuickSettings()`   打开快速设置

### 3.16 窗口布局结构

1. `void dumpWindowHierarchy(String fileName)`   用于调试转储当前窗口的布局层次结构。文件保存在/data/local/tmp 
2. `void setCompressedLayoutHeirarchy(boolean compressed)`  启用或禁用布局层次压缩。

## 4. UiSelector API

`UiSelector` 就是通过各种属性与节点关系定位组件。如果匹配到多个组件，则返回第一个匹配的组件。 **可以使用链式调用多个属性来定位组件**。

### 4.1 UiSelector 使用组件属性介绍

|属性值|值类型|示例|
|:--|:--|:--|
|index|int |0|
|instance|int |5
|class|String|android.wideget.TextView
|package|String|com.csmijo.test
|Content-desc|String|string
|checkable|boolean|false
|checked|boolean|false
|clickable|boolean|true
|enabled|boolean|false
|focusable|boolean|false
|focused|boolean|false
|Scrollable|boolean|false
|Long-clickable|boolean|false
|password|boolean|false
|selected|boolean|false
|bounds|Rect|[366,999][708,1197]
|text|String|联系人

### 4.2 四种匹配关系介绍

1. 完全匹配(默认)
2. 包含匹配(Contains)
3. 正则匹配(Matches)，可以包含其他三种匹配
4. 起始匹配(StartsWith)

### 4.3 节点关系介绍

1. xml 文档节点关系
    * 父    Parent
    * 子    Children
    * 同胞  Sibling
    * 先辈  Ancestor
    * 后代  Descendant

### 4.4 对象搜索——文本与描述

1. `UiSelector text(String text)`
2. `UiSelector textContains(String text)`
3. `UiSelector textMatches(String text)`
4. `UiSelector textStartsWith(String text)`
5. `UiSelector description(String text)`
6. `UiSelector descriptionContains(String text)`
7. `UiSelector descriptionMatches(String text)`
8. `UiSelector descriptionStartsWith(String text)`

### 4.5 对象搜索——类名与包名

1. 类名属性定位对象
    * `UiSelector className(String className)`
    * `UiSelector classNameMatches(String regex)`

2. 包名属性定位对象
    * `UiSelector packageName(String name)`
    * `UiSelector packageNameMatches(String regex)`

### 4.6 对象搜索——索引与实例

1. 索引与实例说明
    * 索引 `index` 指的是在同级类中的编号，在兄弟层级的编号
    * 实例 `instance` 指的是在同一个布局文件同一个类中的编号

2. 索引与实例属性定位对象
    * `UiSelector index(int index)`
    * `UiSelector instance(int instance)`

### 4.7 对象搜索——特殊属性与节点

1. 特殊属性定位对象
    * `UiSelector checked(boolean val)`   选择属性
    * `UiSelector clickable(boolean val)`   可点击属性
    * `UiSelector enabled(boolean val)`     enabled属性
    * `UiSelector focusable(boolean val)`     焦点属性
    * `UiSelector focused(boolean val)`       当前焦点属性
    * `UiSelector longClickable(boolean val)`    长按属性
    * `UiSelector scrollable(boolean val)`      滚动属性
    * `UiSelector selected(boolean val)`        背景选择属性

2. 节点属性定位对象

* `UiSelector childSelector(UiSelector selector)`  从当前类中往下递归找符合条件的子类组件(**一般用来找子类**)，也就是说在当前类的子子孙孙中查找符合条件的组件
* `UiSelector fromParent(UiSelector selector)`   从父类往下递归找符合条件的组件 (**一般用来找兄弟类**)

示例：

```
// childSelector() 的使用
UiScrollable scrollable = new UiScrollable(new UiSelector().scrollable(true));   // 找到当前对象listview

UiScrollable scrollable = new UiScrollable(new UiSelector().scrollable(true)
.childSelector(new UiSelector().text("Android")));   // 在当前对象的子类中查找 text 为 Android 的对象

scrollable.click();
```

```
// fromParent() 的使用
UiObject Obj = new UiObject(new UiSelector().resourceId("xxx"));  // 找到当前对象

UiObject parentObj = new UiObject(new UiSelector().resourceId("xxx")
.fromParent(new UiSelector().className("android.widget.LinearLayout").index(1)));  // 从当前对象的父类中查找 className 为 LinearLayout 并且 index 为 1 的对象

parentObj.click();
```


### 4.8 对象搜索——资源ID

**注意：** 只能在 Android 4.3 及以上才能使用，在之前的版本不能使用

1. 资源ID定位对象
    * `UiSelector resourceId(String id)`
    * `UiSelector resourceIdMatches(String regex)`


## 5. UiObject API

`UiObject` 代表一个组件对象，对象有很多模拟实际操作手机的方法和属性，主要包括： **点击/长按，拖动/滑动，属性，对象是否存在，文本输入与清除，手势操作，获取子类**

### 5.1 点击与长按

* `boolean click()`
* `boolean clickAndWaitForNewWindow(long timeout)`
* `boolean clickAndWaitForNewWindow()`
* `boolean clickBottomRight()`
* `boolean clickTopLeft()`
* `boolean longClick()`
* `boolean longClickBottomRight()`
* `boolean longClickTopLeft()`

### 5.2 拖拽与滑动

可以拖拽到另一个组件上，或者拖拽到另一个点上

* `boolean dragTo(UiObject destObj,int steps)`
* `boolean dragTo(int destX,int destY, int steps)`
* `boolean swipeDown(int steps)`
* `boolean swipeLeft(int steps)`
* `boolean swipeRight(int steps)`
* `boolean swipeUp(int steps)`
 

### 5.3 输入文本与清除文本

* `boolean setText(String text)` 
    * **注意：只能输入英文，中文需要特殊处理**
    * 假如在已有文字的情况下进行输入，会先进行删除操作，然后再进行输入操作
* `void clearTextField()`
    * 在某些输入框中使用会出错，比如短信联系人输入框，这是就需要通过自定义的删除方法进行删除，比如移动光标到行尾，按del 键机型删除

### 5.4 获取对象的属性与属性的判断

1. 获取对象的属性

* `Rect getBounds()`             获取对象矩形坐标，矩形左上角坐标和右下角坐标
* `int getChildCount()`      获取下一级子类数量
* `String getClassName()`     获取对象类名属性的类名文本
* `String getContentDescription()`   
* `String getPackageName()`
* `String getText()`
* `Rect getVisibleBounds()`   返回可见视图的范围，如果视图的部分是可见的，是有可见部分报告的范围

### 5.5 手势操作

* `boolean performMultiPointerGesture(PointerCoords[] ... touches)`
* `boolean performTwoPointerGesture(Point startPoint1,Point startPoint2,Point endPoint1,Point endPoint2)`
* `boolean pinchIn(int percent,int steps)`   其中，percent指的是对角线的百分比
* `boolean pinchOut(int percent,int steps)`

### 5.6 判断对象是否存在

* `boolean waitForExists(long timeout)`      等待对象出现
* `boolean waitUtilGone(long timeout)`    等待对象消失
* `boolean exists()`      检查对象是否存在

## 6. UiCollection API

### 6.1 UiCollection 类介绍

1. UiCollection 类说明
    * UiCollection 是 UIObject 的子类
    * UiCollection 代表元素条目集合，代表同一种类型的合集

2. UiCollection功能说明
    * 先按照一定的条件枚举出容器类界面所有符合条件的子元素
    * 再从符合条件的元素再次通过一定的条件最终定位需要的组件

3. UiCollection 使用场景
    * 一般使用容器类组件作为父类
    * 一般使用在需要找子类且子类由于某些因素不好定位
    * 获取某一类的数量，如获取联系人列表下当前视图下联系人的数量

### 6.2 从集合中查找对象

* `UiObject getChildByDescription (UiSelector childPattern, String text)`
* `UiObject getChildByText (UiSelector childPattern, String text)`
* `UiObject getChildByInstance (UiSelector childPattern, int instance)`

在 `UiSelector` 选择器的查找条件中从子 `ui` 元素中搜索，**递归寻找**所有符合条件的子集。再次用`description/text/instance` 条件从前面搜索子集定位到想要的元素

参数：
* `childPattern`： `UiSelector` 从子元素中搜索的条件一
* `text,instance` 从搜索出的元素中再次用 `text/instance` 条件二搜索元素


先找到父类 `UiCollection`，然后找出符合条件一 `UiSelector` 的所有组件的集合，然后再在集合中根据条件二进一步查找需要的目标组件

### 6.3 获取某种搜索条件组件的数量

* `int getChildCount(UiSelector childPattern)`
* `int getChildCount()`

第二个方法返回的是 `UiCollection` 的直接子类的个数，**只包含直接子类**，不包含后代。第一个方法会在 `UiCollection` 中**递归的查找**符合条件的组件的数量

## 7. UiScrollable API

### 7.1 UiScrollable 类介绍

1. `UiScrollable` 是 `UiCollection` 的子类
2. `UiScrollable` 是专门处理滚动事件，提供各种滚动方法

### 7.2 快速滚动

1. `boolean flingBackward()`        以步长为5滑动
2. `boolean flingForward()`        以步长为5滑动
3. `boolean flingToBeginning(int maxSwipes)`        以步长为5滑动
4. `boolean flingToEnd(int maxSwipes)`        以步长为5滑动

### 7.3 获取列表子元素

![UiScrollable获取列表子元素.png](http://on9czqsf5.bkt.clouddn.com/UiScrollable获取列表子元素.png?2017-04-24-19-52-42)

**注意**： `getChildByInstance()` **是不能滚动的**，他只能搜索当前页面的组件

### 7.4 获取与设置最大滚动次数常量值

1. `int getMaxSearchSwipes()`      **默认常量值为 30**
2. `UiScrollable setMaxSearchSwipes(int swipes)`

查找时先在当前页面查找，如果当前页面不存在，则**先滑动到原点**，然后再从原点开始寻找。如果没有设置 `swipes` 次数，则默认最多滑动 30 次就停止，如果此时还是没有找到需要的组件，则会等待超时时间，**超时时间为10s**，之后测试结果，报告异常。

### 7.5 滑动取悦校准常量设置与获取

**校准常量：** 指的是滑动操作坐标时的偏移量，用来获取偏移比例
 
1. `double getSwipeDeadZonePercentage()`       **默认为0.1,10%**
2. `UiScrollable setSwipeDeadZonePercentage(double swipeDeadZonePercentage)`     设置一个部件的大小，在滑动时，视为无接触区的百分比

通过设置 `DeadZone` 的百分比可以控制滑动的距离以及滑动的速度。当`DeadZone` 的百分比达到 50%时，直接变成点击事件，不再滑动。

### 7.6 向前向后滚动

![UiScrollable向前滚向后滚API.png](http://on9czqsf5.bkt.clouddn.com/UiScrollable向前滚向后滚API.png?2017-04-24-20-22-01)


### 7.7 滚动到某个对象


![UiScrollable滚动到某个对象.png](http://on9czqsf5.bkt.clouddn.com/UiScrollable滚动到某个对象.png?2017-04-25-15-44-20)

### 7.8 设置滚动方向

1. `UiScrollable setAsHorizontalList()`    水平滚动（左右滚动）
2. `UiScrollable setAsVerticalList()`     纵向滚动（上下滚动）

## 8. UIWatcher API

### 8.1 UiWatcher 类介绍与中断监听检查条件

`UiWatcher`  用于处理脚本执行过程中遇到非预想的步骤

1. `public boolean checkForCondition()`
    
    **在测试框架无法找到一个匹配时**，使用`uiselector` 测试框架会自动调用此方法才处理程序方法。在超时未找到匹配项时，框架调用 `checkForCondition()` 方法查找设备上的所有已注册的监听检查条件。可以使用此方法来处理中断问题保证测试用例正常运行。

### 8.2 监听器操作

1. `void registerWatcher(String name, UiWatcher watcher)`
2. `void removeWatcher(String name)`
3. `void resetWatcherTriggers()`
4. `void runWatchers()`


**注意：**

1. 使用循环体时，比如 for 循环体，一旦监听器被触发，跳出循环体之后是不会再次进入到循环体之内的
2. 使用方法体时，一旦监听器被触发，跳出方法体之后就不能再进入了，所以在使用自己的方法时如果被打断是无法被监听器恢复的
2. `UiDevice` 的操作是不会触发监听器的

## 9. Configurator API

`Configurator` 用于设置脚本动作的默认延时

![ConfiguratorAPI.png](http://on9czqsf5.bkt.clouddn.com/ConfiguratorAPI.png?2017-04-25-18-18-43)

## 10. UiAutomator 报告查看

### 10.1 报告简介及查看

1. 错误类型
    * 断言错误：`AssertionFailedError`
    * 脚本错误：`UiObjectNotFoundException、java异常等等`

2. 报告状态
    * 运行状态
    * 结果状态
    * 运行信息

3. 运行状态
    * 1  : 运行前
    * -1 : 运行完成

4. 结果状态
    * 0  : ok
    * -1 : Errors   脚本错误
    * -2 : Failures  断言错误

5. 运行信息
    * 运行前信息   状态码为： 1
    * 运行中信息
    * 运行后信息   状态码为： -1
6. 运行后打印的结果信息中：
`Test results for WatcherResultPrinter= .` 
    * `.`  表示没有任何异常
    * `.E` 表示脚本错误
    * `.F` 表示断言错误

### 10.2 输出信息到报告

1. 输出信息使用的 API 说明
    * `Bundle`
    * `getAutomationSupport().sendStatus(int, Bundle)`

### 10.3 传入参数控制脚本

1. 通过 `-e` 传入数据到用例中，如：拨打一个电话
2. 通过 `-e` 传入整体控制参数控制脚本，如：清理应用数据

* `Bundle bundle = new Bundle `    新建一个 Bundle 对象
* `Bundle bundle = getParams()`    获取命令行传入的参数
* `String value = (String)bundle.get(key)`  获取命令行传入的具体参数值
* `-e key value`  通过这样的格式从命令行进行参数的输入

## 11. UiAutomator 正则表达式的使用

### 11.1 正则表达式

![正则表达式常用元字符.png](http://on9czqsf5.bkt.clouddn.com/正则表达式常用元字符.png?2017-04-26-17-39-59)

![表示次数的元字符.png](http://on9czqsf5.bkt.clouddn.com/表示次数的元字符.png?2017-04-26-17-40-06)

|描述|JAVA API|
|:--|:--|
|匹配字符串|matcher(regex)|
|替换字符串|replace(regex,input)|
|萃取字符串|Matcher.group()|
|拆分字符串|split(regex)|


```
# 萃取
Stirng s = "testa1234bafasda";
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher(s);
while(matcher.find()){
    System.out.println(m.group());
}
```

## 12. 断言函数

![断言函数分类.PNG](http://on9czqsf5.bkt.clouddn.com/断言函数分类.PNG?2017-04-26-19-59-52)

## 13. UiAutomator 图像处理

### 13.1 在图片上嵌入文字

```
public Bitmap drawTextBitmap(Bitmap bitmap,String text){
	    int x = bitmap.getWidth();
	    int y = bitmap.getHeight();
	    
	    // 创建一个比原来图片更大的位图
	    Bitmap newBitmap = Bitmap.createBitmap(x, y+80, Bitmap.Config.ARGB_8888);
	    Canvas canvas = new Canvas(newBitmap);
	    Paint paint = new Paint();
	    
	    // 在原图位置（0，0）叠加一张图片
	    canvas.drawBitmap(bitmap, 0, 0,paint);
	    // 画笔颜色
	    paint.setColor(Color.parseColor("#FF0000"));
	    paint.setTextSize(30);
	    canvas.drawText(text, 20, y+55, paint);
	    canvas.save(Canvas.ALL_SAVE_FLAG);
	    canvas.restore();
	    return newBitmap;    
	}
```

### 13.2 图片对比

```
public boolean imageSameAs(String targetImagePath,String comPath,double percent){
		Bitmap m1 = BitmapFactory.decodeFile(targetImagePath);
		Bitmap m2 = BitmapFactory.decodeFile(comPath);
		
		int width = m2.getWidth();
		int height = m2.getHeight();
		int numDiffPixels = 0;
		for(int x=0;x<width;x++){
			for(int y=0;y<height;y++){
				if(m2.getPixel(x, y) != m1.getPixel(x, y)){
					numDiffPixels++;
				}
			}
		}
		
		double totalPixels = height*width;
		double diffPercent = numDiffPixels/totalPixels;
		return percent <= 1.0 - diffPercent;
	}
```

## 14. UiAutomator 中辅助 APK 的使用

### 14.1 在测试中弹出提示框

1. 使用 `am broadcast -a com.csmijo.test.action -e key value` 发送一个广播
2. 新建一个 Apk ，它包含一个 `BroadcastReceiver` 可以响应 `com.csmijo.test.action` 这样的 `action`，然后在 `onReceive()` 方法中弹出 一个对话框，用于显示 `value`的信息 

### 14.2 在测试中输入中文

由于 `UiAutomator` 默认只能输入 `ASCII` 字符，所有为了输入其他类型的文字，需要使用 `UiAutomator Unicode` 输入助手来输入，即 **Jutf-7输入法**

使用步骤：

1. 安装 `Jutf-7` 输入法
2. 系统设置 --> 语言和输入法 --> 设置 UTF-7 为默认输入法
3. UTF-7 UiAutomator 辅助工具类加入脚本
4. `UiObject.setText(Utf7lmeHelper.e("各种语言"));`

