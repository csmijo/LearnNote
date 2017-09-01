# adb am 命令

`adb` `am`

---

在 `Android` 测试中，经常会使用 `am` 命令启动程序，这里对 `am` 的一些命令做个小结。
使用的 Android 系统是 `Android 4.3，API 18`

## 1. am 命令

在`adb shell` 工作区内输入 `am --help`， 可以看到以下的帮助文档：

```
usage: am [subcommand] [options]           # 命令格式
usage: am start [-D] [-W] [-P <FILE>] [--start-profiler <FILE>]      
               [--R COUNT] [-S] [--opengl-trace]
               [--user <USER_ID> | current] <INTENT>     # 启动 Activity
       am startservice [--user <USER_ID> | current] <INTENT>   # 启动 Service
       am force-stop [--user <USER_ID> | all | current] <PACKAGE>  # 强杀进程
       am kill [--user <USER_ID> | all | current] <PACKAGE>     # 杀死指定后台进程
       am kill-all   # 杀死所有后台进程
       am broadcast [--user <USER_ID> | all | current] <INTENT>  # 发送广播
       am instrument [-r] [-e <NAME> <VALUE>] [-p <FILE>] [-w]
               [--user <USER_ID> | current]
               [--no-window-animation] <COMPONENT>    # Typically this target <COMPONENT>  is the form <TEST_PACKAGE>/<RUNNER_CLASS>. 
       am profile start [--user <USER_ID> current] <PROCESS> <FILE>
       am profile stop [--user <USER_ID> current] [<PROCESS>]
       am dumpheap [--user <USER_ID> current] [-n] <PROCESS> <FILE>
       am set-debug-app [-w] [--persistent] <PACKAGE>
       am clear-debug-app
       am monitor [--gdb <port>]       # 监控
       am hang [--allow-restart]       
       am screen-compat [on|off] <PACKAGE>
       am to-uri [INTENT]
       am to-intent-uri [INTENT]
       am switch-user <USER_ID>
       am stop-user <USER_ID>
```

## 2. options

### 2.1 启动 Activity

启动 `Activity` 命令使用 `options` 参数，下面是详细的介绍：

```
1.    -D: enable debugging 允许调试功能
2.    -W: wait for launch to complete  等待 app 启动完成
3.    --start-profiler <FILE>: start profiler and send results to <FILE>  启动 profiler，并将结果发送到 <FILE>
4.    -P <FILE>: like above, but profiling stops when app goes idle  类似 --start-profiler，不同的是当 app 进入 idle 状态，则停止 profiling
5.    -R: repeat the activity launch <COUNT> times.  Prior to each repeat,
        the top activity will be finished.   重复启动 Activity COUNT 次。每次重启之前，最上层的 activity 会先 finish
6.    -S: force stop the target app before starting the activity 启动 activity 之前，先调用 forceStopPackage() 方法强制停止 app
7.    --opengl-trace: enable tracing of OpenGL functions  运行获取 OpenGL 函数的trace
8.    --user <USER_ID> | current: Specify which user to run as; if not
        specified then run as the current user.  指定运行的用户
```

启动 `activity` 的实现原理： 存在 `-W` 参数则调用 `startActivityAndWait()` 方法来运行，否则调用 `startActivityAsUser()` 来运行


## 3. Intent

`Intent` 的参数和 `flags` 比较多，为了方便起见，分为三种类型参数： **常用参数，Extra参数，Flags参数**

### 3.1 常用参数

1. `-a <ACTION>`: 指定`Intent action`， 实现原理`Intent.setAction()`；
2. `-n <COMPONENT>`: 指定组件名，格式为`{包名}/.{主Activity名}`，实现原理`Intent.setComponent()`；
3. `-d <DATA_URI>`: 指定`Intent data URI`，实现原理 `Intent.setData()`？？？？；
4. `-t <MIME_TYPE>`: 指定`Intent MIME Type`，实现原理 `Intent.setType()`????;
5. `-c <CATEGORY> [-c <CATEGORY>] ...]`:指定`Intent category`，实现原理`Intent.addCategory()`;
6. `-p <PACKAGE>`: 指定包名，实现原理`Intent.setPackage()`;
7. `-f <FLAGS>`: 添加`flags`，实现原理`Intent.setFlags(int )`，**紧接着的参数必须是int型；**


实例：

```
am start -a android.intent.action.VIEW
am start -n com.csmijo.test/.MainActivity
am start -d content://contacts/people/1
am start -t image/png
am start -c android.intent.category.APP_CONTACTS
```

### 3.2 Extra 参数

#### 3.2.1 基本类型
 
|参数|	-e/-es|	-esn|	-ez|	-ei|	-el|	-ef|	-eu|	-ecn|
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
|类型|	String|	(String)null|	boolean|	int|	long|	float|	uri|	component|

比如参数es是Extra String首字母简称，实例：

```
am start -n com.yuanhh.app/.MainActivity -es website gityuan.com
```

此处`-es website gityuan.com`，等价于`Intent.putExtra(“website”, “gityuan.com”)`;

#### 3.2.2 数组类型

|参数|-eia|	-ela|	-efa|
|:--|:--|:--|:--|
|数组类型|int[]	|long[]|	float[]|

比如参数`eia`，是`Extra int array`首字母简称，多个`value`值之间以逗号隔开，实例：

```
am start -n com.yuanhh.app/.MainActivity -ela weekday 1,2,3,4,5
```

此处`-ela weekday 1,2,3,4,5`，等价于`Intent.putExtra(“weekday”, new int[]{1,2,3,4,5})`;

### 3.3 Flags 参数

在#3.1 常用参数中，提到有`-f <FLAGS>`，是通过`Intent.setFlags(int )`方法，来设置`Intent`的`flags`.本小节也是关于`flags`，是通过`Intent.addFlags(int )`方法。如下所示，所有的`flags`参数。
```
    [--grant-read-uri-permission] [--grant-write-uri-permission]
    [--debug-log-resolution] [--exclude-stopped-packages]
    [--include-stopped-packages]
    [--activity-brought-to-front] [--activity-clear-top]
    [--activity-clear-when-task-reset] [--activity-exclude-from-recents]
    [--activity-launched-from-history] [--activity-multiple-task]
    [--activity-no-animation] [--activity-no-history]
    [--activity-no-user-action] [--activity-previous-is-top]
    [--activity-reorder-to-front] [--activity-reset-task-if-needed]
    [--activity-single-top] [--activity-clear-task]
    [--activity-task-on-home]
    [--receiver-registered-only] [--receiver-replace-pending]
```

例如，发送`action=”broadcast.demo”`的广播，并且对于`forceStopPackage()`的应用不允许接收该广播，命令如下：

```
am broadcast -a broadcast.demo --exclude-stopped-packages
```

---
[参考文献]

* [Android 调试桥](https://developer.android.com/studio/command-line/adb.html?hl=zh-cn)
* [Am 命令用法](http://gityuan.com/2016/02/27/am-command/)