# UiAutomator2学习笔记

---


## 1. InstrumentationRegistry 

## 1.1 简介

它是一个暴露的注册实例，持有 `instrumentation` 运行进程和它的参数。还提供了一个简便的方法调用 `instrumentation`, `application context` 和 `instrumentation` 参数

![instrumentationRegistry_api.PNG](http://on9czqsf5.bkt.clouddn.com/instrumentationRegistry_api.PNG?2017-05-16-20-04-58)

**注意：** 最低版本是 `Andriod 4.3` 以上，也就是 `API 18` 以上


部分实例代码：

```
@Test
public void testCase() throws IOException{
    
    // 获取运行时的参数
    Bundle args = InstrumentationRegistry.getArguments();
    // 输出到运行报告中
    instrumentation.setStatus(100,args);
    
    // 获取测试包的 Context
    Context testContext = InstrumentationRegistry.getContext();
    // 获取被测应用的 Context
    Context targetContext = InstrumentationRegistry.getTargetContext();
    
    // 通过 Context 来启动一个 Activity，比如：浏览器
    String url = "https://www.baidu.com";
    Intent i1 = new Intent(Intent.ACTION_VIEW);
    i1.setData(Uri.parse(url));
    i1.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    testContext.startActivity(i1);
    
    // 通过目标 Context 来启动拨号功能
    Intent i2 = new Intent(Intent.ACTION_CALL);
    i2.setData(Uri.parse("tel:"+10086);
    i2.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    targetContext.startActivity(i2);

    Bundle inputBundle = new Bundle();
    inputBundle.putString("key","value");
    // 注入一个 Bundle
    InstrumentationRegistry.registerInstance(instrumentation,inputBundle);
    // 输出到结果报告中
    instrumentation.sendStatus(110,inputBundle);
}
```



## 2. UiDevice 新增 API

![UiDevice新增APi.PNG](http://on9czqsf5.bkt.clouddn.com/UiDevice新增APi.PNG?2017-05-16-20-17-13)

## 3. BySelector 与By 静态类

### 3.1 BySelector 类介绍

* **作用：** 指定匹配 UI 元素的标准搜索条件
* **使用：** 通过 `UiDevice.findObject(BySelector)` 进行使用
* 返回的对象是 `UiObject2` 对象

### 3.2 By 类介绍

* **作用：** 是一个使用程序类，通过它可以以简洁的方式创建 `BySelector`。它的主要功能时使用缩短语法提供静态工厂方法构造 `BySelector`。
* **使用：** 
    *  `findObject(By.text("foo"))`  这种方式比下面的方式就更加的简洁
    *  `findObject(new BySelector().text("foo"))`  其中 `new BySelector()` 是不能直接使用的，因为构造函数不是 `public` 的

* 基本上 `BySelector` 中有的方法 `By` 中都有，但有一些例外。

### 3.3 常规属性搜索

![常规属性搜索.PNG](http://on9czqsf5.bkt.clouddn.com/常规属性搜索.PNG?2017-05-19-09-46-32)
![常规属性搜索2.PNG](http://on9czqsf5.bkt.clouddn.com/常规属性搜索2.PNG?2017-05-19-09-48-38)

与 `UiSelector` 的不同点：

1. 使用简写的形式来缩短代码长度
2. 增加了 `EndsWith()` 的结尾匹配方法
3. 在使用正则匹配的时候，需要先创建一个 `Pattern`，它的参数不在是 `String`

### 3.4 后代搜索

![后代搜索.PNG](http://on9czqsf5.bkt.clouddn.com/后代搜索.PNG?2017-05-19-09-59-13)

1. `子类` 是指当前层级之下的直接层级；`后代` 是指当前层级之下的所有层级。
2. `源码可能存在问题，使用时小心！`

### 3.5 逻辑属性搜索

和 `UiSelector` 的属性都一样

### 3.6 搜索层级深度控制（重要更新）

**概念：** 深度就是 `UI 层级的深度`


![深度搜索API.PNG](http://on9czqsf5.bkt.clouddn.com/深度搜索API.PNG?2017-05-19-10-20-56)

1. `By.depth(int exactDepth)` 只有这一个可以使用 `By` 来直接调用，其他的三个不可以，他们定义在 `BySelector` 中
2. 其他三个的使用方法为： 先使用 `By` 的任意一个搜索，得到返回的 `BySelector`，然后再调用方法


## 4. UiAutomator2 Until 

官方文档： https://developer.android.com/reference/android/support/test/uiautomator/Until.html

### 4.1 状态条件 <UiObject2Condition>

1. 状态改变通过属性的改变来体现
2. 一个 `UiObject2Condition` 代表 `UiObject2` 满足某个条件的特定状态
    * `UiObject2` 对象使用 API： `public <R> R wait(UiObject2Condition<R> condition, long timeout)`


这些方法都是**静态方法**，所以可以直接使用 `Until.xxx` 来调用

![Util状态条件API.PNG](http://on9czqsf5.bkt.clouddn.com/Util状态条件API.PNG?2017-05-19-10-45-36)

![Util状态条件API2.PNG](http://on9czqsf5.bkt.clouddn.com/Util状态条件API2.PNG?2017-05-19-10-48-33)

### 4.2 搜索条件 <SearchCondition>

1. `SearchCondition` 代表满足一定条件的需要查找的 UI 元素，主要用于判断是否存在某个组件。


|返回类型|API|说明|
|:--|:--|:--|
|`static SearchCondition<UiObject2>`|`findObject(BySelector selector)`|Returns a SearchCondition that is satisfied when at least one element matching the selector can be found.|
|`static SearchCondition<List<UiObject2>>`|`findObjects(BySelector selector)`|Returns a SearchCondition that is satisfied when at least one element matching the selector can be found.|
|`static SearchCondition<Boolean>`|`gone(BySelector selector)`|Returns a SearchCondition that is satisfied when no elements matching the selector can be found. 与 selector 搜索条件匹配的组件消失|
|`static SearchCondition<Boolean>`|`hasObject(BySelector selector)`|Returns a SearchCondition that is satisfied when at least one element matching the selector can be found.|


### 4.3 事件条件 <EventCondition>

1. `EventCondition` 是一种条件，它依赖于一个时间或一个系列事件发生，主要用于判断某个事件是否发生了。

|返回类型|API|说明|
|:--|:--|:--|
|`static EventCondition<Boolean>`|`newWindow()`|Returns a condition that depends on a new window having appeared.|
|`static EventCondition<Boolean>`|`scrollFinished(Direction direction)`|Returns a condition that depends on a scroll having reached the end in the given direction.|
 


## 5. UiAutomator2 Gradle 执行测试

### 5.1 Gradle Task介绍

#### 5.1.1 Gradle 介绍

1. `Gradle` 是一种 `依赖管理/自动化构建工具`，和 `ant/maven` 一样。但是又有不同之处，它采用了 `Groovy语言`，并没有使用 `xml` 语言。这使得它更加的简介、灵活。更强大的是，`gradle` 完全兼容 `maven`。
2. `Project`: 项目代表一个正在构建的组件(如一个 jar 文件)，或者一个想要完成的目标(如部署应用程序)
3. `Task`: 任务代表完成一个最小动作
4. Gradle 安装配置
    1. 安装 `JDK`，配置 `JAVA_HOME` 环境变量
        * 因为 `Gradle` 是用 `Groovy` 编写的，而 `Groovy` 基于 `Java`
    2. 下载 `Gradle`
        * 地址： http://www.gradle.org/downloads
    3. 解压 `gradle-xx-all.zip`
    4. 配置环境变量
        * 添加环境变量： `GRADLE_HOME` 值为： `gradle` 根目录
        * 添加到 `PATH`： `%GRADLE_HOME/bin` 加到 `PATH` 的环境变量 
    5. 验证安装
        * 配置完成之后，运行 `gradle -v`，检查是否安装无误 


### 5.2 Android Gradle Task

![android-gradle-task.PNG](http://on9czqsf5.bkt.clouddn.com/android-gradle-task.PNG?2017-05-19-11-20-19)

1. 使用 `gradle tasks` 就可以查看支持的 `task`
2. `assemble`： 也就是编译 `apk`，使用 `gradle assemble` 就可以编译 `apk`
3. `assembleAndroidTest`: 也就是编译组装测试 apk

### 5.3 Gradle 执行测试

#### 5.3.1 执行测试步骤

1. 打开 `Android Studio` 的 `Terminal` 或者打开 `cmd` 命令行，进入工程目录
2. 连接手机到 PC ，输入命令：`gradle connectedCheck` 或者 `gralde cC`
3. 开始执行，等待测试完成
    * 报告目录： `app/build/reports/androidTests/connected/index.html`



