# Jmeter之Bean shell的使用

`Jmeter` `Bean shell`

--

最近在学习使用 Jmeter 来进行接口测试，使用 Jmeter 提供的基础方法无法完成测试需求，所以需要编写一些 Bean shell 脚本。下面将对 Bean shell的一些使用方法进行简单的介绍。

## 1. 什么是 Bean shell

`Bean shell`官网：http://www.BeanShell.org/

* `BeanShell` 是一种完全符合Java语法规范的脚本语言,并且又拥有自己的一些语法和方法;
* `BeanShell` 是一种松散类型的脚本语言(这点和JS类似);
* `BeanShell` **是用Java写成的**,一个小型的、免费的、可以下载的、嵌入式的Java源代码解释器,具有对象脚本语言特性,非常精简的解释器jar文件大小为175k。
* `BeanShell` **执行标准Java语句和表达式**,另外包括一些脚本命令和语法。

## 2. Jmeter 有哪些 Bean shell

* 定时器：`BeanShell Timer`
* 前置处理器：`BeanShell PreProcessor`
* 采样器：`BeanShell Sampler`
* 后置处理器：`BeanShell PostProcessor`
* 断言：`BeanShell断言`
* 监听器：`BeanShell Listener`

## 3. Bean shell常用內置变量

JMeter在它的BeanShell中内置了变量，用户可以通过这些变量与JMeter进行交互，其中主要的变量及其使用方法如下:

* `log`：写入信息到jmeber.log文件，使用方法：`log.info(“This is log info!”);`
* `ctx`：该变量引用了当前线程的上下文
* `vars - (JMeterVariables)`：操作 `jmeter` 变量，这个变量实际引用了JMeter线程中的局部变量容器（本质上是Map），它是测试用例与BeanShell交互的桥梁，常用方法：
    * a) vars.get(String key)：从jmeter中获得变量值
    * b) vars.put(String key，String value)：数据存到jmeter变量中
* `props - (JMeterProperties - class java.util.Properties)`：操作jmeter属性，该变量引用了JMeter的配置信息，可以获取Jmeter的属性，它的使用方法与vars类似，但是**只能put进去String类型的值，而不能是一个对象**。对应于java.util.Properties。 
　　　　
    * a) props.get("START.HMS");　　注：START.HMS为属性名，在文件jmeter.properties中定义 
    * b) props.put("PROP1","1234"); 
* `prev - (SampleResult)`：获取前面的sample返回的信息，常用方法：
    * a) getResponseDataAsString()：获取响应信息
    * b) getResponseCode() ：获取响应code
             
## 3. Bean shell 的用法

### 3.1 测试需求

假设现在有这样的测试需求：

对 http 接口xx 进行压力测试，需要根据时间戳随机 uid 模拟用户访问，然后解析接口返回的 Json 串从中得到 vid 值并作为下一步使用，判断该次请求返回的列表组长度及响应码是否符合要求。

### 3.2 测试流程

1. 创建线程组
2. 创建 http sampler
3. 创建 BeanShell PreProcessor 生成随机 uid
    可以创建 BeanShell PreProcessor，然后编写随机生成方法，这里可以自由的指定自己需要的规则。比如下图中使用"test_ + 当前时间 + 随机数" 的规则生成伪uid。
    
    可以使用 `log.info()` 方便进行调试，可以在下方的 log 中进行查看。
    
    可以使用 `vars.put()` 方法将变量存放到 jmeter 的变量中，然后就可以在 http 请求中通过 `${uid}` 的方式进行使用了。
    

    ![2017-06-19_202106.PNG](http://on9czqsf5.bkt.clouddn.com/2017-06-19_202106.PNG?2017-06-19-20-29-08)

4. 创建 BeanShell PostProcessor 
    解析返回的 json串，这里使用第三方的 fastjson.jar 来进行解析，只需要将 `fastjson.jar` 放到 jmter 安装目录的 `lib\ext` 文件夹下即可使用。
    
    ![2017-06-19_202520.PNG](http://on9czqsf5.bkt.clouddn.com/2017-06-19_202520.PNG?2017-06-19-20-29-17)

5. 创建 Beanshell断言
    ![2017-06-19_202722.PNG](http://on9czqsf5.bkt.clouddn.com/2017-06-19_202722.PNG?2017-06-19-20-29-30)




--

[参考文献]
* [Jmeter之Bean shell使用(一)](http://www.cnblogs.com/puresoul/p/4915350.html)
* [Jmeter之Bean shell使用(二)](http://www.cnblogs.com/puresoul/p/4949889.html)
* [[测试]Jmeter-BeanShell的使用介绍](http://www.jianshu.com/p/bc537d6acb3a)
* [ Jmeter BeanShell PostProcessor提取json数据](http://blog.csdn.net/silencemylove/article/details/51373873)