# Jmeter测试报告生成

`Jmeter` `report`

---

本文使用的 `Jmeter` 版本为 `apache-jmeter-3.2`。

## 1. 命令行模式将 jtl 文件转成测试图表 

**注意：** 这种方式只适用于 `jmeter3.0` 以后的版本

### 1.1 在测试的过程中将 jtl 转换成测试报告

可以执行如下命令：
```
jmeter -n -t test_request.jmx -l test_result.jtl -e -o /home/csmijo/resultReport
```

参数说明：
  * `-n` : 非GUI 模式执行JMeter
  * `-t` : 执行测试文件所在的位置及文件名
  * `-r` : 远程将所有agent启动用在分布式测试场景下，不是分布式测试只是单点就不需要-r
  * `-l` : 指定生成测试结果的保存文件， jtl 文件格式
  * `-e` : 测试结束后，生成测试报告
  * `-o` : 指定测试报告的存放位置
      * **-o 指定的文件及文件夹，必须不存在**，否则执行会失败，对应上面的命令就是 `resultReport` 文件夹必须不存在否则报错，**如果存在，则文件夹必须为空**

报告文件大致如下图所示：

![2017-06-26_165038.PNG](http://on9czqsf5.bkt.clouddn.com/2017-06-26_165038.PNG?2017-06-26-16-51-13)


### 1.2 使用之前的测试结果，生成测试报告

如果在执行压测脚本的时候没有指定生成测试报告，在测试结束之后，也可以通过如下的命令生成：
```
jmeter -g log.jtl -e -o resultReport
```

参数说明:

* `-g` : 指定已存在的测试结果文件
* `-e` ：测试结果后，生成测试报告
* `-o` : 指定测试报告的存放位置
    * **-o 指定的文件及文件夹，必须不存在** ，否则执行会失败

效果如上图

## 2. 插件模式将 jtl 转成测试图表

## 2.1 图表 plugin 的类型

1. `AggregateReport` = JMeter's native Aggregate Report, can be **saved only as CSV**
2. `SynthesisReport` = mix between JMeter's native Summary Report and Aggregate Report, can be **saved only as CSV**
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


## 2.2 不同 plugin 类型的生成方式

### 2.2.1 主要 plugin 类型

这里使用 `JMeterPluginCMD` 来生成对应的图片或者 csv 统计文件。

1. 从官网下载： `jpgc-cmd-2.1.zip`: https://jmeter-plugins.org/wiki/JMeterPluginsCMD/ 
2. 解压文件，对应的文件分别放入 `jmeter` 安装目录的对应文件夹下，即解压后 `bin` 目录下的文件都放入到 `jmeter` 安装目录的 `bin` 目录下，以此类推。
3. 为了使用 `JMeterPluginCMD` 生成结果图片或 csv 文件，还需要依赖一下的组件：
    1. `jpgc-filterresults-2.1.zip`  https://jmeter-plugins.org/wiki/FilterResultsTool/
    2. `jpgc-synthesis-2.1.zip`  https://jmeter-plugins.org/?search=jpgc-synthesis  
    3. `GUI` 界面中的 [plugins manager](https://jmeter-plugins.org/install/Install/) 中的 `jpgc-Standard set`，其中共包含以下的文件：
        * jpgc-dummy
        * jpgc-fifo
        * jpgc-graphs-basic
        * jpgc-perfmon
        * jpgc-tst
        * jpgc-sense
        * jpgc-functions
        * jpgc-casutg
        * jpgc-ffw        
![jpgc-Standard-Set.PNG](http://on9czqsf5.bkt.clouddn.com/jpgc-Standard-Set.PNG?2017-06-26-18-46-44)

        
   
4. 然后就可以使用 `JMeterPluginsCMD.bat/sh` 生成图片或 CSV 文件了。比如，生成性能测试结果图片或 CSV 文件的命令为：

    ```
    # 生成图片
    JMeterPluginsCMD.bat --generate-png cpu.png --input-jtl cpu.jtl --plugin-type PerfMon
    
    # 生成 CSV 文件
     JMeterPluginsCMD.bat --generate-csv cpu.csv --input-jtl cpu.jtl --plugin-type PerfMon
    ```
    
5. 添加完上述的文件就可以生成如下 `plugin` 类型的图表
    1. AggregateReport
    2. SynthesisReport
    3. ThreadsStateOverTime
    4. PerfMon
    5. ResponseTimesOverTime
    6. TransactionsPerSecond

### 2.2.2 其他类型的 plugin

如果要生成如下 `plugin` 类型的图表：

1. BytesThroughputOverTime
2. HitsPerSecond
3. LatenciesOverTime
4. ResponseCodesPerSecond
5. ResponseTimesDistribution
6. ResponseTimesPercentiles
7. ThroughputVsThreads
8. TimesVsThreads

就还需要添加如下的 `jar` 包到 `jmeter` 安装目录的 `lib\ext`下：

* `JMeterPlugins-Standard.jar`  https://jmeter-plugins.org/downloads/old/
* `JMeterPlugins-Extras.jar`  https://jmeter-plugins.org/downloads/old/

### 2.2.3 生成所有 plugin 类型的命令

添加好上面的依赖文件后，就可以使用如下的脚本批量生成图表了。
```
source /etc/profile
file="./csv/test"
jtlfile="test.jtl"
JMeterPluginsCMD.sh --generate-csv $file"_AggregateReport.csv" --input-jtl  $jtlfile  --plugin-type AggregateReport
JMeterPluginsCMD.sh --generate-csv $file"_SynthesisReport.csv" --input-jtl  $jtlfile  --plugin-type SynthesisReport
JMeterPluginsCMD.sh --generate-csv $file"_ThreadsStateOverTime.csv" --input-jtl  $jtlfile  --plugin-type ThreadsStateOverTime
JMeterPluginsCMD.sh --generate-csv $file"_BytesThroughputOverTime.csv" --input-jtl  $jtlfile  --plugin-type BytesThroughputOverTime
JMeterPluginsCMD.sh --generate-csv $file"_HitsPerSecond.csv" --input-jtl  $jtlfile --plugin-type HitsPerSecond
JMeterPluginsCMD.sh --generate-csv $file"_LatenciesOverTime.csv" --input-jtl  $jtlfile  --plugin-type LatenciesOverTime
JMeterPluginsCMD.sh --generate-csv $file"_ResponseCodesPerSecond.csv" --input-jtl  $jtlfile  --plugin-type ResponseCodesPerSecond
JMeterPluginsCMD.sh --generate-csv $file"_ResponseTimesDistribution.csv" --input-jtl  $jtlfile  --plugin-type ResponseTimesDistribution
JMeterPluginsCMD.sh --generate-csv $file"_ResponseTimesOverTime.csv" --input-jtl  $jtlfile  --plugin-type ResponseTimesOverTime
JMeterPluginsCMD.sh --generate-csv $file"_ResponseTimesPercentiles.csv" --input-jtl  $jtlfile  --plugin-type ResponseTimesPercentiles
JMeterPluginsCMD.sh --generate-csv $file"_ThroughputVsThreads.csv" --input-jtl  $jtlfile  --plugin-type ThroughputVsThreads
JMeterPluginsCMD.sh --generate-csv $file"_TimesVsThreads.csv" --input-jtl  $jtlfile  --plugin-type TimesVsThreads
JMeterPluginsCMD.sh --generate-csv $file"_TransactionsPerSecond.csv" --input-jtl  $jtlfile  --plugin-type TransactionsPerSecond
```


---

[参考文献]

* [JMeterPluginsCMD Command Line Tool](https://jmeter-plugins.org/wiki/JMeterPluginsCMD/)
* [JMeter执行压测输出HTML图形化报表（三）](http://www.cnblogs.com/qmfsun/p/6382540.html)
* [jmeter之jtl文件解析](http://www.cnblogs.com/miaomiaokaixin/p/6118081.html)
* [JMeter Plugins Manager](https://jmeter-plugins.org/wiki/PluginsManager/)