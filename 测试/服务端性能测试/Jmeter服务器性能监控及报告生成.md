Jmeter 服务器性能监控及报告生成

`Jmeter`

---


`Jmeter版本`：**apache-jmeter-3.2**

## 1. 前提条件--必须的插件

jmeter也可以像loadrunner一样监控服务器CPU、内存等性能参数，不过需要安装一些插件：

1. `jpgc-perfmon-2.1.zip`(本机使用): https://jmeter-plugins.org/wiki/PerfMon/#Concept
2. `ServerAgent-2.2.1`(服务器使用): https://jmeter-plugins.org/wiki/PerfMonAgent/


## 2. GUI 监控服务器性能

### 2.1 使用步骤

1. 首先需要在 **服务器** 通过 `startAgent.sh` 或者 `startAgent.bat` 启动 `PerfMon Server Agent`
2. 解压 `jpgc-perfmon-2.1.zip` 文件，然后将 `lib` 文件夹的：
    * `jmeter-plugins-cmn-jmeter-0.3.jar` ---> 放入 `jmeter` 安装目录的 `lib` 目录
    * `perfmon-2.2.2.jar` ---> 放入 `jmeter` 安装目录的 `lib` 目录
    * `ext\jmeter-plugins-manager-0.11.jar` ---> 放入 `jmeter` 安装目录的 `lib\ext` 目录
    * `ext\jmeter-plugins-perfmon-2.1.jar` ---> 放入 `jmeter` 安装目录的 `lib\ext` 目录
 

这样基本的环境就配置好了，启动 `jmeter`，可以发现`jmeter GUI` 界面的 `选项` 菜单中增加了 `Plugins Manager` 菜单。而且线程组的监听器中增加了 `jp@gc - PerfMon Metrics Collector` 的选项。这样就可以为线程组添加性能监听的服务。

使用GUI界面进行性能监控，可以通过界面添加需要监控的服务器的 `Ip`，`端口`，`Metric` 等。

### 3. NO GUI 监控服务器性能

使用 GUI 界面会消耗额外的性能，所以最好还是使用 NO GUI 界面来进行性能监控。

### 3.1 使用步骤

1. 使用 GUI 界面编写好 xxx.jmx 性能测试脚本，当然在脚本中要指定需要监控的服务器的 IP，端口，Metric。
    
    **注意**：这里需要在脚本中指定性能测试结果的保存文件，即 cpu，memory等测试结果的保存文件，否则会全部存入下面的 log.jtl 文件中。 如下图所示：
    ![](http://img.blog.csdn.net/20140730151453344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2xvdWRfbGw=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2. 使用如下命令执行测试脚本：

    ```
    jmeter -n -t xxx.jmx -l log.jtl -JforcePerfmonFile=true
    ```
3. 执行结束之后，结果会保存在 `log.jtl` 文件中，可以通过如下命令生成测试报告：

    ```
    jmeter -g log.jtl -e -o resultReport
    ```
    注意： 这里必须确保 resultReport 文件夹存在，不然会执行错误
    
### 3.2 插件模式将jtl转成测试图表

#### 3.2.1 GUI 方式

打开 GUI 窗口，导入 log.jtl 即可生成性能结果，但是速度比较慢。

### 3.2.2 插件方式

这里使用 JMeterPluginCMD 来生成对应的图片或者 csv 统计文件。

1. 从官网下载： `jpgc-cmd-2.1.zip`: https://jmeter-plugins.org/wiki/JMeterPluginsCMD/ 
2. 解压文件，对应的文件分别放入 jmeter 安装目录的对应文件夹下，：
    * `bin\JMeterPluginsCMD.bat` --> `jmeter\bin\JMeterPluginsCMD.bat`
    * `bin\JMeterPluginsCMD.sh`--> `jmeter\bin\JMeterPluginsCMD.bat`
    * `lib\cmdrunner-2.0.jar` --> `jmeter\lib\cmdrunner-2.0.jar`
    * `lib\jmeter-plugins-cmn-jmeter-0.3.jar` --> `jmeter\lib\jmeter-plugins-cmn-jmeter-0.3.jar`
    * `lib\ext\jmeter-plugins-cmd-2.1.jar` --> `jmeter\lib\ext\jmeter-plugins-cmd-2.1.jar`
    * `lib\ext\jmeter-plugins-manager-0.11.jar` --> `jmeter\lib\ext\jmeter-plugins-manager-0.11.jar`

* 然后就可以使用 JMeterPluginsCMD.bat/sh 生成图片或 CSV 文件了。比如，生成性能图片和文件的命令为：

    ```
    # 生成图片
    JMeterPluginsCMD.bat --generate-png cpu.png --input-jtl cpu.jtl --plugin-type PerfMon
    
    # 生成 CSV 文件
     JMeterPluginsCMD.bat --generate-csv cpu.csv --input-jtl cpu.jtl --plugin-type PerfMon
    ```

### 3.2.3 其他的 Plugin Type

#### 1. plugin-type 类型

Most of class names are self-explanatory:

* `AggregateReport` = JMeter's native Aggregate Report, can be **saved only as CSV**
* `SynthesisReport` = mix between JMeter's native Summary Report and Aggregate Report, can be **saved only as CSV**
* `ThreadsStateOverTime` = Active Threads Over Time
* `BytesThroughputOverTime`
* `HitsPerSecond`
* `LatenciesOverTime`
* `PerfMon` = PerfMon Metrics Collector
* `DbMon` = DbMon Metrics Collector, DataBase, get performance counters via sql
* `JMXMon` = JMXMon Metrics Collector, Java Management Extensions counters
* `ResponseCodesPerSecond`
* `ResponseTimesDistribution`
* `ResponseTimesOverTime`
* `ResponseTimesPercentiles`
* `ThroughputVsThreads`
* `TimesVsThreads` = Response Times VS Threads
* `TransactionsPerSecond`
* `PageDataExtractorOverTime`
* `MergeResults` = MergeResults Command Line Merge Tool to simplify the comparison of two or more load tests, need properties file (like merge-results.properties)

## tmp

standard set
* AggregateReport
* SynthesisReport
* ThreadsStateOverTime
* PerfMon
* ResponseTimesOverTime
* TransactionsPerSecond

----

JMeterPlugins-Standard.jar，JMeterPlugins-Extras.jar

* BytesThroughputOverTime
* HitsPerSecond
* LatenciesOverTime
* ResponseCodesPerSecond
* ResponseTimesDistribution
* ResponseTimesPercentiles
* ThroughputVsThreads
* TimesVsThreads
 


#### 2. 依赖的其他组件

为了使用 JMeterPluginCMD 生成结果图片或 csv 文件，还需要依赖一下的组件：

1. `jpgc-filterresults-2.1.zip`  https://jmeter-plugins.org/wiki/FilterResultsTool/
2. `jpgc-synthesis-2.1.zip`  https://jmeter-plugins.org/?search=jpgc-synthesis  
3. `GUI界面中的 plugins manager 中的 jpgc-Standard set`，其中共包含以下的文件：
    * jpgc-dummy
    * jpgc-fifo
    * jpgc-graphs-basic
    * jpgc-perfmon
    * jpgc-tst
    * jpgc-sense
    * jpgc-functions
    * jpgc-casutg
    * jpgc-ffw

## 4. 性能监控依赖组件小结

### 4.1 ServerAgent-2.2.1.zip

`ServerAgent-2.2.1`(服务器使用): https://jmeter-plugins.org/wiki/PerfMonAgent/

### 4.2 jpgc-perfmon-2.1.zip

`jpgc-perfmon-2.1.zip`(本机使用): https://jmeter-plugins.org/wiki/PerfMon/#Concept

### 4.3 jpgc-cmd-2.1.zip

`jpgc-cmd-2.1.zip`: https://jmeter-plugins.org/wiki/JMeterPluginsCMD/ 

### 4.4 其他

1. `jpgc-filterresults-2.1.zip`  https://jmeter-plugins.org/wiki/FilterResultsTool/
2. `jpgc-synthesis-2.1.zip`  https://jmeter-plugins.org/?search=jpgc-synthesis  
3. `GUI界面中的 plugins manager 中的 jpgc-Standard set`，其中共包含以下的文件：
    * jpgc-dummy
    * jpgc-fifo
    * jpgc-graphs-basic
    * jpgc-perfmon
    * jpgc-tst
    * jpgc-sense
    * jpgc-functions
    * jpgc-casutg
    * jpgc-ffw


### 4.5 jar包列表

#### 4.5.1 lib

* `jmeter-plugins-cmn-jmeter-0.3.jar`
* `perfmon-2.2.2.jar` ------ jpgc-perfmon-2.1.zip
* `cmdrunner-2.0.jar` ------ jpgc-cmd-2.1.zip


#### 4.5.2 lib\ext

* `jmeter-plugins-manager-0.13.jar`
* `jmeter-plugins-perfmon-2.1.jar` ------ jpgc-perfmon-2.1.zip
* `jmeter-plugins-cmd-2.1.jar` ------ jpgc-cmd-2.1.zip
* `jmeter-plugins-filterresults-2.1.jar` ------ jpgc-filterresults-2.1.zip
* `jmeter-plugins-synthesis-2.1.jar` ------ jpgc-synthesis-2.1
* `jmeter-plugins-dummy-0.2.jar`
* `jmeter-plugins-fifo-0.2.jar`
* `jmeter-plugins-graphs-basic-2.0.jar`
* `jmeter-plugins-tst-2.0.jar`
* `jmeter-plugins-senseuploader-2.4.jar`
* `jmeter-plugins-functions-2.0.jar`
* `jmeter-plugins-casutg-2.1.jar`
* `jmeter-plugins-ffw-2.0.jar`

#### 4.5.3 bin

* `JMeterPluginsCMD.bat` ------ jpgc-cmd-2.1.zip
* `JMeterPluginsCMD.sh` ------ jpgc-cmd-2.1.zip
* `FilterResults.bat` ------ jpgc-filterresults-2.1.zip
* `FilterResults.sh` ------ jpgc-filterresults-2.1.zip

---

参考文献：
* [JMeter执行压测输出HTML图形化报表（三）](http://www.cnblogs.com/qmfsun/p/6382540.html)
* [PerfMon Server Agent](https://jmeter-plugins.org/wiki/PerfMonAgent/#PerfMon-Server-Agent)
* [Servers Performance Monitoring](https://jmeter-plugins.org/wiki/PerfMon/#Concept)
* [JMeterPluginsCMD Command Line Tool](https://jmeter-plugins.org/wiki/JMeterPluginsCMD/)
