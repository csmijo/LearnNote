#Android HTTP请求方式：HttpURLConnection

`Android` `HttpURLConnection`

--

##1. HttpURLConnection的介绍

　　它是一种多用途、轻量极的HTTP客户端，使用它来进行HTTP操作可以适用于大多数的应用程序。 虽然`HttpURLConnection`的API提供的比较简单，但是同时这也使得我们可以更加容易地去使用和扩展它。**继承至URLConnection，抽象类，无法直接实例化对象。通过调用`openCollection()` 方法获得对象实例，默认是带gzip压缩的**。


##2. HttpURLConnection使用示例

###1）工具类

　　我们在HttpURLConnection的Get和Post方式时，使用conn.getInputStream()获得的是一个流，所以我们需要一个类将**流**转化成**二进制数据**。

```java
public class StreamTool {
    // 从流读取数据
    public static byte[] read(InputStream inStream) throws Exception{
        ByteArrayOutputStream outStream = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len = 0;
        while((len = inStream.read(buffer)) != -1)
        {
            outStream.write(buffer,0,len);
        }
        inStream.close();
        return outStream.toByteArray();
    }
}
```

另外，还可以使用read的方式，将**流**直接转成**字符串**。
```java
public static String read2(InputStream inStream) throws Exception{
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        StringBuilder response = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null){
            response.append(line);
        }
        /*资源释放*/
        in.close();
        // 返回字符串
        return response.toString();
}
```

###2) HttpURLConnection发送Get请求的示例

```java
public class GetData {
    // 1. 获取网络图片数据:
    public static byte[] getImage(String path) throws Exception {
        URL url = new URL(path);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        // 设置连接超时为5秒
        conn.setConnectTimeout(5000);
        // 设置请求类型为Get类型
        conn.setRequestMethod("GET");
        // 判断请求Url是否成功
        if (conn.getResponseCode() != 200) {
            throw new RuntimeException("请求url失败");
        }
        InputStream inStream = conn.getInputStream();
        byte[] bt = StreamTool.read(inStream);
        inStream.close();
        return bt;
    }

    // 2. 获取网页的html源代码
    public static String getHtml(String path) throws Exception {
        URL url = new URL(path);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setConnectTimeout(5000);
        conn.setRequestMethod("GET");
        if (conn.getResponseCode() == 200) {
            InputStream in = conn.getInputStream();
            byte[] data = StreamTool.read(in);
            in.close();
            String html = new String(data, "UTF-8");  //对二进制数据进行UTF-8编码
            return html;
        }
        return null;
    }
}
```

**注意：**不要忘记加上联网权限

`<uses-permission android:name="android.permission.INTERNET" />`


###3) HttpURLConnection发送Post请求的示例

```java
public class PostUtils {
    public static String LOGIN_URL = "http://172.16.2.54:8080/HttpTest/ServletForPost";
    public static String LoginByPost(String number,String passwd)
    {
        String msg = "";
        try{
            HttpURLConnection conn = (HttpURLConnection) new URL(LOGIN_URL).openConnection();
            /* 设置请求方式,请求超时信息 */
            conn.setRequestMethod("POST");
            conn.setReadTimeout(5000);
            conn.setConnectTimeout(5000);
            //设置运行输入,输出:
            conn.setDoOutput(true);
            conn.setDoInput(true);
            //Post方式不能缓存,需手动设置为false
            conn.setUseCaches(false);
            //我们请求的数据:
            String data = "passwd="+ URLEncoder.encode(passwd, "UTF-8")+
                    "&number="+ URLEncoder.encode(number, "UTF-8");
            //这里可以写一些请求头的东东...
            //获取输出流
            OutputStream out = conn.getOutputStream();
            out.write(data.getBytes());
            out.flush();
            
             if (conn.getResponseCode() == 200) {  
                    // 获取响应的输入流对象  
                    InputStream is = conn.getInputStream();  
                    byte[] data = StreamTool.read(in);
                    // 返回字符串  
                    msg = new String(data,"UTF-8");  
                    return msg;
             }
        }catch(Exception e){e.printStackTrace();}
        return msg;
    }
}
```

--
**参考文献：**
* [Android HTTP请求方式:HttpURLConnection](http://www.runoob.com/w3cnote/android-tutorial-httpurlconnection.html "")