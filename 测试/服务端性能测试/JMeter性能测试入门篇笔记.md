# JMeter性能测试入门篇笔记

`JMeter`

---


## 1. JMeter 整体介绍

1. 使用 `Jmeter` 进行 `B/S` 架构的测试
2. 官网： `Https://jemter.apache.org`
3. `Jmeter` 的组成主要包括三个部分：
    * **取样器** ： 进行脚本逻辑的控制
    * **线程组** ： 场景设置
    * **监视器** ： 监控脚本的运行，取得性能指标

 
### 1.1 Jmeter 界面介绍

1. 线程数： 相当于要模拟的用户数
2. Ramp-Up Period(in seconds): 加压的策略
3. 循环次数： 脚本运行的次数

**小坑**：在使用 `Jmeter` 多线程并发进行性能测试时，尽量不要使用断言，因为实际使用中发现多线程共享数据时会出现断言不准确的问题。

## 2. JMeter 脚本录制方式及 badboby 简介

### 2.1 JMeter 脚本的两种录制方式

1.  使用 `badbody` 进行录制
    
    `badboby` 是一款工具，它可以进行浏览器操作的录制，然后导出为 `JMeter`脚本
    
2. 使用代理方式进行录制

### 2.2 脚本录制的流程与思路

1. 业务流程
2. 录制工具
3. 脚本制作
4. 性能测试

### 2.3 Badbody 介绍也演示

这款工具主要包括四个部分：

1. 工具区
2. 地址栏
3. 脚本区
4. 视图区


## 3. JMeter 代理录制及运行

### 3.1 代理录制的步骤

1. HTTP 请求默认值
2. HTTP 代理服务器
3. 浏览器设置

**一定要使用好 `包含模式` 和 `排除模式`**


## 4. JMeter非图形界面运行

### 4.1 命令行脚本

* `../jmeter -n -t default_online.jmx -l result_1557.jtl`

* 指定远程压力机进行压力测试：
`../jmeter -n -t default_online.jmx -R 10.173.210.141 -l result_1557.jtl`

* 一边运行一边分析日志
`../jmeter -n -t baidu_requests_results.jmx -r -l baidu_requests_results.jtl -e -o /home/tester/apache-jmeter-3.0/resultReport`

* 命令行参数：
 
    * -n : 非GUI 模式执行JMeter
    * -t : 执行测试文件所在的位置及文件名
    * -r : 远程将所有agent启动用在分布式测试场景下，不是分布式测试只是单点就不需要-r
    * -l : 指定生成测试结果的保存文件， jtl 文件格式
    * -e : 测试结束后，生成测试报告
    * -o : 指定测试报告的存放位置
    
        -o 指定的文件及文件夹，必须不存在 ，否则执行会失败，对应上面的命令就是resultReport文件夹必须不存在否则报错
 
### 4.2 jmeter压力机服务启动

1. `nohup ./jmeter-server &`  后台启动 jmeter server 服务
2. `ps -ef | grep jmeter` 查看 jmeter 服务


`jmeter -n -t weather.jmx -l test.jtl -JforcePerfmonFile=true`

`java -jar D:\program\apache-jmeter-3.1\lib\ext\CMDRunner.jar --tool Reporter --input-jtl test.jtl --plugin-type PerfMon --generate-png report-cpu.png`

### 4.3 返回值中文乱码

修改 `jmeter\bin\jmeter.properties` 的 `sampleresult.default.encoding` 属性，默认值为 `ISO-8859-1`，修改为 `utf-8` 即可 