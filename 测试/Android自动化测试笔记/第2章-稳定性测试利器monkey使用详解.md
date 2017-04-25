# Android monkey使用详解

`测试` `monkey`

--

## 1. monkey 的基本使用

1. `monkey`文档官方网址：https://developer.android.com/studio/test/monkey.html
2. 使用 `monkey` 有两种方式：
    * 第一种方式：shell 端启动
        1. 进入 `adb shell`  
        2. 运行 `"/system/bin"` 路径下的 `monkey`脚本
        
            ```
            $ adb shell
            # cd /system/bin
            # monkey
            ```
    * 第二种方式：直接 pc 启动 
        
        直接通过以下的命令运行：
        
        ```
        $ adb shell /system/bin/monkey
        ```
    * 这两种方式的区别： 通过 PC 端启动，monkey 运行日志可以保存到 PC 上；通过 Shell 端启动，monkey 运行日志可以保存到手机里。
        
3. 加上选项`[options]` 的命令如下：

    ```
    $ adb shell monkey [options] <event-count>
    ```

## 2. monkey 的命令及其使用

`monkey` 的 `option` 操作都是根据具体的需求设定的，主要分为五类，分别为： **常规类、事件类、约束类、调试类和官方隐藏类参数**。

### 2.1 monkey 的常规类命令

![monkey常规类命令.PNG](http://on9czqsf5.bkt.clouddn.com/monkey常规类命令.PNG?2017-04-10-17-03-49)

1. `-h`: 显示 `monkey` 参数帮助信息 usage
2. `-v`: 打印出日志信息，每个 `-v` 将增加反馈信息的级别。命令格式为：
    ```
    $ adb shell monkey -v <event-count>
    ```

  `-v` 越多日志信息月详细，不过目前最多支持 3 个 `-v`，即：
    * 0级： 除启动提示、测试完成和最终结果外提供较少信息
    * 1级： 提供较详细测试信息，如逐个发送 Activity 的事件
    * 2级： 提供更详细安装信息，如测试中被选中或为被选中的 Activity


### 2.2 monkey 的事件类命令

![monkey事件类命令.PNG](http://on9czqsf5.bkt.clouddn.com/monkey事件类命令.PNG?2017-04-10-17-04-14)

1. `-f`: 后接测试脚本名，表示要使用 `monkey` 运行指定的 `monkey` 脚本，命令示例：
    
    ```
    $ adb shell monkey -f <scriptfile>  <event-count>
    $ abd shell monkey -f /mnt/sdcard/test 10
    ```
2. `-s`: 后接随机数生成器的 seed 值。**如果使用相同的seed 值再次运行 monkey，将生成相同的事件序列**，也就是说重复执行刚才的随机操作。

    命令格式为：
    ```
    $ adb shell monkey -s <seed> <event-count>
    ```
3. `--throttle`: 后接时间，单位为 ms(<milliseconds>)，表示事件之间的固定延迟(即执行每一个指令间隔的时间)，如果不接该选项，`monkey` 将不会延迟。

    命令格式：
    ```
    $ adb shell monkey --throttle <milliseconds>
    ```
    
4. `--ptc-touch`: 后接**触摸事件百分比**，命令格式：
    ```
    $ adb shell monkey --ptc-touch <percent>
    ```
    
5. `--ptc-motion`: 后接**动作事件百分比**。动作事件不单单指手势操作，它泛指从某一个位置按下(即Down事件)后经过一系列伪随机事件后弹起(即Up事件)。
6. `--ptc-trackball`: 后接**轨迹球事件百分比**。轨迹球事件包括一系列的随机移动，以及偶尔跟随在移动后面的点击事件。
7. `--ptc-nav`: 后接**基本导航事件百分比**。 基本导航事件主要指来自方向输入设备的上、下、左、右事件。
8. `--ptc-majornav`: 后接**主要导航事件百分比**。主要导航事件通常指引发图形界面的一些动作，如 5-way 键盘中间按键、返回按键、菜单按键等。
9. `--ptc-syskeys`: 后接**系统按键事件百分比**。系统按键事件通常指仅供系统使用的保留按键，比如 home键，back键，拨号键等。
10. `--ptc-appswitch`: 后接**应用启动事件百分比**。医用启动事件俗称 打开应用，通过调用`startActivity()` 方法最大限度地开启该 package 下的所有应用。
11. `--ptc-anyevent`: 后接**其他类型事件百分比**。除了上述提到的事件外全部都属于其他事件。

### 2.3 monkey 的约束类命令

![monkey约束类命令.PNG](http://on9czqsf5.bkt.clouddn.com/monkey约束类命令.PNG?2017-04-10-17-15-55)

1. `-p`: 后接一个或多个包名(<allowed-package-name>)，如果应用需要访问其他包里面的 Activity，那相关的包也需要在此同时指定。如果不指定任何包，`monkey`将允许系统启动全部包里的 Activity。 **每一个 -p 对应一个包，指定多个包时每个包名前都需要加上 -p**，如：
    ```
    $ adb shell monkey -p <allowed-package-name> <event-count>
       
    $ adb shell monkey -p com.csmijo.test 1000
    ```
2. `-c`: 后接一个或多个类别名(即 <main-category> 参数)，`monkey` 将只允许系统启动这些类别中某个类别列出的 Activity。如果不指定任何类别，monkey 将选择`Intent.CATEGORY_LAUNCH` 和 `Intent.CATEGORY_monkey`里的 Activity。

### 2.4 monkey 调试类命令

![monkey调试类命令.PNG](http://on9czqsf5.bkt.clouddn.com/monkey调试类命令.PNG?2017-04-10-17-38-31)


1. `--dbg-no-events`: 在设置此选项后，monkey 将进入初始启动，进入到某个测试 Activity 中不会进一步生成事件。命令格式：
    ```
    $ adb shell monkey --dbg-no-events <event-count>
    ```
    
2. `--hprof`: 在设置此项后，将在`monkey`事件序列前后立即生成 `profiling report`。 **该选项将在 data/misc 中生成 5MB 大小的文件，慎用！**
3. `--ignore-crashes`: 在设置此项后，当应用程序崩溃或者发生失控异常时， `monkey` 将继续运行直到计数完成。如果不设置此选项，`monkey` 遇到上述崩溃或者异常将停止运行。
4. `--ignore-timeouts`: 在设置此选项后，当应用程序发生任何超时错误(如ANR)时，`monkey` 将继续运行直到计数结束。如果不设置此选项，`monkey` 遇到此类超时对话框将停止运行。
5. `--ignore-security-exceptions`: 在设置此选项后，当应用程序发生任何权限错误(如启动一个需要某些权限的 Activity)时，`monkey` 将继续运行直到计数完成。如果不设置此选项，`monkey` 遇到此类权限错误将停止运行。
6. `--kill-process-after-error`: 在设置此选项后，当`monkey` 因为应用程序发生错误停止时，将会通知系统体质发生错误的进程。如果不设置此项，在`monkey` 停止时发生错误的应用程序将继续处于运行状态。
7. `--monitor-nativie-crashes`: 在设置此选项后，`monkey` 运行时 `native code` 的崩溃事件将被监视被报告。如果不设置则不会监视。
8. `--wait-dbg`: 在设置此选项后，将暂停执行中的 `monkey`,知道有调试器与它连接。

### 2.5 官方隐藏类参数

1. `--pkg-blacklist-file`: 限制 `monkey` 不测试指定黑名单文档中记录的包(package)。如果没有使用这个参数，monkey 会测试系统内所有的的包。如果使用了这个参数，可通过在黑名单文档中记录所有不需要测试的包名称，来相纸 monkey 的执行范围。 **黑名单文档中每一行只能放一个包名**

2. `--pkg-whitelist-file`: 限制`monkey` 只测试指定的白名单文档中记录的包。如果没有使用这个参数，monkey 会测试系统内所有的包。如果使用了这个参数，可通过在白名单文档内记录所有要测试的包，来限制monkey 的执行范围。**白名单文档中每一行只能放一个包名。**

    **注意：**如果要测试的包与其他的包有关联，则必须一起指定这些包来执行这项参数。

## 3. monkey 脚本编写

### 3.1 monkey API 详解

1. **轨迹球事件**

    ```
    DispatchTrackball(long downTime, long eventTime, int action, float x, float y, float pressure, float size, int metaState, float xPrecision, float yPrecision, int device, int edgeFlags)
    ```
   **只需要关注： action、x、y 即可**
   * ACTION_DOWN = 0
   * ACTION_UP = 1
   * ACTION_MULTIPLE = 2
   
2. **输入字符串事件**

    ```
    DispatchString(String text)
    ```
    
3. **点击事件**

    ```
    DispatchPointer(long downTime, long eventTime, int action, float x, float y, float pressure, float size, int metaState, float xPrecision, float yPrecision, int device, int edgeFlags)
    ```
    **只需要关注： action、x、y 即可**
    
4. **启动应用**

    ```
    LaunchActivity(String pkg_name,String cl_name)
    ```
    
5. **等待事件**

    ```
    UserWait(long sleeptime)
    ```
    **时间的单位为：毫秒(millisecond)**
6. **按下键值**

    ```
    DispatchPress(int keyCode)
    ```
    
7. **长按键值**

    ```
    LongPress(int keyCode)
    ```
    
8. **发送键值**

    ```
    DispatchKey(long downTime, long eventTime, int action, int code, int repeat, int metaState, int device, int scancode)
    ```
    
9. **开关软键盘**

    ```
    DispatchFlip(boolean keyboardOpen)
    ```
    
### 3.2 monkey 脚本编写

```
type= raw events
count= 10
speed= 1.0
start data >>
captureDispatchPointer(5109520,5109520,0,230.75429,458.1814,0.20784314,0.06666667,0,0.0,0.0,65539,0)
captureDispatchKey(5113146,5113146,0,20,0,0,0,0)
captureDispatchFlip(true)
...
```

## 4. monkey 日志分析

### 4.1 monkey 日志的保存方法

1. 保存在 pc 中，命令如下：
    
    ```
    $ adb shell monkey [options] <event-count> > d:\monkeylog.txt
    ```
    
2. 保存在手机中，命令如下：

    ```
    $ adb shell
    # monkey [options] <event-count> /mnt/sdcard/monkeylog.txt
    ```
    
3. 标准流与错误流分开保存，命令如下：

    ```
    # monkey [options] <event-count> 1>/mnt/sdcard/monkeylog.txt 2>/mnt/sdcard/monkeyErrorlog.txt
    ```
    
### 4.2 monkey 日志内容解析

1. 搜索关键字"ANR" 查找 ANR 相关信息
2. 搜索关键字"CRASH" 查找 Crash 相关信息