#Service和Thread的关系

`Android` `Service` `Thread`

--

##1. Service和Thread的关系

### PartA

　　不少Android初学者都可能会有这样的疑惑，Service和Thread到底有什么关系呢？什么时候应该用Service，什么时候又应该用Thread？

　　答案可能会有点让你吃惊，因为**Service和Thread之间没有任何关系**！

　　之所以有不少人会把它们联系起来，主要就是因为**Service的后台概念**。Thread我们大家都知道，是用于开启一个子线程，在这里去执行一些耗时操作就不会阻塞主线程的运行。而Service我们最初理解的时候，总会觉得它是用来处理一些后台任务的，一些比较耗时的操作也可以放在这里运行，这就会让人产生混淆了。但是，如果我告诉你**Service其实是运行在主线程里的**，你还会觉得它和Thread有什么关系吗？

　　你可能会惊呼，这不是坑爹么！？那我要Service又有何用呢？其实大家不要把后台和子线程联系在一起就行了，这是两个完全不同的概念。

　　**Android的后台就是指，它的运行是完全不依赖UI的**。即使Activity被销毁，或者程序被关闭，**只要进程还在，Service就可以继续运行**。比如说一些应用程序，始终需要与服务器之间始终保持着心跳连接，就可以使用Service来实现。你可能又会问，前面不是刚刚验证过Service是运行在主线程里的么？在这里一直执行着心跳连接，难道就不会阻塞主线程的运行吗？当然会，但是我们可以在Service中再创建一个子线程，然后在这里去处理耗时逻辑就没问题了。

　　额，既然在Service里也要创建一个子线程，那为什么不直接在Activity里创建呢？这是因为**Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作**。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

### PartB

作者：rlei

链接：https://www.zhihu.com/question/19591125/answer/15998566

来源：知乎

著作权归作者所有，转载请联系作者获得授权。


　　把Service等同于thread或process是一个非常常见的误解。需要**强调又强调**的第一点是，**Android的Service是一个Context，并不必然等于一个额外的thread**。如果你有仔细看文档(Service | Android Developers)，里面专门强调
* **A Service is not a separate process**. The Service object itself does not imply it is running in its own process; unless otherwise specified, it runs in the same process as the application it is part of.
* **A Service is not a thread**. It is not a means itself to do work off of the main thread (to avoid Application Not Responding errors).

　　那Service是什么？**"A Service is an application component representing either an application's desire to perform a longer-running operation while not interacting with the user or to supply functionality for other applications to use."** 

　　Service只是从逻辑上表示一个用来执行**"longer-running"**的"后台"任务或功能的**"application component"**。

　　为什么强调说**"longer running"**的**"component"**? 非常明显，这里是和Activity这种活跃周期相对短暂的component对比而言。一旦用户切换到其他应用，当前Activity就必须pause, stop甚至被destroy，不能在后台处理其他事务。需要强调的事，严格来说你仍然可以在activity里创建自己的worker thread或async task之类做后台的事情，但这样做没有任何保障——**因为一旦你所有的activity都被切换到了后台，系统随时可能kill掉你的process，你的后台任务随时可能悄无声息的死掉。**

　　Activity为什么这样设计？Windows下面的窗口在最小化的时候不是一样可以处理消息或者继续运行吗？这是另外一个话题，不过回答也很简单：这是移动设备，内存/电池都有限。

　　对你的应用而言，通过在manifest里声明Service，把需要后台相对长期运行的逻辑放在Service里，你便获得了这样的保障：只要系统内存不是极端不够用，你的Service一定不会被kill掉。对系统而言，当看到一个进程里有Service在运行，这个进程就具有较高的优先级，会在内存不足被杀的行列里排得比较靠后。

　　可是前面不是强调了Service并不等于process或thread吗？为什么上面又在说杀进程什么的？这里再次强调，你可以把Service看成一砣代码，用来在后台做些事情，仅此而已。至于这砣代码在哪里运行，完全是取决于你自己的喜好。你的local service，如果不建立worker thread，仍然是在你的应用进程的主线程即UI线程里运行。如果你不想阻塞UI线程，你就建一个worker thread。但这些细节，Android framework并不怎么care，它只知道你声明了一个service，然后在你的manifest里面找到这个service是声明在哪个process里运行，那么这个process就不容易被kill。

　　什么时候选择local service（即不指定额外的进程），什么时候选择remote service（额外的进程）？**通常我们会把真的需要长期运行的service（例如IM之类）放在单独的进程里**，这样UI所在的进程在必要的时候仍然可以被系统kill掉来腾出内存。**而local service通常用来处理一些需要短期运行但仍然超出activity活动周期的任务**，打个比方，发送短信或彩信。这样的任务执行完以后，service就可以stop自己，仍然不妨碍整个UI进程被回收掉。

--
**[参考]**
* [ Android Service完全解析，关于服务你所需知道的一切(上)](http://blog.csdn.net/guolin_blog/article/details/11952435)
* [Android中Local Service最本质的作用是什么？](https://www.zhihu.com/question/19591125)