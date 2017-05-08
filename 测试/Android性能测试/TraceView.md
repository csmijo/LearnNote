# TraceView 学习笔记

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


---
[参考文献]

* [正确使用Android性能分析工具——TraceView](http://blog.jobbole.com/78995/)
* [Android 编程下的 TraceView 简介及其案例实战](http://www.cnblogs.com/sunzn/p/3192231.html)
* [Android application (performance and more) analysis tools - Tutorial](http://www.vogella.com/tutorials/AndroidTools/article.html)