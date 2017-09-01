# ADB Shell命令学习

`adb shell`

--

【官方网址】：[ADB Shell](http://adbshell.com/commands)

本文记录了一些自己在工作中经常使用到的一些 adb 命令，更多的请查看官网 [ADB Shell](http://adbshell.com/commands)。


## 1. ADB Debbugging

### 1. adb devices

显示当前连接模拟器或者手机设备。

如果当前连接了设备，该命令会返回：**设备的序列码(serial number)**和**状态信息(state)**，如下：

|serial number|state|
|:--|--:|
|e4b25377|device|
|emulator-5554|device|

### 2. adb kill-server

结束 adb 服务进程

`kill the server if it is running`

### 3. adb start-server

`ensure that server if it is running`

### 4. adb usb

`restarts the adbd daemon listening on USB`

## 2. Package Manager

### 1. adb install [option]<path>

安装apk

* `adb install test.apk`
* `adb install -l test.apk` *forward lock application*
* `adb install -r test.apk` *replace existing application*
* `adb install -t test.apk` *allow test packages*
* `adb install -s test.apk` *install application on sdcard*
* `adb install -d test.apk` *allow version code downgrade*
* `adb install -p test.apk` *partial application install*

### 2. adb uninstall [options]<PACKAGE>

卸载apk

* `adb uninstall com.test.app`
* `adb uninstall -k com.test.app` *Keep the data and cache directories around after package removal*

### 3. adb shell pm list packages

命令格式：
`adb shell pm list packages [options] <FILTER>`

* `adb shell pm list packages`
* `adb shell pm list packages -f` *See their associated file.*
* `adb shell pm list packages -d` *Filter to only show disabled packages.*
* `adb shell pm list packages -e` *Filter to only show enabled packages.*
* **`adb shell pm list packages -s`** *Filter to only show system packages.*
* **`adb shell pm list packages -3`** *Filter to only show third party packages.*
* `adb shell pm list packages -i` *See the installer for the packages.*
* `adb shell pm list packages -u` *Also include uninstalled packages.*
* `adb shell pm list packages --user <USER_ID>` *The user space to query.*

### 4. adb shell pm path

打印指定 package 的路径

命令格式为： `adb shell pm path <PACKAGE>`

示例： `adb shell pm path com.android.phone`

### 5. adb shell pm clear

删除与指定 package 相关联的所有数据

命令格式： `adb shell pm clear <PACKAGE>`

示例：`adb shell pm clear com.test.abc`

## 3. FileManager

### 1. adb pull

将手机或者模拟器上的文件下载到电脑（**手机-->电脑**）

命令格式： `adb pull <remote> [local]`

示例： `adb pull /sdcard/demo.mp4 e:\`

### 2. adb push

将电脑中的文件上传到手机（**电脑-->手机**）

命令格式： `adb push <local> <remote>`

示例： `adb push d:\test.apk /sdcard`

## 4. NetWork

### 1. adb shell netstat

网络数据统计

使用步骤：

1. adb shell
2. netstat

## 5. Logcat

### 1. adb logcat

在屏幕上打印log信息

命令格式：`adb logcat [option] [filter-specs]`

* `adb logcat *:V`   lowest priority, filter to only show Verbose level
* `adb logcat *:D`   filter to only show Debug level
* `adb logcat *:I`   filter to only show Info level
* `adb logcat *:W`   filter to only show Warning level
* `adb logcat *:E`   filter to only show Error level
* `adb logcat *:F`   filter to only show Fatal level
* `adb logcat *:S`   Silent, highest priority, on which nothing is ever printed

#### a. adb logcat -b <Buffer>

* `adb logcat -b radio`  View the buffer that contains radio/telephony related messages.
* `adb logcat -b event`  View the buffer containing events-related messages.
* `adb logcat -b main`  default
* `adb logcat -c`  Clears the entire log and exits.
* `adb logcat -d`  Dumps the log to the screen and exits.
* `adb logcat -f test.logs`  Writes log message output to test.logs .
* `adb logcat -g`  Prints the size of the specified log buffer and exits.
* `adb logcat -n <count>`  Sets the maximum number of rotated logs to <count>. **The default value is 4. Requires the -r option.**
* `adb logcat -r <kbytes>`  Rotates the log file every <kbytes> of output. **The default value is 16. Requires the -f option.**
* `adb logcat -s`  Sets the default filter spec to silent.

#### b. adb logcat -v <format>

* `adb logcat -v brief`   Display priority/tag and PID of the process issuing the message (default format).
* `adb logcat -v process`   Display PID only.)
* `adb logcat -v tag`   Display the priority/tag only.
* `adb logcat -v raw`   Display the raw log message, with no other metadata fields.
* `adb logcat -v time`   Display the date, invocation time, priority/tag, and PID of the process issuing the message.
* `adb logcat -v threadtime`   Display the date, invocation time, priority, tag, and the PID and TID of the thread issuing the message.
* `adb logcat -v long`   Display all metadata fields and separate messages with blank lines.

### 2. adb shell dumpsys

* `adb shell dumpsys meminfo`  collects memory info 
* `adb shell dumpsys battery` **A mobile device with Developer Options enabled running Android 5.0 or higher.**
* `adb shell dumpsys batterystats`  collects battery data from your device.  **[Battery Historian](https://github.com/google/battery-historian) converts that data into an HTML visualization. STEP 1 adb shell dumpsys batterystats > batterystats.txt STEP 2 python historian.py batterystats.txt > batterystats.html**
* `adb shell dumpsys batterystats --reset`  erases old collection data
* `adb shell dumpsys activity`  collect activity info
* `adb shell dumpsys gfxinfo com.android.phone`  measuring com.android.phone **ui performance**

## 6. Screenshot

### 1. adb shell screencap

截图

命令格式： `adb shell screencap <filename>`

示例： `adb shell screencap /sdcard/screen.png`

### 2. adb shell screenrecord [4.4+]

**Android 4.4 (API Level 19)或者更高才可以使用该命令**

1. `adb shell screenrecord [options] <filename>`

    示例： `adb shell screenrecord /sdcard/demo.mp4`
    
    (press Ctrl-C to stop recording)
    
    **注意**：Stop the screen recording by pressing **Ctrl-C**, otherwise the recording stops automatically at **three minutes** or **the time limit set by --time-limit**.

2. `adb shell screenrecord --size <WIDTHxHEIGHT>`

    Sets the video size: 1280x720. **The default value is the device's native display resolution (if supported)**, 1280x720 if not. For best results, use a size supported by your device's Advanced Video Coding (AVC) encoder.

3. `adb shell screenrecord --bit-rate <RATE>`

    Sets the video bit rate for the video, in megabits per second. **The default value is 4Mbps**. You can increase the bit rate to improve video quality, but doing so results in larger movie files. The following example sets the recording bit rate to 5Mbps: **adb shell screenrecord --bit-rate 5000000 /sdcard/demo.mp4**

4. `adb shell screenrecord --time-limit <TIME>`

    Sets the maximum recording time, in seconds. T**he default and maximum value is 180 (3 minutes)**.

5. `adb shell screenrecord --rotate`

    Rotates the output 90 degrees. This feature is experimental.

6. `adb shell screenrecord --verbose`

    Displays log information on the command-line screen. If you do not set this option, the utility does not display any information while running.
  
## 7.  System

### 1. adb shell getprop
 
get property via the android property service.

命令格式： `getprop [options]`

使用步骤：

1. `adb shell`
2. `getprop`

    `getprop ro.build.version.sdk`
    
    `getprop ro.chipname`

    `getprop | grep adb`
    
### 2. adb shell setprop

set property service

命令格式： `setprop <key> <value>`