#Retrofit笔记

`Android` `Retrofit`

--

##1. Retrofit注解详解

　　Retrofit共有22个注解，可以分为三类进行解释。

### 1. 第一类： HTTP请求方法

![HTTP请求方法注解](http://upload-images.jianshu.io/upload_images/1724103-db95c51539b62c96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "HTTP请求方法注解")

　　上述表格总的**HTTP** 注解可以代替其他的任意一个注解，它有三个属性：`method`,`path`,`hasBody`。
* method 表示请求的方法，不区分大小写
* path 表示路径
* hasBody 表示是否有请求体

**示例：**

```java
public interface BlogService{

    @HTTP(method = "get", path = "blog/{id}", hasBody = false)
    Call<ResponseBody> getFirstBlog(@Path("id") int id);
}
```


### 2. 第二类： 标记类
![标记类](http://upload-images.jianshu.io/upload_images/1724103-4d09b5595bfb3291.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3. 第三类：参数类
![参数类](http://upload-images.jianshu.io/upload_images/1724103-073abf80aacf492e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **注一：** {占位符}和`Path`尽量只用在URL的path部分，url中的参数使用`Query`和`QueryMap`代替，保证接口定义的简洁
* **注二：** `Query`、`Field`和`Part`这三者都支持`数组`和实现了`Iterable`接口的类型，如List，set等，方便向后台传递数组。


##2. Gson与Converter

　　**在默认情况下Retrofit只支持将HTTP的响应体转换为ResponseBody**，但是返回值Call支持泛型，那就说明泛型参数可以是其他类型的。**而`Converter`就是Retrofit为用户提供的将`ResponseBody`转换成需要的类型的**。

当然只是改变泛型的参数是不行的，我们需要在创建Retrofit时**明确告知**用于将`ResponseBody`转换成我们泛型中的类型时需要使用的`Converter`。

例如，引入对Gson的支持：
```
compile 'com.squareup.retrofit2:converter-gson:2.0.2'
```

通过`GsonConverterFactory`为Retrofit添加Gson支持：
```
Gson gson = new GsonBuilder()
        .setDateFormat("yyyy-MM-dd hh:mm:ss")
        .create();
        
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("http://gank.io/api/")
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build();
```

##3. 其他

* **`Retrofit`**: 网络请求封装库
* **`okHttp`**: 网络请求client
* **`RxJava`**: 进行异步处理，它可以把网络请求放到子线程中执行，然后把请求结果放到主线程中去处理
* `Retrofit`和`okHttp`的关系是: `Retrofit`主要负责请求的数据和请求的结果，使用接口的形式来呈现，`okHttp`主要负责网络请求的过程，打印log，网络不稳定的状态下进行重连接，设置超时等。换句话说，`Retrofit`负责处理**头和尾**，**okHttp**负责中间部分。



--

**参考文献**
* [你真的会用Retrofit2吗?Retrofit2完全教程](http://www.jianshu.com/p/308f3c54abdd)