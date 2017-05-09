# TraceView 学习笔记

`TraceView` `测试`

---

## 1. TraceView 简介

`TraceView` 是 `Android` 平台配备一个很好的性能分析的工具。它主要用于分析 `Android` 中应用程序的 `hotspot`，它可以通过图形化的方式让我们了解我们要跟踪的程序的性能，并且能具体到 `method`。详细内容参考：[Profiling with Traceview and dmtracedump](https://developer.android.com/studio/profile/traceview.html)

## 2. TraceView 的使用

使用 `TraceView` 主要有两种方式：

1. **没有目标应用源码的情况下**，可以使用 `DDMS`来进行数据的采集。具体的步骤：
    * 选择需要分析的进程
    * 点击 `DDMS`界面中的 `Start Method Profiling` 按钮，等红色的小点变成黑色之后就表示 `TraceView` 已经开始工作了。然后就可以进行应用的操作了，**操作最好不要超过 5 秒，因为最好是进行小范围的性能测试**。然后再点击一次 `Stop Method Profiling` 按钮，则会出现另外一个界面。

2. **在有源码的情况下**， 在源码中使用 `android.os.Debug.startMethodTracing()` 和　`android.os.Debug.stopMethodTracing()` 方法。当运行这两个方法之间的代码时就会有一个 `trace` 文件在 `/sdcard` 目录中生成。也可以通过`startMethodTracing(String traceName)` 设置 `trace` 文件的文件名，最后可以通过 `adb pull /sdcard/xxx.trace  /tmp` 命令将 `trace` 文件拉取到电脑中，然后使用 `DDMS` 工具进行分析。

3. 使用 `DDMS` 的方法适用范围更广，但是使用`startMethodProfiling()` 的方法更加精确一些。

## 3. TraceView 界面

![traceview界面.PNG](http://on9czqsf5.bkt.clouddn.com/traceview界面.PNG?2017-05-09-10-48-24)


整个 `TraceView` 如上图所示，其中需要注意的地方如图标识:

1. 被监控应用的进程名
2. `Start Method Profiling`,`Stop Method Profiling` 按钮
3. 测试进程中每个线程的执行情况，每个线程占一行
4. 时间轴
5. 显示当前选中的方法
6. 当选择某个方法之后(比如：Handler.dispatchMessage())会显示两部分数据：
    * `Parents` : 表示调用 `Handler.dispatchMessage()` 方法的父方法，可以看到方法编号为 0
    * `Children` : 表示 `Handler.dispatchMessage()` 方法内部调用的其他方法，可以看到它调用了三个方法

7. `traceView` 的指标参数列，详细列出了每个方法的 CPU 使用时间，调用次数等，这些信息是查找 `hotspot` 的关键。

    |列名|描述|
    |:--|:--|
    |Name|该线程运行过程中所调用的方法名|
    |Incl Cpu Time|某方法占用的 CPU 时间，**包含内部调用其他方法的 CPU 时间**|
    |Excl CPU Time|某方法占用的 CPU 时间，**但是不包含**内部调用其他方法的 CPU 时间|
    |Incl Real Time|某方法运行的真实时间(以毫秒为单位)，**包含调用其他方法所占用的真实时间**|
    |Excl Real Time|某方法运行的真实时间(以毫秒为单位)，**但是不包含**调用其他方法所占用的真实时间|
    |Call+RecurCalls/Total|某方法被调用次数以及递归调用的次数|
    |Cpu Time/Call|某方法调用 CPU 时间与调用次数的比，也就是该方法平均执行时间|
    |Real Time/Call|某方法真实的平均执行时间|

## 4. TraceView 实战

了解完 `TraceView` 的界面后，现在介绍如何利用 `TraceView` 来查找 `hotspot`。一般而言，`hotspot` 包括两种类型的函数：

* 一类是调用次数不多，但每次调用却需要花费很长时间的函数。
* 一类是那些自身占用时间不长，但调用却非常频繁的函数。

这里主要使用 [Android application (performance and more) analysis tools - Tutorial](http://www.vogella.com/tutorials/AndroidTools/article.html) 提供的示例来进行练习。

我的结果分析步骤如下：

1. 按照 `Incl Cpu Time`进行降序排列
2. 从 Top1 的方法开始，逐层的进入它的 `Children` 方法，判断的标准是 别调用方法的`Incl Cpu Time`占调用方法的百分比。

   比如上图中 `Handler.dispatchMessage()` 的 `Children` 中，调用 `Handler.handleCallback()` 方法占据了 99.8% 的比例，所以下一步就进入到 `Handler.handleCallback()` 方法进行进一步的分析
    
3. 这样一层层的分析，直到找到我们监控的应用的方法，然后再详细查看该方法具体在那块存在性能问题。

    ![traceview实战.PNG](http://on9czqsf5.bkt.clouddn.com/traceview实战.PNG?2017-05-09-11-27-58)
    
    以上述 `demo` 为例，应用的包名为 `csmijo.com.traceviewdemo`。 一层层分析后发现在 `csmijo/com/traceviewdemo/MyArrayAdapter.getView()`方法中出现问题，而该方法主要的 `CPU` 消耗用在标识的三个方法的调用中，所以就需要在相应的代码处查看并进行优化。
    


以上是我个人学习使用 `TraceView` 的经验，尤其实战部分，仅供参考，如果有任何错误的地方，请帮忙指正，谢谢。



---
[参考文献]

* [正确使用Android性能分析工具——TraceView](http://blog.jobbole.com/78995/)
* [Android 编程下的 TraceView 简介及其案例实战](http://www.cnblogs.com/sunzn/p/3192231.html)
* [Android application (performance and more) analysis tools - Tutorial](http://www.vogella.com/tutorials/AndroidTools/article.html)