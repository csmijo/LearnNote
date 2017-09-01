# gradle学习笔记

`gradle`

---
## 0. 重要

* [Gradle版本管理-升级与降级](http://hucaihua.cn/2016/09/27/Gradle_upgrade/)
* [compileSdkVersion , targetSdkVersion 和 minSdkVersion](http://hucaihua.cn/2017/01/10/sdkversion/)

非常的清晰的说明了gradle 的所有事情。

## 1. Android studio 中的 Gradle

### 1.1 自动下载

每一次**重新导入工程时**, 因为默认会使用 `Use default gradle    wrapper (recommended)` 设置, 所以 `As` 会下载 `gradle-wrapper.properties` 中 `distributionUrl` 所指定的 `gradle` 版本. 这个过程在国内普遍巨慢无比. 我的建议是找到上述文件, **动手修改 distributionUrl** 为已下载过的 gradle 版本(比如你之前已经下载过 `gradle-2.14.1` 了), 这样就能直接打开了.

实际上, 这个配置项位于 `As` 自动生成的配置文件(`.idea/gradle.xml`) 

#### 1.1.1 新建工程

操作步骤： `File --> New --> New Project` 新建项目

对于新建项目，As 是这样给我们生成的：

1. 在 project 的 `.idea/gradle.xml` 中可以看到 `GradleProjectSettings` 中有这样的参数设置 

    ```
     <GradleProjectSettings>
        <option name="distributionType" value="LOCAL" />
        <option name="externalProjectPath" value="$PROJECT_DIR$" />
        <option name="gradleHome" value="D:\program\Android Studio\gradle\gradle-2.14.1" />    // 重点
        <option name="gradleJvm" value="1.8" />
        <option name="modules">
          <set>
            <option value="$PROJECT_DIR$" />
            <option value="$PROJECT_DIR$/learnanimation" />
            <option value="$PROJECT_DIR$/learnview" />
          </set>
        </option>
      </GradleProjectSettings>
    ```
    
2. 它在 AndroidStudio 的 `Settings` 的表现是这样的： 
    
    ```
    Gradle --> Project-level settings: 
        Use local gradle distribution:
             Gradle Home: D:\program\Android Studio\gradle\gradle-2.14.1
    ```
    
3. 可见，As 帮我们限定了 gradle 的版本。当然，如果指定的这个 gradle 版本不存在的话，一样会去下载。
    

### 1.1.2 Open 工程

操作步骤： `File --> Open` (**注意**：不是 `Reopen`) 打开一个项目


1. 在 project 的 `.idea/gradle.xml` 中可以看到 `GradleProjectSettings` 中有这样的参数设置 

    ```
    <GradleProjectSettings>
        <option name="distributionType" value="DEFAULT_WRAPPED" />
        <option name="externalProjectPath" value="$PROJECT_DIR$" />
        <option name="modules">
          <set>
            <option value="$PROJECT_DIR$" />
            <option value="$PROJECT_DIR$/app" />
          </set>
        </option>
        <option name="resolveModulePerSourceSet" value="false" />
      </GradleProjectSettings>  
    ```
    
2. 它在 AndroidStudio 的 `Settings` 的表现是这样的： 
    
    ```
    Gradle --> Project-level settings: 
        Use default gradle wrapper(recommended)
    ```
3. 也就是说, 每次你 Open 一个项目的时候, As 会忽略你之前设置的 gradle 版本, 而使用 wrapper 文件中所指定的版本. 所以你 clone 下来的代码第一次打开都会卡很久的原因, 就是因为他们在下载 wrapper 中的 gradle 啊…

### 1.2 gradle-wrapper.properties

AS 创建的工程, 在根目录下有 `gradlew.bat` 和 `gradlew.sh` 文件. 这两个文件会读取 `gradle/wrapper/gradle-wrapper.properties` 并使用其中指定的 `gradle` 版本进行编译. 一般而言这套机制用于 `CI` 服务器中来保障每次编译都在同样的环境下.

```
#Mon Dec 28 10:00:20 PST 2015
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip
```

1. `GRADLE_USER_HOME` 默认值是电脑的 `USER_HOME/.gradle`(windows 下是 `c:·\Users\<username>\.gradle`, linux 下是 `~/.gradle` )
2. 对于 `gradlew` 而言, 如果上述路径没有找到可执行的 `gradle` 文件, 则会使用 `distributionUrl` 中所指定的 url 下载后在执行.


[疑问]

1. Android studio new project 是一个什么样的过程？？
    1. new project 过程中做了哪些操作？？download gradle 版本？？ use defalut 和 use local 的区别？？ gradlew.bat 怎么配合使用？？

2. Android studio open project 有是一个什么样的过程？？
3. Android studio reopen project？？ 
4. 这三种方式打开 project 的区别？？


### 1.3 Android Plugin for Gradle 

#### 1.3.1 简介

```
1. The Android Studio build system is based on Gradle. (Android studio 使用 Gradle 来进行编译)
2. The Android plugin for Gradle adds several features that are specific to building Android apps. (Android plugin for Gradle 添加了一些构建Android app的特性)
3. Although the Android plugin is typically updated in lock-step with Android Studio, the plugin (and the rest of the Gradle system) can run independent of Android Studio and be updated separately. (虽然 Android plugin 是随着 Android studio 的更新而更新的，但是其他 Android plugin和 Gradle程序是可以独立于 Android studio 运行和更新的)
```

#### 1.3.2 对应关系

参考：[Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin.html#revisions)

|Plugin version|Required Gradle version|Required version of SDK Build Tools|
|:--:|:--:|:--:|
|1.0.0|2.2.1--2.3.x|21.1.1 or higher|
|1.0.1|2.2.1--2.3.x|21.1.1 or higher|
|1.1.0|2.2.1 or higher|21.1.1 or higher|
|1.1.1|2.2.1 or higher|21.1.1 or higher|
|1.1.2|2.2.1 or higher|21.1.1 or higher|
|1.1.3|2.2.1 or higher|21.1.1 or higher|
|1.2.0|2.2.1 or higher|21.1.1 or higher|
|1.3.0|2.2.1 or higher|21.1.1 or higher|
|1.3.1|2.2.1 or higher|21.1.1 or higher|
|1.5.0|2.2.1 or higher|21.1.1 or higher|
|2.0.0|2.10  or higher|21.1.1 or higher|
|2.1.0|2.10  or higher|23.0.2 or higher|
|2.1.3|2.14.1 or higher|23.0.2 or higher|
|2.2.0|2.14.1 or higher|23.0.2 or higher|
|2.3.0|3.3 or higher|25.0.0 or higher|
|2.3.1|3.3 or higher|25.0.0 or higher|
|2.3.2|3.3 or higher|25.0.0 or higher|
|2.3.3|3.3 or higher|25.0.0 or higher|

### 1.4 Android stuido如何更改 gradle 版本

参考：[Android studio如何更改gradle版本？](http://www.10tiao.com/html/430/201609/2650037559/1.html)

1. 修改 `Android studio` 工程目录下的 `build.gradle`，修改该文件的：
    ```
     dependencies {
            classpath 'com.android.tools.build:gradle:x.x.x'
        }
    ```
    修改 `Android Plugin for Gradle` 版本号为你本地安装 `Gradle` 对应的版本号，参考上文 1.3.2 的对应关系列表
    
2. 定位到 项目 `gradle` 目录下 `gradle-wrapper.properties` 文件，修改 `distributionUrl` 中的版本号为你本地安装的 `gradle` 版本
3. 在 `Android studio` --> `settings` --> `Gradle` --> `Project-level settings` 中设置 `Use local gradle distribution`, 选择本地安装 `gradle` 的路径
4. `Sync Now` 即可更改成功




## 2. Groovy

### 2.2 Groovy 安装

以 Ubuntu16.04 为例：

1. curl -s "https://get.sdkman.io" | bash
2. source "$HOME/.sdkman/bin/sdkman-init.sh"  或者打开新的 terminal
3. sdk install groovy
4. groovy -version 

### 2.1 Groovy 一些前提知识

1. Groovy注释标记和Java一样，支持//或者/**/ 
2. Groovy语句可以不用分号结尾。Groovy为了尽量减少代码的输入，确实煞费苦心 
3. Groovy中支持动态类型，即**定义变量的时候可以不指定其类型。Groovy中，变量定义可以使用关键字def。注意，虽然def不是必须的，但是为了代码清晰，建议还是使用def关键字**
4. 函数定义时，**参数的类型也可以不指定**。比如
    ```
    String testFunction(arg1,arg2){//无需指定参数类型
      ...
    }
    ```
    
5. 除了变量定义可以不指定类型外，**Groovy中函数的返回值也可以是无类型的**。
6. 函数返回值：Groovy的函数里，可以不使用return xxx来设置xxx为函数返回值。如果不使用return语句的话，则函数里最后一句代码的执行结果被设置成返回值。
7. Groovy对字符串支持相当强大，充分吸收了一些脚本语言的优点
    * 单引号''中的内容严格对应Java中的String，不对$符号进行转义
    * 双引号""的内容则和脚本语言的处理有点像，如果字符中有$号的话，则它会$表达式先求值。
    * 三个引号'''xxx'''中的字符串支持随意换行

8. 除了每行代码不用加分号外，Groovy中函数调用的时候还可以不加括号



### 2.2 Groovy 中的数据类型

Groovy中的数据类型我们就介绍两种和Java不太一样的：

* 一个是Java中的基本数据类型。 
* 另外一个是Groovy中的容器类。 
* 最后一个非常重要的是闭包。

#### 2.2.1 基本数据类型


作为动态语言，Groovy世界中的所有事物都是对象。所以，**int，boolean这些Java中的基本数据类型，在Groovy代码中其实对应的是它们的包装数据类型。比如int对应为Integer，boolean对应为Boolean**

#### 2.2.2 容器类

Groovy中的容器类很简单，就三种：

* List：链表，其底层对应Java中的List接口，一般用ArrayList作为真正的实现类。 
* Map：键-值表，其底层对应Java中的LinkedHashMap。 
* Range：范围，它其实是List的一种拓展。 

实例：

1. list 类： 

    ```
    def aList = [5,'string',true] //List由[]定义，其元素可以是任何对象
    
    变量存取：可以直接通过索引存取，而且不用担心索引越界。如果索引超过当前链表长度，List会自动往该索引添加元素
    ```
    
2. Map 类：

    ```
    def aMap = ['key1':'value1','key2':true] 

    Map由[:]定义，注意其中的冒号。冒号左边是key，右边是Value。key必须是字符串，value可以是任何对象。另外，key可以用''或""包起来，也可以不用引号包起来。比如
    
    def aNewMap = [key1:"value",key2:true] //其中的key1和key2默认被处理成字符串"key1"和"key2",不过Key要是不使用引号包起来的话，也会带来一定混淆。
    ```
   
3. Range 类

    ```
    def aRange = 1..5  <==Range类型的变量 由begin值+两个点+end值表示左边这个aRange包含1,2,3,4,5这5个值

    如果不想包含最后一个元素，则
    
    def aRangeWithoutEnd = 1..<5  <==包含1,2,3,4这4个元素
    println aRange.from
    println aRange.to
    ```

#### 2.2.3 闭包

闭包，是一种数据类型，它代表了一段可执行的代码。其外形如下：
```
def aClosure = {//闭包是一段代码，所以需要用花括号括起来..  
    Stringparam1, int param2 ->  //这个箭头很关键。箭头前面是参数定义，箭头后面是代码  
    println"this is code" //这是代码，最后一句是返回值，  
   //也可以使用return，和Groovy中普通函数一样  
}  
```

简而言之，Closure的定义格式是：

```
def xxx = {paramters -> code}  //或者  
def xxx = {无参数，纯code}  这种case不需要->符号
```

**说实话，从C/C++语言的角度看，闭包和函数指针很像**。闭包定义好后，要调用它的方法就是：
* 闭包对象.call(参数)  
* 或者更像函数指针调用的方法：闭包对象(参数)  

比如：
```
aClosure.call("this is string",100)  或者  
aClosure("this is string", 100) 
```


在闭包中，还需要注意一点：

**如果闭包没定义参数的话，则隐含有一个参数，这个参数名字叫it，和this的作用类似。it代表闭包的参数。**

比如：

```
def greeting = { "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
```

等同于：

```
def greeting = { it -> "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
```


**但是，如果在闭包定义时，采用下面这种写法，则表示闭包没有参数！**

```
def noParamClosure = { -> true }
```

这个时候，我们就不能给noParamClosure传参数了！

```
noParamClosure ("test")  <==报错喔！
```

#### 2.2.4 Closure 使用的注意点

1. 省略圆括号

```
def iamList = [1,2,3,4,5]  //定义一个List
iamList.each{  //调用它的each，这段代码的格式看不懂了吧？each是个函数，圆括号去哪了？
      println it
}
```

上面代码有两个知识点：
* each函数调用的圆括号不见了！原来，**Groovy中，当函数的最后一个参数是闭包的话，可以省略圆括号。**------这个点特别的重要！！！

## 3. Gradle

**Gradle 是一个编程框架**，所以编写所谓的编译脚本，其实就是玩 Gradle 的 API。 它的API：https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html

### 3.1 基本组件

1. Gradle中，每一个待编译的工程都叫**一个Project**。
2. 每一个Project在构建的时候都包含**一系列的Task**。
3. 一个Project到底包含多少个Task，其实是由编译脚本指定的**插件**决定。插件是什么呢？插件就是用来定义Task，并具体执行这些Task的东西。
4. Gradle是一个框架，作为框架，**它负责定义流程和规则**。而具体的编译工作则是通过插件的方式来完成的。


1. **每一个Project都必须设置一个build.gradle文件**。至于其内容，我们留到后面再说。 
2. **对于multi-projects build，需要在根目录下也放一个build.gradle，和一个settings.gradle。** 
3. **一个Project是由若干tasks来组成的，当gradle xxx的时候，实际上是要求gradle执行xxx任务。这个任务就能完成具体的工作。** 
4. 当然，具体的工作和不同的插件有关系。编译Java要使用Java插件，编译Android APP需要使用Android APP插件。这些我们都留待后续讨论 

### 3.2 gradle 命令介绍

1. gradle projects 查看工程信息
2. gradle tasks 查看任务信息
    * gradle \<project-path\>:tasks 其中：\<project-path\> 是目录名，后面必须跟着冒号

3. gradle \<task-name\> 执行命令 
    * gradle tasks会列出每个任务的描述，通过描述，我们大概能知道这些任务是干什么的，然后gradle task-name执行它就好。
    * Task和Task之间往往是有关系的，**这就是所谓的依赖关系**。比如，assemble task就依赖其他task先执行，assemble才能完成最终的输出。

### 3.3 gradle 工作流程

1. Gradle有一个初始化流程(Initiliazation phase)，这个时候settings.gradle会执行。 
2. 在配置阶段(Configration phase)，每个Project 的 build.gradle 都会被解析，其内部的任务也会被添加到一个有向图里，用于解决执行过程中的依赖关系。 
3. 然后才是执行阶段(Execution phase)。你在gradle xxx中指定什么任务，gradle就会将这个xxx任务链上的所有任务全部按依赖顺序执行一遍！ 

### 3.4 gradle编程模型及 API实例详解

**重要文档：** https://docs.gradle.org/current/dsl/


---

[参考文献]
* [android gradle 看这一篇就够了](http://android.walfud.com/android-gradle-看这一篇就够了/)
* [深入理解gradle](http://www.infoq.com/cn/articles/android-in-depth-gradle)
* [Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin.html#revisions)
* [Android studio如何更改gradle版本？](http://www.10tiao.com/html/430/201609/2650037559/1.html)
* [Gradle版本管理-升级与降级](http://hucaihua.cn/2016/09/27/Gradle_upgrade/)