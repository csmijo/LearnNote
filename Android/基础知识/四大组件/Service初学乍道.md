#Service初学乍道

`Android` `Service`

--

##1. Service的生命周期

![service的生命周期](http://www.runoob.com/wp-content/uploads/2015/08/11165797.jpg "")

##2. 生命周期解析

从上图可以看到，Android中使用Service的方式有两种：
* **startService()** 启动service
* **bindService()** 启动service
* 上述两种方法可以同时使用

###1）相关方法详解

* **onCreate()**：当Service第一次被创建后立即回调该方法，该方法在整个生命周期中**`只会调用一次`**！
* **onDestory()**：当Service被关闭时会回调该方法，该方法**`只会回调一次`**！
* **onStartCommand(intent,flag,startId)**：早期版本是`onStart(intent,startId)`, 当客户端调用`startService(Intent)`方法时会回调，可多次调用StartService方法，但不会再创建新的Service对象，而是继续复用前面产生的Service对象，但会继续回调`onStartCommand()`方法！
* **IBinder onOnbind(intent)**：该方法是Service都必须实现的方法，该方法会返回一个 IBinder对象，app通过该对象与Service组件进行通信！
* **onUnbind(intent)**：当该Service上绑定的所有客户端都断开时会回调该方法！

###2）startService启动service 

1. 首次启动会创建一个Service实例,依次调用`onCreate()`和`onStartCommand()`方法,此时Service 进入运行状态,如果再次调用StartService启动Service,将不会再创建新的Service对象, 系统会直接复用前面创建的Service对象,调用它的`onStartCommand()`方法！
2. **但这样的Service与它的调用者无必然的联系**,就是说当调用者结束了自己的生命周期, 但是只要不调用stopService,那么Service还是会继续运行的!
3. **无论启动了多少次Service,只需调用一次StopService即可停掉Service**

###3) bindService启动service 

1. 当首次使用bindService绑定一个Service时,系统会实例化一个Service实例,并调用其`onCreate()`和`onBind()`方法,然后调用者就可以通过IBinder和Service进行交互了,此后如果再次使用bindService绑定Service,系统不会创建新的Sevice实例,**也不会再调用`onBind()`方法**,只会直接把IBinder对象传递给其他后来增加的客户端!
2. 如果我们解除与服务的绑定,只需调用`unbindService()`,此时`onUnbind()`和`onDestory()`方法将会被调用!这是一个客户端的情况,假如是多个客户端绑定同一个Service的话,情况如下 当一个客户完成和service之间的互动后，它调用 `unbindService()` 方法来解除绑定。当所有的客户端都和service解除绑定后，系统会销毁service。（除非service也被`startService()`方法开启）
3. 另外,和上面那张情况不同,bindService模式下的Service是与调用者相互关联的,可以理解为 **"一条绳子上的蚂蚱",要死一起死**,在bindService后,一旦调用者销毁,那么Service也立即终止!

**通过BindService调用Service时调用的Context的bindService的解析** `bindService(Intent Service,ServiceConnection conn,int flags)`

* **service**: 通过该intent指定要启动的Service
* **conn**: ServiceConnection对象,用户监听访问者与Service间的连接情况, 连接成功回调该对象中的`onServiceConnected(ComponentName,IBinder)`方法; 如果Service所在的宿主由于异常终止或者其他原因终止,导致Service与访问者间断开连接时调用`onServiceDisconnected(CompanentName)`方法。**主动通过unBindService() 方法断开并不会调用上述方法**!
* **flags**: 指定绑定时是否自动创建Service(如果Service还未创建), 参数可以是0(不自动创建),BIND_AUTO_CREATE(自动创建)

###4) startService启动service后bindService绑定

　　如果Service已经由某个客户端通过`startService()`启动,接下来由其他客户端再调用`bindService()`绑定到该Service后,调用`unbindService()`解除绑定最后,再调用`bindService()`绑定到Service的话,此时所触发的生命周期方法如下:
>`onCreate( )->onStartCommand( )->onBind( )->onUnbind( )->onRebind( )`

**前提是**:**onUnbind()方法返回true!!!** 这里或许部分读者有疑惑了,调用了`unbindService()`后Service不是应该调用`onDistory()`方法么!其实这是因为这个Service是由我们的`startService()`来启动的 ,所以你调用`onUnbind()`方法取消绑定,Service也是不会终止的!

**得出的结论**: 假如我们使用`bindService()`来绑定一个启动的Service,注意是已经启动的Service!!! 系统只是将Service的内部IBinder对象传递给Activity,并不会将Service的生命周期 与Activity绑定,因此调用`unBindService()`方法取消绑定时,Service也不会被销毁！

##3. bindService启动service具体操作

###3.1 binService使用方法

* **ServiceConnection对象**:监听访问者与Service间的连接情况,如果成功连接,回调`nServiceConnected()`,如果异常终止或者其他原因终止导致Service与访问者断开 连接则回调`onServiceDisconnected()`方法,调用`unBindService()`不会调用该方法!
* `onServiceConnected()`方法中有一个IBinder对象,该对象即可实现与被绑定Service 之间的通信!我们再开发Service类时,默认需要实现`IBinder onBind()`方法,该方法返回的IBinder对象会传到ServiceConnection对象中作为`onServiceConnected()`方法的参数,我们就可以在这里通过这个IBinder与Service进行通信!

**总结：**
>Step 1. 在自定义的Service中继承Binder,实现自己的IBinder对象

>Step 2:通过`onBind()`方法返回自己的IBinder对象

>Step 3:在绑定该Service的类中定义一个ServiceConnection对象,重写两个方法, `onServiceConnected()`和`onDisconnected()`，然后直接读取IBinder传递过来的参数即可!

###3.2 bindService()

1. 一般来讲，有三种方式可以获得 IBinder 对象：
    * 继承 Binder 对象
    * 使用 Messenger
    * 使用 AIDL

2. 继承 Binder 对象
    
    使用方法如上所述
    
3. 使用 Messenger 

* 对于跨进程通信，Messenger 在很多情况下比使用 AIDL 简单的多
* Messenger 的核心其实就是利用 Message 以及 Handler 来进行进程间的通信
* Messenger 进行跨进程通信的一般步骤：
    * 服务端实现一个Handler，由其接受来自客户端的每个调用的回调
    * 使用实现的Handler创建Messenger对象
    * 通过Messenger得到一个IBinder对象，并将其通过onBind()返回给客户端
    * 客户端使用 IBinder 将 Messenger（引用服务的 Handler）实例化，然后使用后者将 Message 对象发送给服务
    * 服务端在其 Handler 中（具体地讲，是在 handleMessage() 方法中）接收每个 Message

4. 使用 AIDL 

* 见笔记 AIDL
* 使用 AIDL 的基本步骤如下：
    * 服务端创建一个AIDL文件，将暴露给客户端的接口在里面声明
    * 在service中实现这些接口
    * 客户端绑定服务端，并将onServiceConnected()得到的IBinder转为AIDL生成的IInterface实例
    * 通过得到的实例调用其暴露的方法
     
5. Messenger 与 AIDL 比较

* 从实现的难度上，Messenger 要简单的多——至少不用写 AIDL 文件，虽然 Messenger 的底层实现还是 AIDL
* 使用 Messenger 还有一个显著的好处是他会把所有的请求排入队列，因此几乎可以不用担心多线程可能会带来的问题
* 如果项目中存在大量并发请求处理，使用 Messenger 就不合适了，应该使用 AIDL。 因为 Messenger 的特性让他只能串行的解决请求
* 使用 Messenger 的时候只能通过 Message 来传递信息实现交互，但是在有些时候也许我们需要直接跨进程调用服务端的方法，这时候只能使用 AIDL


##4. IntentService的使用

　　如果我们直接把耗时线程放在Service的`onStart()`方法中，虽然可以这样做，但是这样很容易会引起ANR异常(Application Not Response)，而Android官方是这样介绍Service的：
![IntentService](http://www.runoob.com/wp-content/uploads/2015/08/71905858.jpg "")

**直接翻译：**
* Service不是一个单独的进程,它和它的应用程序在同一个进程中
* Service不是一个线程,这样就意味着我们应该避免在Service中进行耗时操作

　　于是乎，Android给我们提供了解决上述问题的替代品，就是下面要讲的**IntentService**； IntentService是继承与Service并处理异步请求的一个类,在IntentService中有 一个工作线程来处理耗时操作,请求的Intent记录会加入队列。

**工作流程：**

　　客户端通过`startService(Intent)`来启动IntentService; 我们并不需要手动地区控制IntentService,当任务执行完后,IntentService会自动停止; 可以启动IntentService多次,每个耗时操作会以工作队列的方式在IntentService的 `onHandleIntent()`回调方法中执行,并且每次只会执行一个工作线程,执行完一，再到二。

##5. 一个简单的前台服务的实现

　　学到现在，我们都知道Service一般都是运行在后来的，但是Service的系统优先级 还是比较低的，当系统内存不足的时候，就有可能回收正在后台运行的Service，对于这种情况我们可以使用前台服务，从而让Service稍微没那么容易被系统杀死，当然还是有可能被杀死的。**所谓的前台服务就是状态栏显示的Notification**。

实现起来也很简单，具体步骤如下：
* 在自定义的Service类中，重写`onCreate()`
* 根据自己的需求定制Notification
* 定制完毕后，调用startForeground(1,notification对象)即可！

```java
public void onCreate()
{
    super.onCreate();
    Notification.Builder localBuilder = new Notification.Builder(this);
    localBuilder.setContentIntent(PendingIntent.getActivity(this, 0, new Intent(this, MainActivity.class), 0));
    localBuilder.setAutoCancel(false);
    localBuilder.setSmallIcon(R.mipmap.ic_cow_icon);
    localBuilder.setTicker("Foreground Service Start");
    localBuilder.setContentTitle("Socket服务端");
    localBuilder.setContentText("正在运行...");
    startForeground(1, localBuilder.getNotification());
}
```

##6. 简单定时的后台服务的实现

　　除了上述的前台服务外，实际开发中Service还有一种常见的用法，就是执行定时任务， 比如轮询，就是每间隔一段时间就请求一次服务器，确认客户端状态或者进行信息更新等！而Android中给我们提供的定时方式有两种：使用**`Timer类`**与**`Alarm机制`**！

　　前者不适合于需要长期在后台运行的定时任务，CPU一旦休眠，Timer中的定时任务就无法运行；Alarm则不存在这种情况，他具有唤醒CPU的功能，另外，也要区分CPU 唤醒与屏幕唤醒！
###1. 使用流程:
* **Step 1:获得Service：**

```java
 AlarmManager manager = (AlarmManager)getSystemService(ALARM_SERVICE);
```
* **Step 2:通过set方式设置定时任务：**

```java
int anHour = 2 * 1000;
long triggerAtTime = SystemClock.elapsedRealtime() + anHour;
manager.set(AlarmManager.RTC_WAKEUP,triggerAtTime,pendingIntent);
```
* **Step 3:定义一个Service：**

　　在onStartCommand()中开辟一条事务线程，用于处理一些定时逻辑
* **Step 4：定义一个Broadcast(广播)：**

　　广播用于启动service，最后一定要在AndroidManifest.xml文件中对Service和Broadcast进行注册。

###2. 参数详解：
>`set(int type,long startTime,PendingIntent pi)`

**1.type: 有五个可选值:**

* **AlarmManager.ELAPSED_REALTIME**: 闹钟在手机睡眠状态下不可用，该状态下闹钟使用相对时间（相对于系统启动开始），状态值为3; 
* **AlarmManager.ELAPSED_REALTIME_WAKEUP**: 闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟也使用相对时间，状态值为2； 
* **AlarmManager.RTC**: 闹钟在睡眠状态下不可用，该状态下闹钟使用绝对时间，即当前系统时间，状态值为1；
* **AlarmManager.RTC_WAKEUP**: 表示闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟使用绝对时间，状态值为0; 
* **AlarmManager.POWER_OFF_WAKEUP**: 表示闹钟在手机关机状态下也能正常进行提示功能，所以是5个状态中用的最多的状态之一， 该状态下闹钟也是用绝对时间，状态值为4；**不过本状态好像受SDK版本影响，某些版本并不支持**

**2.startTime:**

　　闹钟的第一次执行时间，以毫秒为单位，可以自定义时间，不过一般使用当前时间。 **需要注意的是**,本属性与第一个属性（type）密切相关:
* 如果第一个参数对应的闹钟 使用的是相对时间（ELAPSED_REALTIME和ELAPSED_REALTIME_WAKEUP），那么本属 性就得使用相对时间（相对于系统启动时间来说）,比如当前时间就表示为: `SystemClock.elapsedRealtime()`；
* 如果第一个参数对应的闹钟使用的是绝对时间 (RTC、RTC_WAKEUP、POWER_OFF_WAKEUP）,那么本属性就得使用绝对时间， 比如当前时间就表示为：`System.currentTimeMillis()`。

**3.PendingIntent**

　　绑定了闹钟的执行动作，比如发送一个广播、给出提示等等。PendingIntent 是Intent的封装类。

###3. 需要注意的是:

* 如果是通过`启动服务`来实现闹钟提示的话， PendingIntent对象的获取就应该采用`Pending.getService (Context c,int i,Intent intent,int j)`方法;
* 如果是通过`广播`来实现闹钟提示的话， PendingIntent对象的获取就应该采用 `PendingIntent.getBroadcast (Context c,int i,Intent intent,int j)`方法；
* 如果是采用`Activity`的方式来实现闹钟提示的话，PendingIntent对象的获取 就应该采用 `PendingIntent.getActivity(Context c,int i,Intent intent,int j) `方法。
* 如果这三种方法错用了的话，虽然不会报错，但是看不到闹钟提示效果。

**另外:**

　　从4.4版本后(API 19),Alarm任务的触发时间可能变得不准确,有可能会延时,是系统 对于耗电性的优化,如果需要准确无误可以调用setExtra()方法~

###4. 核心代码：

```java
public class LongRunningService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        //这里开辟一条线程,用来执行具体的逻辑操作:
        new Thread(new Runnable() {
            @Override
            public void run() {
                Log.d("BackService", new Date().toString());
            }
        }).start();
        AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
        //这里是定时的,这里设置的是每隔两秒打印一次时间=-=,自己改
        int anHour = 2 * 1000;
        long triggerAtTime = SystemClock.elapsedRealtime() + anHour;
        Intent i = new Intent(this,AlarmReceiver.class);
        PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pi);
        return super.onStartCommand(intent, flags, startId);
    }
}
```

**AlarmReceiver.java**
```java
public class AlarmReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Intent i = new Intent(context,LongRunningService.class);
        context.startService(i);
    }
}
```

##7. Service的回收与重启

　　**android系统的设计原则之一是：保证用户的app尽可能久地运行**。。这事与IOS完全相反的设计哲学，孰优孰略，暂且不谈，现在讨论下android系统中对Service实例的回收问题。

　　android系统中，当剩余内存紧急的时候，系统为了保证用户正在进行的工作正常进行以及系统关键的功能能够正常工作，会对已经使用的内存进行部分回收，会把那些不再重要的进程、后台Task、Activity实例、Service实例等进行销毁。所以用户的Service是随时可能被回收的。以上提到，android的设计哲学是让app尽可能久的运行，所以，当机器的内存危机过去之后，android系统会再尝试重建曾经被销毁的实例，比如Service对象就可以被系统再重新创建，并继续之前的工作。

　　Service 的 `onStartCommand()` 方法的返回值是一个整数，取值必须是这几个中的一个**Service.START_NOT_STICKY**、**Service.START_STICKY**、**Service.START_REDELIVER_INTENT**。这几个返回值会影响到android系统对Service的重建。当然，这里的Service指的是“可启动”的Service。这几个值对Service重建的影响如下：

|返回值|系统描述|描述|
|---|:---:|:----:|
|**Service.START_NOT_STICKY**| 如果系统在Service的onStarnCommand方法回调过之后销毁了该Service，系统不会主动重建该Service，直到有新的组件来启动它。|系统对待该Service的态度是，销毁就销毁了，无所谓|
|**Service.START_STICKY**|如果系统在Service的onStarnCommand方法回调过之后销毁了该Service，系统会主动重新创建该Service的对象，会重新调用该Service的OnStartCommand方法，但是传递进去的Intent对象是一个null|系统对待该Service的态度是，等内存不那么紧张的时候重建该Service，但是并不会给它传递Intent对象，而是传递null|
|**Service.START_REDELIVER_INTENT**|如果系统在Service的onStarnCommand方法回调过之后销毁了该Service，系统会主动重新创建该Service的对象，会重新调用该Service的OnStartCommand方法，并且把上销毁前的最后一个Intent 对象传递进去，主要用在需要恢复现场的地方，比如下载文件|系统对待该Service的态度是，等内存不那么紧张的时候重建该Service，并且会把销毁前最后一个Intent对象重新传递进去|


##8. service 的生命周期

当服务与所有客户端之间的绑定全部取消时，Android 系统便会销毁这个服务（除非还使用 onStartCommand() 启动了该服务）。因此，如果服务是纯粹的绑定服务，原则上我们是无需对其生命周期进行管理的—Android 系统会根据它是否绑定到任何客户端帮我们管理。但实际上，我们应该始终在完成与服务的交互时或 Activity 暂停时取消绑定，以便服务能够在未被占用时关闭。

如果你只需要在 Activity 可见时与服务交互，则可以在 onStart() 期间绑定，在 onStop() 期间取消绑定。如果你希望 Activity 在后台停止运行状态下仍可接收响应，则可在 onCreate() 期间绑定，在 onDestroy() 期间取消绑定。但是注意，这意味着你的 Activity 在其整个运行过程中（甚至包括后台运行期间）都需要使用服务，因此如果服务位于其他进程内，那么当你提高该进程的权重时，系统终止该进程的可能性会增加。通常情况下，**切勿在 Activity 的 onResume() 和 onPause() 期间绑定和取消绑定**，因为每一次生命周期转换都会发生这些回调，我们应该使发生在这些转换期间的处理保持在最低水平。此外，如果我们的应用内的多个 Activity 绑定到同一服务，并且其中两个 Activity 之间发生了转换，则如果当前 Activity 在下一次绑定（恢复期间）之前取消绑定（暂停期间），系统可能会销毁服务并重建服务。

此外，如果我们的服务已启动并接受绑定，则当系统调用 onUnbind() 方法时，如果我们想在客户端下一次绑定到服务时接收 onRebind() 调用（而不是接收 onBind() 调用），则可选择返回 true。onRebind() 返回空值，但客户端仍在其 onServiceConnected() 回调中接收 IBinder。下图说明了这种生命周期的逻辑： 

![service生命周期](http://ac-cnyv47la.clouddn.com/e64329342b5d3318.png)


--
参考文献：

* [Service初涉](http://www.runoob.com/w3cnote/android-tutorial-service-1.html "")
* [Service进阶](http://www.runoob.com/w3cnote/android-tutorial-service-2.html "")
* [Android 组件 Service 研究](http://android.jobbole.com/84450/)
* [Android中的Service：Binder，Messenger，AIDL(2)](http://blog.csdn.net/luoyanglizi/article/details/51594016)