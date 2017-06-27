# 安装 apk 到 /system/app 目录下

`adb` `system`

---

在 Android 中，如果要使用系统限制的权限（比如 `android.permission.WRITE_SECURE_SETTINGS`），我们需要把程序安装到 `/system/app/` 下。

下面以 `test.apk` 为例，演示这个操作。**需要准备一台已经获得 `Root` 权限的手机。**

1. 通过 USB 连接手机和电脑。
2. 使用 adb 控制手机。
3. 执行如下操作：

    ```
    $ adb push test.apk /sdcard/  // 上传要安装的文件，为安装做准备。
    $ adb shell
    $ su // 切换到 root 用户。如果没有获得 Root 权限，这一步不会成功。
    # mount -o remount,rw -t yaffs2 /dev/block/mtdblock3 /system // 让分区可写。
    # cat /sdcard/test.apk > /system/app/test.apk // 这一步可以用 cp 实现，但一般设备中没有包含该命令。如果使用 mv 会出现错误：failed on '/sdcard/NetWork.apk' - Cross-device link。 
    # mount -o remount,ro -t yaffs2 /dev/block/mtdblock3 /system // 还原分区属性，只读。
    # cd /system/app
    # chmod 777 test.apk  // 给test.apk最高权限
    # exit
    $ exit
    ```
4. 在 `/system/app` 目录下找到 `test.apk` 点击执行安装

---
[参考文献]

* [安装 apk 到 /system/app 目录下](https://my.oschina.net/zengsai/blog/11195)