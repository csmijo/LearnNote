#BroadcastReceiver初学乍道

`Android` `BroadcastReceiver`

--
##1. 两种广播类型：

![两种广播类型](http://www.runoob.com/wp-content/uploads/2015/08/72916726.jpg "")

##2. 接收系统广播

###1) 两种注册广播的方式

　　系统在某些时候会发送相应的系统广播，比如电量低或者充足，刚启动完，插入耳机，输入法改变等。下面我们就来让我们的APP接收系统广播， 接收之前，还需要为我们的APP注册广播接收器哦！而注册的方法又分为以下两种：**动态**与**静态**！

![动态注册广播](http://www.runoob.com/wp-content/uploads/2015/08/17322218.jpg "")
![静态注册广播](http://www.runoob.com/wp-content/uploads/2015/08/93737904.jpg "")

###2) 动态注册广播实例

监听网络的变化情况。

**MyBRReceiver.java**

```java
public class MyBRReceiver extends BroadcastReceiver{
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"网络状态发生改变~",Toast.LENGTH_SHORT).show();
    }
}
```

**MainActivity.java中动态注册广播：**

```java
public class MainActivity extends AppCompatActivity {

    MyBRReceiver myReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // 核心部分代码
        myReceiver = new MyBRReceiver();
        IntentFilter itFilter = new IntentFilter();
        itFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        registerReceiver(myReceiver, itFilter);
    }

    //别忘了将广播取消掉哦~
    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(myReceiver);
    }
}
```

**缺点：**

　　动态广播需要在**App已经启动**的情况下才可以接收广播。假如我们需要在App还没有启动的情况下还能接收广播，那就需要使用**静态注册广播**的方式了。

###3) 静态注册广播实例

监听开机启动。

**1. 自定义一个BroadcastReceiver，重写onReceive完成事务处理**

```java
public class BootCompleteReceiver extends BroadcastReceiver {
    private final String ACTION_BOOT = "android.intent.action.BOOT_COMPLETED";
    @Override
    public void onReceive(Context context, Intent intent) {
    if (ACTION_BOOT.equals(intent.getAction()))
        Toast.makeText(context, "开机完毕~", Toast.LENGTH_LONG).show();
    }
}
```
**2.在AndroidManifest.xml中对该BroadcastReceiver进行注册，添加开机广播的intent-filter!
对了，别忘了加上android.permission.RECEIVE_BOOT_COMPLETED的权限哦！**

```xml
<receiver android:name=".BootCompleteReceiver">
    <intent-filter>
        <action android:name = "android.intent.cation.BOOT_COMPLETED">
    </intent-filter>
</receiver>

<!-- 权限 -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```
**注意**：

　　Android 4.3以上的版本，是允许将程序安装到SD卡上的，假如你的程序是安装在SD上 的，就会收不到开机广播，

###4) 使用广播的注意事项

　　**不要在广播里添加过多逻辑或者进行任何耗时操作**,因为在**广播中是不允许开辟线程的**, 当`onReceiver()`方法运行较长时间`(超过10秒)`还没有结束的话，那么程序会报错**`(ANR)`**, 广播更多的时候扮演的是一个打开其他组件的角色,比如启动Service,Notification提示, Activity等！

##3. 发送全局广播

###1) 如何发送

**a. 发送广播前需要先定义一个广播接收器。**
* 自定义一个BroadcastReceiver，重写onReceive()方法
* 在AndroidManifest.xml中注册该BroadcastReceiver
* 在AndroidManifest.xml中对Intent-filter通过`android:priority="100"`来设置优先级，值越大则优先级越高，越先接受到广播。优先级可选值的范围：**`-1000~1000`**
* 对于`有序广播`，还可以调用`abortBroadcast()`截断广播的继续传递。

**b. 发送广播**
* **标准广播**：`sendBroadcast(intent) `
* **有序广播**：`sendOrderedBroadcast(intent,null)`

###2) 发送广播

**1. MyBroadcastReceiver.java**
```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    private final String ACTION_BOOT = "com.example.broadcasttest.MY_BROADCAST";
    @Override
    public void onReceive(Context context, Intent intent) {
        if(ACTION_BOOT.equals(intent.getAction()))
        Toast.makeText(context, "收到告白啦~",Toast.LENGTH_SHORT).show();
    }
}
```

**2. 然后AndroidManifest.xml中注册下，写上Intent-filter：**
```xml
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

**3. 在MainActivity.java中发送广播**
```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn_send = (Button) findViewById(R.id.btn_send);
        btn_send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            // 发送标准广播
                sendBroadcast(new Intent("com.example.broadcasttest.MY_BROADCAST"));
            // 发送有序广播
             sendOrderedBroadcast(new Intent("com.example.broadcasttest.MY_BROADCAST"),null);
            }
        });
    }
}
```

**4. 全局广播的缺点：**

　　以上用到的都是**全局广播**，也就是其他应用也能收到我们的广播，这样可能会引起一些**安全性问题**。

##4. 发送本地广播

###1) 核心用法：
使用`LocalBroadcastManager`来管理广播：
* 调用`LocalBroadcastManager.getInstance()`获得实例，例如`localBrManager`
* 调用`localBrManager.registerReceiver()`注册广播
* 调用`localBrManager.sendBroadcast()`发送广播
* 调用`localBrManager.unregisterReceiver()`取消注册

###2) 注意事项：
* 本地广播**无法通过静态注册**的接收器来接收
* 在广播中启动`Activity`的话，需要为`intent`加入`FLAG_ACTIVITY_NEW_TASK`的标记，不然会报错，因为需要一个栈来存放新打开的Activity
* 广播中弹出的`Alertdialog`的话，需要设置对话框的类型为`TYPE_SYSTEM_ALERT`，不然无法弹出

###3) 发送广播

**1. MyBcReceiver.java**

```java
public class MyBcReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(final Context context, Intent intent) {
        AlertDialog.Builder dialogBuilder = new AlertDialog.Builder(context);
        dialogBuilder.setTitle("警告：");
        dialogBuilder.setMessage("您的账号在别处登录，请重新登陆~");
        dialogBuilder.setCancelable(false);
        dialogBuilder.setPositiveButton("确定",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        ActivityCollector.finishAll();
                        Intent intent = new Intent(context, LoginActivity.class);
                        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);//intent添加FLAG_ACTIVITY_NEW_TASK的标记，不然会报错
                        context.startActivity(intent);
                    }
                });
        AlertDialog alertDialog = dialogBuilder.create();
        alertDialog.getWindow().setType(
                WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);  //dialog设置TYPE_SYSTEM_ALERT，不然无法弹出
        alertDialog.show();
    }
}
```

**注意**

　　别忘了`AndroidManifest.xml`中加上系统对话框权限： `< uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />`

**2. 在MainActivity中，实例化localBroadcastManager，拿他完成相关操作，另外销毁时 注意unregisterReceiver！**

```java
public class MainActivity extends BaseActivity {

    private MyBcReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;
    private IntentFilter intentFilter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        localBroadcastManager = LocalBroadcastManager.getInstance(this);  // 获得LocalBroadcastManager的实例

        // 初始化广播接收者，设置过滤器
        localReceiver = new MyBcReceiver();
        intentFilter = new IntentFilter();
        intentFilter.addAction("com.jay.mybcreceiver.LOGIN_OTHER");
        localBroadcastManager.registerReceiver(localReceiver, intentFilter); //注册广播

        Button btn_send = (Button) findViewById(R.id.btn_send);
        btn_send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("com.jay.mybcreceiver.LOGIN_OTHER");
                localBroadcastManager.sendBroadcast(intent);  //发送广播
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver); // 取消注册，一定要记得取消注册广播！
    }
}
```
##5. Android 4.3以上版本监听开机启动广播的问题解决：

　　在Android 4.3以上的版本，允许我们将应用安装在SD上，我们都知道是系统开机 间隔一小段时间后，才装载SD卡的，这样我们的应用就可能监听不到这个广播了！ 所以我们需要既监听开机广播又监听SD卡挂载广播！
另外，有些手机可能并没有SD卡，所以这两个广播监听我们不能写到同一个`Intetn-filter`里 而是应该写成两个，配置代码如下：

```xml
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
    <intent-filter>
        <action android:name="ANDROID.INTENT.ACTION.MEDIA_MOUNTED"/>
        <action android:name="ANDROID.INTENT.ACTION.MEDIA_UNMOUNTED"/>
        <data android:scheme="file"/>
    </intent-filter>
</receiver>
```


--
**参考文献：**
* [BroadcastReceiver牛刀小试](http://www.runoob.com/w3cnote/android-tutorial-broadcastreceiver.html "")
* [BroadcastReceiver庖丁解牛](http://www.runoob.com/w3cnote/android-tutorial-broadcastreceiver-2.html "")