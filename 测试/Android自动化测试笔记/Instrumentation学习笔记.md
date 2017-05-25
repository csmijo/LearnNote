# Instrumentation 学习笔记

`Instrumentaion`

---

* [移动测试基础: Android Instrumentation 框架简单说明](https://testerhome.com/topics/6592)   
* [移动测试基础: 创建 instrumentation 测试工程](https://testerhome.com/topics/782)
* [Android Instrumentation 简介](http://blog.csdn.net/gb112211/article/details/45419669)


## 1. APIs && Source code

* [官方APIs地址(需要翻墙)](https://developer.android.com/reference/android/app/Instrumentation.html)
* [Source code](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.4.2_r1/android/app/Instrumentation.java#Instrumentation)


## 2. Instrumentation 特点

* 该框架基于`JUnit`，因此既可以直接使用`Junit` 进行测试，也可以使用`Instrumentation` 来测试`Android` 组件
* 其为`Android` 应用的每种组件提供了测试基类
* 可以在`Eclipse` 中方便地创建`Android Test Project`，并将`Test Case`直接以`Android JUnit`的方式运行

## 3. 其他

[移动测试基础: Android Instrumentation 框架简单说明](https://testerhome.com/topics/6592)   对 robotium 框架也有很深如的分享，可以详细研究一下