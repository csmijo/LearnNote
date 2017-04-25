#AIDL学习

`Android` `AIDL`

--

##1. 概述

　　`AIDL` 是一个缩写，全称是**Android Interface Definition Language**，也就是**Android 接口定义语言**。它主要用于实现**进程间通信**。通过AIDL 可以非常方便的在一个进程中访问另一个进程的数据，甚至调用另一个进程的方法，当然，只能是特定的方法。

##2. AIDL语言

　　`AIDL`语法和`Java`语法相似，只是有一些细微的差别，主要包括以下这些：

* **文件类型**：用AIDL书写的文件的后缀是 `.aidl`，而不是 `.java`。
* **数据类型**：AIDL默认支持一些数据类型，在使用这些数据类型的时候是不需要导包的，但是除了这些类型之外的数据类型，在使用之前必须导包，**就算目标文件与当前正在编写的 .aidl 文件在同一个包下**——在 Java 中，这种情况是不需要导包的。比如，现在我们编写了两个文件，一个叫做**Book.java** ，另一个叫做 **BookManager.aidl**，它们都在 **com.lypeer.aidldemo** 包下 ，现在我们需要在** .aidl **文件里使用** Book **对象，那么我们就必须在** .aidl **文件里面写上** import com.lypeer.aidldemo.Book**; 哪怕`.java`文件和 `.aidl` 文件就在一个包下。

    默认支持的数据类型包括：
    * Java中的八种基本数据类型，包括 `byte`，`short`，`int`，`long`，`float`，`double`，`boolean`，`char`。
    * `String` 类型。
    * `CharSequence`类型。
    * `List`类型：`List`中的所有元素必须是`AIDL`支持的类型之一，或者是一个其他`AIDL`生成的接口，或者是定义的`parcelabl`e（下文关于这个会有详解）。**`List`可以使用泛型。**
    * `Map`类型：Map中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。**Map是不支持泛型的**。

* **定向tag**：这是一个极易被忽略的点——这里的“被忽略”指的不是大家都不知道，而是很少人会正确的使用它。在我的理解里，**定向 tag 是这样的：AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。**   ?????

    另外，**Java 中的基本类型和 String ，CharSequence 的定向 tag 默认且只能是 `in`** 。还有，请注意，请不要滥用定向 tag ，而是要根据需要选取合适的——要是不管三七二十一，全都一上来就用 inout ，等工程大了系统的开销就会大很多——因为排列整理参数的开销是很昂贵的
    
* **两种AIDL文件**：在我的理解里，所有的AIDL文件大致可以分为**两类**。
    * 一类是**用来定义parcelable对象**，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。
    * 一类是**用来定义方法接口**，以供系统使用来完成跨进程通信的。可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。
    
    注：**所有的非默认支持数据类型必须通过第一类AIDL文件定义才能被使用**。
    
    两种`AIDL`文件的示例：
    ```java
    // Book.aidl
    //第一类AIDL文件的例子
    //这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
    //注意：Book.aidl与Book.java的包名应当是一样的
    package com.lypeer.ipcclient;
    
    //注意parcelable是小写
    parcelable Book;
    ```
    
    ```java
        // BookManager.aidl
    //第二类AIDL文件的例子
    package com.lypeer.ipcclient;
    //导入所需要使用的非默认支持数据类型的包
    import com.lypeer.ipcclient.Book;
    
    interface BookManager {
    
        //所有的返回值前都不需要加任何东西，不管是什么数据类型
        List<Book> getBooks();
        Book getBook();
        int getBookCount();
    
        //传参时除了Java基本类型以及String，CharSequence之外的类型
        //都需要在前面加上定向tag，具体加什么量需而定
        void setBookPrice(in Book book , int price)
        void setBookName(in Book book , String name)
        void addBookIn(in Book book);
        void addBookOut(out Book book);
        void addBookInout(inout Book book);
    }
    ```
    
##3. 如何使用 AIDL 文件来完成跨进程通信？

在进行跨进程通信的时候，在AIDL中定义的方法里包含非默认支持的数据类型与否，我们要进行的操作是不一样的。如果不包含，那么我们只需要编写一个AIDL文件，如果包含，那么我们通常需要写 n+1 个AIDL文件（ n 为非默认支持的数据类型的种类数）——显然，包含的情况要复杂一些。所以我接下来将只介绍AIDL文件中包含非默认支持的数据类型的情况，至于另一种简单些的情况相信大家是很容易从中触类旁通的。

###3.1 使用数据类实现 Parcelable 接口

由于不同的进程有着不同的内存区域，并且他们只能访问自己的那一块内存区域，所以我们传递数据时就**必须要将传输的数据转化成能够在内存间流通的形式**。

##4. 源码分析：AIDL文件是怎么工作的？

1. 在写完 AIDL文件后，编译器帮助我们自动生成了一个 BookManager.java 文件。而这个文件才才是真正协助我们完成跨进程通信的。 AIDL 语言只是在简化我们写这个 BookManager.java 文件的工作而已。
2. Proxy 类中 getBooks() 方法分析：
    * _data 用来存储流向服务器端的数据流
    * _reply 用来存储服务器端流回客户端的数据流

3. Proxy 类的方法里面一般的工作流程：
    * 生成 _data 和 _reply 数据流，并向 _data 中存入客户端的数据
    * 通过 transact() 方法将他们传递给服务端，并请求服务端调用指定方法
    * 接收 _reply 数据流，并从中取出服务端传回来的数据

4. 关于 transact() 方法：这是客户端和服务端通信的核心方法。调用这个方法之后，客户端将会挂起当前线程，等候服务端执行完相关任务后通知并接收返回的 _reply 数据流。关于这个方法的传参，这里有两点需要说明的地方：
    * 方法 ID ：transact() 方法的第一个参数是一个方法 ID ，这个是客户端与服务端约定好的给方法的编码，彼此一一对应。在AIDL文件转化为 .java 文件的时候，系统将会自动给AIDL文件里面的每一个方法自动分配一个方法 ID。
    * 第四个参数：transact() 方法的第四个参数是一个 int 值，它的作用是设置进行 IPC 的模式，为 0 表示数据可以双向流通，即 _reply 流可以正常的携带数据回来，如果为 1 的话那么数据将只能单向流通，从服务端回来的 _reply 流将不携带任何数据。
    
        注：AIDL生成的 .java 文件的这个参数均为 0。
    
5. 服务端的一般工作流程：
    * 获取客户端传过来的数据，根据方法 ID 执行相应操作
    * 将传过来的数据取出来，调用本地写好的对应方法
    * 将需要回传的数据写入 reply 流，传回客户端
  
##5. AIDL中的in，out，inout

1. 官网介绍：

    ```
    All non-primitive parameters require a directional tag indicating which way the data goes .
    Either in , out , or inout . Primitives are in by default , and connot be otherwise .
    ```
    直译过来就是：**所有的非基本参数都需要一个定向tag来指出数据流通的方式，不管是 in , out , 还是 inout 。基本参数的定向tag默认是并且只能是 in**。

    官网正确思想理解：**所有的非基本参数都需要一个定向tag来指出数据的流向，不管是 in , out , 还是 inout 。基本参数的定向tag默认是并且只能是 in**。
    
2. in，out，intout
    * in： 表示数据只能由客户端流向服务端
    * out: 表示数据只能由服务端流向客户端
    * inout： 表示数据可以双向流通
    
    其中，数据流向是针对在客户端中的那个传入方法的对象 A 而言的。
    * in：表现为服务端将会将收到一个**那个对象 A 的完整数据**，但是客户端的那个对象 A **不会因为服务端对传参的修改而发生变动**
    * out：表现为服务端将会接收到**那个对象 A 的参数为空的对象**，但是服务端对接收到的空对象**有任何修改之后客户端都将会同步变动**
    * inout：服务端将接收客户端传过来**那个对象 A 的完整数据**，并且客户端将会**同步服务端对该对象的任何变动**


--
[参考文献]

* [学习 AIDL，这一篇文章就够了(上)](http://android.jobbole.com/84580/)
* [学习 AIDL，这一篇文章就够了(下)](http://android.jobbole.com/84588/)
* [你真的理解AIDL中的in，out，inout么？](http://blog.csdn.net/luoyanglizi/article/details/51958091)