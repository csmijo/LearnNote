#ContentProvider初学乍道

`Android` `ContentProvider`

--
###1. ContentProvider概念讲解
![ContentProvider的概述](http://www.runoob.com/wp-content/uploads/2015/08/58811327.jpg "")

###2. 使用系统提供的ContentProvider

**注意事项：**

　　在Android 4.4以下都可以实现写入短信的功能，而在**Android 5.0上就无法写入**，原因是：从5.0开始，默认短信应用外的软件不能以写入短信数据库的形式发短信！

##3. 自定义ContentProvider

`使用的不多`

![自定义ContentProvider](http://www.runoob.com/wp-content/uploads/2015/08/40787698.jpg "")

##4. 通过ContentObserver监听ContentProvider的数据变化

![使用ContentObserver监听](http://www.runoob.com/wp-content/uploads/2015/08/34168859.jpg "")


--
**参考文献：**
* [ContentProvider初探](http://www.runoob.com/w3cnote/android-tutorial-contentprovider.html "")