#Android网络编程(3):Volley

`Android` `Volley`

--

##1. Volley简介

　　Volley既可以访问网络取得数据，也可以加载图片，并且在性能方面也进行了大幅度的调整，**它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作**，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。

##2. Volley网络请求队列

　　Volley请求网络都是基于请求队列的，开发者只要把请求放在请求队列中就可以了，请求队列会依次进行请求，一般情况下，一个应用程序如果网络请求没有特别频繁则完全可以只有一个请求队列（对应Application），如果非常多或其他情况，则可以是一个Activity对应一个网络请求队列，这就要看具体情况了，首先创建队列：
```java
RequestQueue mQueue = Volley.newRequestQueue(getApplicationContext());
```

##3. StringRequest的用法

```java
        //创建请求队列
        RequestQueue mQueue = Volley.newRequestQueue(getApplicationContext());
        StringRequest mStringRequest = new StringRequest(Request.Method.GET, "http://www.baidu.com",
                new Response.Listener<String>() {
                    @Override
                    public void onResponse(String response) {
                        Log.i("wangshu", response);
                    }
                }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("wangshu", error.getMessage(), error);
            }
        });
        //将请求添加在请求队列中
        mQueue.add(mStringRequest);
```

##4. Volley结构图

　　　　　　　![Volley结构图.png](pic/Volley结构图.png "")

　　从上图可以看到Volley分为三个线程，分别是**主线程**、**缓存调度线程**、和**网络调度线程**。

1. 首先请求会加入缓存队列，如果发现可以找到相应的缓存结果就直接读取缓存并解析，然后回调给主线程；
2. 如果在缓存中没有找到结果，则将这条请求加入到网络队列中，然后发送HTTP请求，解析响应并写入缓存，并回调给主线程。

##5. 使用Volley上传文件

###1. .hprof文件简介

```
HPROF file is a Java Hprof Data. HProf is a tool built into 
JDK for profiling the CPU and heap usage within a JVM.
```

* **MIME 类型：** `application/octet-stream`
* **魔术字节(HEX):**  `-`
* **魔术字符串(ASCII):**  `-`

###2. 提交表单

抓取一条提交表单的请求，详情如下：

<pre>
<code>
Connection: keep-alive
Content-Length: 123
X-Requested-With: ShockwaveFlash/16.0.0.296
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.93 Safari/537.36
Content-Type: multipart/form-data; boundary=Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8
Cookie: bdshare_firstime=1409052493497

--Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1
Content-Disposition: form-data; name=&quot;position&quot;

1425264476444
--Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1
Content-Disposition: form-data; name=&quot;editorIndex&quot;

ue_con_1425264252856
--Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1
Content-Disposition: form-data; name=&quot;cm&quot;

100672
--Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1--
</code>
</pre>

拿出一条上传数据进行分析：
<pre>
<code>1 --Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1
2 Content-Disposition: form-data; name=&quot;cm&quot;
3 
4 100672
5 --Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1--</code>
</pre>

1. **第一行：**

    格式为：“--” + boundary + “\r\n”。
    * 其中“--”是数据开始标志；
    * boundary是为Http实体类定义的数据分割线，boundary可以为任意的字符
    * “\r\n” 是回车换行
    
2. **第二行**

    格式为：Content-Disposition：form-data;name="参数名称"+\r\n
    * Content-Disposition 是上传的内容特性
    * form-data是以表格的形式上传
    
3. **第三行**

    格式为：“\r\n”，回车换行
    
4. **第四行**

    格式为：value + “\r\n”
    * value是参数的值，参数是由key和value组成

5. **第五行：结束行**

    格式为：“--” + boundary + “--” + “\r\n”
    **注意：**所有的数据结束之后，需要有这个结尾标志
    
**如果有多个参数，则重复1，2，3，4，直到最后一个参数加上结束行。**

只需继承 Request 就可以定制一个使用Volley上传表单的类，有几点注意：

1.  **Volley定制Request的时候，需要重写获取实体的方法**

    ```java
    public byte[] getBody() throw AuthFailureError{}
    ```
    
2. **把参数通过二进制的形式传给服务器，当然就不需要重写获取参数的方法**

    ```java
    protected Map<String,String> getParams() throws AuthFailureError{}
    ```
3. **Request 中还有一个关键的地方，需要在 http 头部中声明内容类型为表单数据**

    <pre>
    <code>Content-Type: multipart/form-data; boundary=Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1</code>
    </pre>
    
    所以需要重写下面的方法为：
    ```java
    private String MULTIPART_FROM_DATA = "multipart/form-data";
    private String BOUNDARY = "Ij5ei4KM7KM7ae0KM7cH2ae0Ij5Ef1";
    
    public String getBodyContentType(){
        return MULTIPART_FORM_DATA + ";boundary=" + BOUNDARY;
       }
    ```

###3. 上传文件

抓取一条提交表单的请求，详情如下：

<pre><code>POST <a href="http://chuantu.biz/upload.php">http://chuantu.biz/upload.php</a> HTTP/1.1
Host: chuantu.biz
Connection: keep-alive
Content-Length: 4459
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,<em>/</em>;q=0.8
Origin: <a href="http://chuantu.biz">http://chuantu.biz</a>
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.93 Safari/537.36
Content-Type: multipart/form-data; boundary=WebKitFormBoundaryS4nmHw9nb2Eeusll
Referer: <a href="http://chuantu.biz/">http://chuantu.biz/</a>
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8
Cookie: __cfduid=d9215d649e6e648e0eac7688b406a3d911425089350

--WebKitFormBoundaryS4nmHw9nb2Eeusll
Content-Disposition: form-data; name=&quot;uploadimg&quot;; filename=&quot;spark_bg.png&quot;
Content-Type: image/png

JFIFC
    %# , #&amp;&#39;)*)-0-(0%()(C
    (((((((((((((((((((((((((((((((((((((((((((((((((((&quot;
    }!1AQa&quot;q2#BR$3br
    %&amp;&#39;()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
    w!1AQaq&quot;2B  #3Rbr
    $4%&amp;&#39;()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz?PNG
--WebKitFormBoundaryS4nmHw9nb2Eeusll--</code></pre>


拿出一条进行分析：
<pre>
<code>1 --WebKitFormBoundaryS4nmHw9nb2Eeusll
2 Content-Disposition: form-data; name=&quot;uploadimg&quot;; filename=&quot;spark_bg.png&quot;
3 Content-Type: image/png
4 
5 JFIFC
    %# , #&amp;&#39;)*)-0-(0%()(C
    (((((((((((((((((((((((((((((((((((((((((((((((((((&quot;
    }!1AQa&quot;q2#BR$3br
    %&amp;&#39;()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
    w!1AQaq&quot;2B  #3Rbr
    $4%&amp;&#39;()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz?PNG
6 --WebKitFormBoundaryS4nmHw9nb2Eeusll--</code>
</pre>


1. **第一行**

    格式为：“--” + boundary + “\r\n”
    * 含义同“提交表单”中的解释

2. **第二行**

    格式为：Content-Diposition:form-data;name="参数的名称";filename="上传文件的文件名" + "\r\n"
    * 这里比普通的提交表单多了一个`filename = "上传文件的文件名"`

3. **第三行**

    格式为：Content-Type:文件的mime类型 + "\r\n"
    * **这一行是上传文件必须的，而普通文件提交则可有可无**。 mime类型需要根据文档查询。
    
4. **第四行**

    格式为："\r\n"
    
5. **第五行**

    格式为：文件的二进制数据 + "\r\n"
    
6. **第六行:结束行**

    格式为："--" + boundary + "--" + "\r\n"

**文件也可以同时上传多个文件，上传多个文件时，只需要重复1，2，3，4，5不即可，在最后的一个文件的末尾加上统一的结束行。**

可以看到，文件上传相比普通的表格提交，有两个不同的地方：
1. 第二行增加了一个文件名变量
2. 增加了一行 Content-Type：文件的mime类型 + "\r\n"

--
**[参考]**
* [Android 网络编程（3）：Volley用法全解析](http://mp.weixin.qq.com/s?__biz=MzA3MDMyMjkzNg==&mid=2652261891&idx=1&sn=ff70183bc983577da4a584cde5a7f4ed&scene=1&srcid=0822q8tHD8Coxq1xFGQRxskX#wechat_redirect)
* [Android 网络编程（4）: 从源码解析Volley（上）](http://mp.weixin.qq.com/s?__biz=MzA3MDMyMjkzNg==&mid=2652261893&idx=1&sn=f0ac821aeeea42414caf7d923f65db54&scene=1&srcid=0822hMQo4XYoguBXSG1KHfEq#wechat_redirect)
* [Android 网络编程（4）: 从源码解析Volley（下）](http://mp.weixin.qq.com/s?__biz=MzA3MDMyMjkzNg==&mid=2652261893&idx=2&sn=99c2b1404ad168112a2f481cc2a3c46b&scene=1&srcid=0822MaKKvSr6Zoby59dhJsfl#wechat_redirect)
* [详细的文件扩展名 .hprof](https://www.filedesc.com/zh/file/hprof)
* [Android Volley文件上传（一）](http://blog.csdn.net/hong15007046964/article/details/51153562)
* [Android Volley文件上传（二）](http://blog.csdn.net/hong15007046964/article/details/51153771)
* [Android中自定义MultipartEntity实现文件上传以及使用Volley库实现文件上传](http://blog.csdn.net/bboyfeiyu/article/details/42266869)