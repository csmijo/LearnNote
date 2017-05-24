# adb常用命令

`adb`

---

原文：http://gityuan.com/2015/06/28/adb-notes/



## 1. 基本指令

1. `adb -s serialNumber shell`  // 进入指定设备
2. `adb version` // 查看版本
3. `adb logcat` // 查看日志
4. `adb devices`  // 查看设备
5. `adb get-state`  // 连接状态
6. `adb start-server`  // 启动 adb 服务
7. `adb kill-server`  // 停止 adb 服务
8. `adb push local remote` // 电脑推送到手机
9. `adb pull remote local`  // 手机拉取到电脑
10. `adb shell getprop ro.boot.serialno`  // 获取设备 ID 号
11. `adb shell getprop ro.build.version.release` // 获取 Android 版本号
12. `adb shell getprop ro.build.version.sdk`  // 获取 SDK 版本号
13. `adb shell dumpsys display | grep PhysicalDisplayInfo`  // 获取分辨率数组
14. `adb shell dumpsys battery`  // 获取电池相关信息
15. `adb shell dumpsys input | grep FocusedApplication`  // 获取当前界面的 package 和 application
16. `adb shell am start -W component | grep TotalTime`  // 获取 component 启动的时间
17. `adb shell settings get secure default_input_method`  // 获取默认输入法  http://www.cnblogs.com/yajing-zh/p/5125317.html
18. `adb shell settings put secure default_input_method com.sohu.inputmethod.sogou/.SogouIME`  // 设置默认的输入法
 

## 2. am 与 pm

1. `am start -n {packageName}/.{activityName}`  // 启动 app
2. `am kill <packageName>`   // 杀死指定包的进程
3. `am force-stop <packageName>`  // 强制停止
4. `am startservice`   // 启动服务
5. `am stopservice`    // 停止服务
6. `am start -a android.intent.action.VIEW -d http://www.12306.com`  // 打开12306 网站
7. `am start -a android.intent.action.CALL -d tel:10086`  // 拨打 10086
8. `pm list packages`  // 列出手机上所有的包名
9. `pm install/uninstall <packagename> `  // 安装/卸载

## 3. 模拟用户事件

1. **文本输入**： `adb shell input text <String>`  例手机端输出`demo`字符串，相应指令：`adb shell input "demo"`.
2. **键盘事件**： `input keyevent <KEYCODE>`  例点击返回键，相应指令： `input keyevent 4`.
3. **点击事件**： `input tap <x> <y>` 例点击坐标（500，500），相应指令： `input tap 500 500`
4. **滑动事件**： `input swipe <x1> <y1> <x2> <y2> <time>` 例从坐标(300，500)滑动到(100，500)，相应指令： `input swipe 300 500 100 500`.


## 4. logcat

1. `logcat | grep <str>`  // 显示包含 str 的logcat
2. `logcat | grep -i <str>`  // 显示包含 str 的logcat，并忽略大小写
3. `logcat -s "ActivityManager"`  // 显示该标签的log
4. `logcat -d`  // 读完所有log后返回，而不会一直等待
5. `logcat -c`  // 清空log并退出
6. `logcat -t <count>`    // 打印最近 count 行的log
7. `logcat -v <format>` ，格式化输出 log，其中 `format` 有如下可选值：
    * `brief` — 显示优先级/标记和原始进程的PID (默认格式)
    * `process` — 仅显示进程PID
    * `tag` — 仅显示优先级/标记
    * `thread` — 仅显示进程：线程和优先级/标记
    * `raw` — 显示原始的日志信息，没有其他的元数据字段
    * `time` — 显示日期，调用时间，优先级/标记，PID
    * `long` —显示所有的元数据字段并且用空行分隔消息内容

