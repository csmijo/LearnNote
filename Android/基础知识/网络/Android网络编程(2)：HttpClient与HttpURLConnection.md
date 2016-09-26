#Android网络编程(2):HttpClient与HttpURLConnection

`Android` `HttpClient` `HttpURLConnection`

--

##1. HttpClient

　　Android SDK中包含了HttpClient，在**Android6.0版本直接删除了HttpClient类库**，如果仍想使用则解决方法是：

* 如果使用的是eclipse则在libs中加入**org.apache.http.legacy.jar**,
这个jar包在：**sdk\platforms\android-23\optional目录中（需要下载android
6.0的SDK）
* 如果使用的是android studio则 在相应的module下的build.gradle中加入：

    ```java
    android {
           useLibrary 'org.apache.http.legacy'
            }
    ```
    
##2. HttpURLConnection

　　**Android 2.2版本之前，HttpURLConnection一直存在着一些令人厌烦的bug**。比如说对一个可读的InputStream调用close()方法时，就有可能会导致连接池失效了。那么我们通常的解决办法就是直接禁用掉连接池的功能：
```java
private void disableConnectionReuseIfNecessary() {
      // 这是一个2.2版本之前的bug
      if (Integer.parseInt(Build.VERSION.SDK) < Build.VERSION_CODES.FROYO) {
            System.setProperty("http.keepAlive", "false");
      }
}
```

　　所以**在Android 2.2版本以及之前的版本使用HttpClient是较好的选择，而在Android 2.3版本及以后，HttpURLConnection则是最佳的选择**，它的API简单，体积较小，因而非常适用于Android项目。压缩和缓存机制可以有效地减少网络访问的流量，在提升速度和省电方面也起到了较大的作用。另外**在Android 6.0版本中，HttpClient库被移除了，HttpURLConnection则是以后我们唯一的选择**。