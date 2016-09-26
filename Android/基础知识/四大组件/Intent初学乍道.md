#Intent初学乍道

`Android` `Intent`

--

##1. Intent传递集合

###1) `List<基本数据类型或者String>`

**写入集合：**

```java
intent.putStringArrayListExtra(name, value)
intent.putIntegerArrayListExtra(name, value)
```

**读取集合：**

```java
intent.getStringArrayListExtra(name)
intent.getIntegerArrayListExtra(name)
```

###2) `List<Object>`

将list强制转换成Serializable类型，然后传入(使用Bundle做媒介)

**注意：**Object类需要实现Serializable接口

**写入集合：**

```java
putExtras(key, (Serializable)list)
```

**读取集合：**

```java
(List<Object>) getIntent().getSerializable(key)
```

###3) `Map<String,Object>,或更复杂的`
**解决方法：** 外层嵌套List

```java
//传递复杂些的参数 
Map<String, Object> map1 = new HashMap<String, Object>();  
map1.put("key1", "value1");  
map1.put("key2", "value2");  
List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();  
list.add(map1);  

Intent intent = new Intent();  
intent.setClass(MainActivity.this,ComplexActivity.class);  
Bundle bundle = new Bundle();  

//须定义一个list用于在budnle中传递需要传递的ArrayList<Object>,这个是必须要的  
ArrayList bundlelist = new ArrayList();   
bundlelist.add(list);   
bundle.putParcelableArrayList("list",bundlelist);  
intent.putExtras(bundle);                
startActivity(intent); 
```

##2. Intent传递对象

　　传递对象的方式有两种：**将对象转换为Json字符串**或者**通过Serializable,Parcelable序列化**。

###1. 将对象转成Json字符串传递

　　不建议使用Android内置的抠脚Json解析器，可使用fastjson或者Gson第三方库！

**写入数据**

```java
Book book=new Book();
book.setTitle("Java编程思想");
Author author=new Author();
author.setId(1);
author.setName("Bruce Eckel");
book.setAuthor(author);
Intent intent=new Intent(this,SecondActivity.class);
intent.putExtra("book",new Gson().toJson(book));
startActivity(intent);
```

**读取数据**

```java
String bookJson=getIntent().getStringExtra("book");
Book book=new Gson().fromJson(bookJson,Book.class);
Log.d(TAG,"book title->"+book.getTitle());
Log.d(TAG,"book author name->"+book.getAuthor().getName());
```

###2. 使用Serializable，Parcelable序列化对象

####1) Serializable实现：

* 业务`Bean`实现`Serializable`接口，写上变量的`getter()`和`setter()`方法;
* Intent通过调用`putExtra(String name, Serializable value)`传入对象实例。当然对象有多个的话多个的话,我们也可以先`Bundle.putSerializable(x,x)`，再`Intent.putExtras()`即可;
* 新`Activity`调用`getSerializableExtra()`方法获得对象实例:
```java
Product pd = (Product) getIntent().getSerializableExtra("Product");
```
* 调用对象get方法获得相应参数

####2) Parcelable实现：

**a. 一般流程：**

* 业务`Bean`继承`Parcelable`接口,重写`writeToParcel`方法,将你的对象序列化为一个`Parcel`对象;
* `重写describeContents`方法，内容接口描述，默认返回`0`就可以
* 实例化静态内部对象`CREATOR`实现接口`Parcelable.Creator`
* 同样式通过`Intent`的`putExtra()`方法传入对象实例,当然多个对象的话,我们可以先放到`Bundle`里`Bundle.putParcelable(x,x)`,再`Intent.putExtras()`即可

**b. 一些解释：**

　　通过`writeToParcel`将你的对象映射成`Parcel`对象，再通过`createFromParcel`将`Parcel`对象映射 成你的对象。也可以将`Parcel`看成是一个流，通过`writeToParcel`把对象写到流里面， 在通过`createFromParcel`从流里读取对象，只不过这个过程需要你来实现，因此**写的顺序和读的顺序必须一致**。

**c. 实现Parcelable接口的代码示例：**

```java
//Internal Description Interface,You do not need to manage  
@Override  
public int describeContents() {  
     return 0;  
}  
       
      
 
@Override  
public void writeToParcel(Parcel parcel, int flags){  
    parcel.writeString(bookName);  
    parcel.writeString(author);  
    parcel.writeInt(publishTime);  
}  
      

public static final Parcelable.Creator<Book> CREATOR = new Creator<Book>() {  
    @Override  
    public Book[] newArray(int size) {  
        return new Book[size];  
    }  
          
    @Override  
    public Book createFromParcel(Parcel source) {  
        Book mBook = new Book();    
        mBook.bookName = source.readString();   
        mBook.author = source.readString();    
        mBook.publishTime = source.readInt();   
        return mBook;  
    }  
};
```
####3) 两种序列化方式的比较

两者的比较:
* **在使用内存的时候，`Parcelable`比`Serializable`性能高**，所以推荐使用`Parcelable`
* `Serializable`在序列化的时候会产生大量的临时变量，从而引起频繁的GC。
* `Parcelable`**不能使用在要将数据存储在磁盘上的情况**，因为在外界有变化的情况下`Parcelable`不能很好的保证数据的持续性。尽管`Serializable`效率低点，但此时还是建议使用Serializable。

##3. Intent传递Bitmap

bitmap默认实现Parcelable接口，直接传递即可。

```java
Bitmap bitmap = null;
Intent intent = new Intent();
Bundle bundle = new Bundle();
bundle.putParcelable("bitmap", bitmap);
intent.putExtra("bundle", bundle);
```

##4. 定义全局数据

　　如果是传递简单的数据，有这样的需求，`Activity1` -> `Activity2` -> `Activity3` -> `Activity4`， 你想在`Activity`中传递某个数据到`Activity4`中，怎么破，一个个页面传么？

　　显然不科学是吧，如果你想某个数据可以在任何地方都能获取到，你就可以考虑使用 `Application`全局对象了！

　　**Android系统在每个程序运行的时候创建一个`Application`对象，而且只会创建一个**，所以`Application`是单例(singleton)模式的一个类，而且`Application`对象的生命周期是整个程序中最长的，**他的生命周期等于这个程序的生命周期**。

　　如果想存储一些比静态的值(固定不改变的，也可以变)，如果你想使用 `Application`就需要自定义类实现`Application类`，并且告诉系统实例化的是我们自定义的`Application`而非系统默认的，而这一步，就是在`AndroidManifest.xml`中为我们的`application`标签添加`:name`属性！

**注意事项：**

　　`Application`对象是存在于内存中的，也就有它可能会被系统杀死，比如这样的场景：

　　我们在`Activity1`中往`application`中存储了用户账号，然后在`Activity2`中获取到用户账号，并且显示！

　　如果我们点击`home`键，然后过了N久候，系统为了回收内存kill掉了我们的app。这个时候，我们重新 打开这个app，这个时候很神奇的，回到了`Activity2`的页面，但是如果这个时候你再去获取`Application`里的用户账号，程序就会报`NullPointerException`，然后crash掉~

　　之所以会发生上述`crash`，**是因为这个`Application`对象是全新创建的**，可能你以为App是重新启动的， 其实并不是，仅仅是创建一个新的`Application`，然后启动上次用户离开时的`Activity`，从而创造`App`并没有被杀死的假象！**所以如果是比较重要的数据的话，建议你还是进行本地化，另外在使用数据的时候 要对变量的值进行非空检查！**

　　还有一点就是：**不止是`Application`变量会这样，单例对象以及公共静态变量**也会这样~

--
**参考文献：**
* [Intent之复杂数据的传递](http://www.runoob.com/w3cnote/android-tutorial-intent-pass-data.html "")
