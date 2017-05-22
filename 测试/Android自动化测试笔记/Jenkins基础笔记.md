# Jenkins 基础笔记

---



## 1. Jenkins 

### 1.1 Windows 安装说明

1. 进入 Jenkins 官网(http://jenkins-ci.org/) 下载最新版本
2. 点击安装
3. 安装完成后，打开浏览器，输入 `http://localhost:8080` ，进入 Jenkins 管理界面

### 1.2 Jenkins 基础

主要以一个一个任务(Job) 的执行来管理。

一个任务(Job) 包括以下几个模块：
* 源码管理   ---  SVN/CVS/GIT 等代码管理器
* 构建触发器  --- 什么条件下触发构建，如代码变化，指定时间等
* 构建  --- 执行构建、测试自己想要的步骤
* 构建后操作  --- 构建后的操作，如发送测试报告等

### 1.3 Jenkins 启动和停止服务

1. 如何启动 `Jenkins`
    * step1: 进入 `Jenkins` 的 war 包所在的目录
    * step2: `java -jar jenkins.war` (调用里面的这个war包，如果你的war包名字不是`Jenkins.war`，请用你的 war 包名字，不可生搬硬套)

2. 启动 `Jenkins` 服务
    * `net start jenkins` (注：如果Jenkins曾经启动过，启动服务不需要进入到某个目录)

3. 停止 `Jenkins` 服务
    * `net stop jenkins` 
 
**注：**  Jenkins的关闭和启动都可以通过关闭和启动服务来进行

### 1.4 Jenkins 发送邮件配置

https://my.oschina.net/FrankXin/blog/646084
