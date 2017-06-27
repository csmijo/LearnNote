# Jmeter监控服务端性能

`Jmeter`

---

本文使用的 `Jmeter` 版本为 `apache-jmeter-3.2`。

## 1. 前提条件--必须的插件

`jmeter` 也可以像 `loadrunner` 一样监控服务器CPU、内存等性能参数，不过需要安装一些插件：

1. `jpgc-perfmon-2.1.zip`(本机使用): https://jmeter-plugins.org/wiki/PerfMon/#Concept
2. `ServerAgent-2.2.1.zip`(服务器使用): https://jmeter-plugins.org/wiki/PerfMonAgent/

## 2. GUI 监控服务器性能

### 2.1 使用步骤

1. 首先需要在 **服务器** 解压 `ServerAgent-2.2.1.zip`, 然后通过 `startAgent.sh` 或者 `startAgent.bat` 启动 `PerfMon Server Agent`
2. 解压 `jpgc-perfmon-2.1.zip` 文件，然后将对应文件夹下的 `jar`包放到 `jemter`安装目录对应的文件夹下。比如将 `jpgc-perfmon-2.1\lib` 下的所有 `jar` 放到`jemter`安装目录的 `lib`目录下。


这样基本的环境就配置好了，启动 `jmeter`，可以发现`jmeter GUI` 界面的 `选项` 菜单中增加了 `Plugins Manager` 菜单。而且线程组的监听器中增加了 `jp@gc - PerfMon Metrics Collector` 的选项。这样就可以为线程组添加性能监听的服务。

使用 `GUI` 界面进行性能监控，可以通过界面添加需要监控的服务器的 `Ip`，`端口`，`Metric` 等。

### 3. NO GUI 监控服务器性能

使用 `GUI` 界面会消耗额外的性能，所以最好还是使用 `NO GUI` 界面来进行性能监控。

### 3.1 使用步骤

1. 使用 `GUI` 界面编写好 `xxx.jmx` 性能测试脚本，当然在脚本中要指定需要监控的服务器的 `IP`，`端口`，`Metric`等。
    
    **注意**：这里需要在脚本中指定性能测试结果的保存文件，即 `cpu`，`memory`等测试结果的保存文件，否则会全部存入下面指定的 `log.jtl` 文件中。 如下图所示：
    ![](http://img.blog.csdn.net/20140730151453344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2xvdWRfbGw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2. 使用如下命令执行测试脚本：

    ```
    jmeter -n -t xxx.jmx -l log.jtl -JforcePerfmonFile=true
    ```
3. 执行结束之后，结果会保存在 `log.jtl` 文件中，性能测试结果保存在 `result.jtl` 中， 在 `GUI` 界面中导入 `result.jtl` 中即可查看到测试的结果，如下图所示：
![2017-06-26_115857.PNG](http://on9czqsf5.bkt.clouddn.com/2017-06-26_115857.PNG?2017-06-26-11-59-47)

---
[参考文献]
* [PerfMon Server Agent](https://jmeter-plugins.org/wiki/PerfMonAgent/#PerfMon-Server-Agent)
* [Servers Performance Monitoring](https://jmeter-plugins.org/wiki/PerfMon/#Concept)