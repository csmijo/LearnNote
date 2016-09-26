#Android HTTP请求方式:HttpClient

`Android` `HttpClient`

--

##1. HttpClient介绍

　　HttpClient，尽管**被Google弃用**了，但是我们我们平时也可以拿HttpClient来抓下包，配合Jsoup解析网页效果更佳！HttpClient 用于接收/发送Http请求/响应，但不缓存服务器响应，不执行HTML页面潜入的JS代码，不会对页面内容 进行任何解析，处理！

##2. HttpClient使用流程

1. 创建HttpClient对象：`HttpClient client = new HttpClient()`；
2. 发送Get请求，创建HttpGet对象；发送Post请求，创建HttpPost对象；
3. 设置请求参数，两者都可以使用`setParams(HttpParams)`;Post还可以用`setEntry(HttpEntity)`方法；
4. 调用HttpClient对象的execute方法发送请求，该方法返回一个HttpResponse对象，该对象包装了服务器的相应内容；
5. 对返回的HttpResponse对象调用`getAllHeaders()`，`getHeaders(name)`等方法获得服务器的响应头；调用`getEntry()`方法取得HttpEntity对象。

##3. 使用HttpClient发送Get请求

```java
public void GetUtil(){
    HttpClient httpClient = new DefaultHttpClient();
    HttpGet httpGet = new HttpGet("http://www.w3cschool.cc/python/python-tutorial.html");
    HttpResponse httpResponse = httpClient.execute(httpGet);
    if (httpResponse.getStatusLine().getStatusCode() == 200) {
         HttpEntity entity = httpResponse.getEntity();
         detail = EntityUtils.toString(entity, "utf-8");
    }
}
```

　　另外，如果是带有参数的GET请求的话，我们可以将参数放到一个List集合中，再对参数进行URL编码， 最后和URL拼接下就好了：
```java
List<BasicNameValuePair> params = new LinkedList<BasicNameValuePair>();  
params.add(new BasicNameValuePair("user", "猪小弟"));  
params.add(new BasicNameValuePair("pawd", "123"));
String param = URLEncodedUtils.format(params, "UTF-8"); 
HttpGet httpGet = new HttpGet("http://www.baidu.com"+"?"+param);

```

##4. 使用HttpClient发送Post请求

　　POST请求比GET稍微复杂一点，创建完`HttpPost`对象后，通过`NameValuePair`集合来存储等待提交的参数，并将参数传递到`UrlEncodedFormEntity`中，最后调用`setEntity(entity)`完成， `HttpClient.execute(HttpPost)`即可。

```java
private void PostByHttpClient(final String url)
{
    new Thread()
    {
        public void run() 
        {
            try{
                HttpClient httpClient = new DefaultHttpClient();
                HttpPost httpPost = new HttpPost(url);
                List<NameValuePair> params = new ArrayList<NameValuePair>();
                params.add(new BasicNameValuePair("user", "猪大哥"));
                params.add(new BasicNameValuePair("pawd", "123"));
                UrlEncodedFormEntity entity = new UrlEncodedFormEntity(params,"UTF-8");
                httpPost.setEntity(entity);
                HttpResponse httpResponse =  httpClient.execute(httpPost);
                if (httpResponse.getStatusLine().getStatusCode() == 200) {
                    HttpEntity entity2 = httpResponse.getEntity();
                    detail = EntityUtils.toString(entity2, "utf-8");
                    handler.sendEmptyMessage(SHOW_DATA);
                }
            }catch(Exception e){e.printStackTrace();}
        };
    }.start();
}
```