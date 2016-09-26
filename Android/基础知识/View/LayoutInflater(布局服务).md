#LayoutInflater(布局服务)

`Android` `LayoutInflater`

--

##1. LayoutInflater的相关介绍

###1) LayoutInflater的介绍

　　LayoutInflater是一个用于加载布局的系统服务，就是实例化与`Layout XML`文件对应的View对象。 它不能直接使用，需要通过`getLayoutInflater()`方法或`getSystemService()`方法来获得与当前`Context`绑定的 `LayoutInflater`实例!

###2) LayoutInflater的用法

1. 获取LayoutInflater实例的三种方法：
    * `LayoutInflater inflater1 = LayoutInflater.from(this);`
    * `LayoutInflater inflater2 = getLayoutInflater();`
    * `LayoutInflater inflater3 = (LayoutInflater)getSystemService(LAYOUT_INFLATER_SERVICE);`

2. 加载布局的方法

    `public View inflate(int resource,ViewGroup root,boolean attachToRoot)`
    * **resource：** 要加载的布局对应的资源id
    * **root:** 为该布局的外部再嵌套一层父布局，如果不需要的话，写null即可
    * **attToRoot：** 是否为加载的布局文件的最外层套一层root布局

##2. 纯Java代码加载布局

```java
public class MainActivity extends Activity{
    
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        
        /*1. 创建容器和组件*/
        RelativeLayout rly = new RelativeLayout(this);   
        Button btn_one = new Button(this);
        Button btn_two = new Button(this);
        
        /*2. 为容器或组件设置相关属性*/
        btn_one.setText("按钮一");
        btn_two.setText("按钮二");

        /*为按钮一设置一个id值，必须手动设置，这是RelativeLayout的纯代码加载的缺点*/  
        btn_one.setId(123);      

        /*设置按钮一的位置，在父容器中居中*/
        Relativelayout.LayoutParams params1 = new RelativeLayout.LayoutParams(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
        params1.addRule(RelativeLayout.CENTER_IN_PARENT);
        
        /*设置按钮二的位置，在按钮一的下方，并且对齐父容器右边*/
        RelativeLayout.LayoutParams params2 = new RelativeLayout.LayoutParams(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
        params2.addRule(RelativeLayout.BELOW,123);
        params2.addRule(RelativeLayout.ALIGN_PARENT_RIGHT);
        
        /*3. 将组件添加到外部容器中*/
        rly.addView(btn_one,params1);
        rly.addView(btn_two,params2);
        
        /*4. 设置当前视图加载的view即rly*/
        setContentView(rly);
    }
}
```

##3. Java代码动态添加控件或xml布局

###1) Java代码动态添加View

　　**动态添加组件的写法有两种**，区别在于是否需要先**setContentView(R.Layout.activity_main)**。

先写个布局文件：`activity_main.xml`
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:id="@+id/RelativeLayout1"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
  
    <TextView   
        android:id="@+id/txtTitle"  
        android:layout_width="match_parent"  
        android:layout_height="wrap_content"  
        android:text="我是xml文件加载的布局"/>  
     
</RelativeLayout>  
```
####第一种：不需要setContentView（）先加载布局文件 
```java
public class MainActivity extends Activity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        Button btnOne = new Button(this);  
        btnOne.setText("我是动态添加的按钮");  
        RelativeLayout.LayoutParams lp2 = new RelativeLayout.LayoutParams(    
                LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);    
        lp2.addRule(RelativeLayout.CENTER_IN_PARENT);    
       
        /*使用LayoutInflater加载布局文件，然后在查找RelativeLayout1*/
        LayoutInflater inflater = LayoutInflater.from(this);  
        RelativeLayout rly = (RelativeLayout) inflater.inflate(  
                R.layout.activity_main, null)  
                .findViewById(R.id.RelativeLayout1);  
        rly.addView(btnOne,lp2);  
        setContentView(rly);  
    }  
}
```

#####第二种：需要setContentView（）先加载布局文件 

```java
public class MainActivity extends Activity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        
        Button btnOne = new Button(this);  
        btnOne.setText("我是动态添加的按钮");  
        RelativeLayout.LayoutParams lp2 = new RelativeLayout.LayoutParams(    
                LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);    
        lp2.addRule(RelativeLayout.CENTER_IN_PARENT);   
        
        /*已经加载了activity_main，直接查找RelativeLayout1*/ 
        RelativeLayout rly = (RelativeLayout) findViewById(R.id.RelativeLayout1);  
        rly.addView(btnOne,lp2);  
    }  
} 
```
###2) Java代码动态加载xml布局

**activity_main.xml**
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/RelativeLayout1"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <Button
        android:id="@+id/btnLoad"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="动态加载布局"/>
</RelativeLayout>  
```

**inflate.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:gravity="center"  
    android:orientation="vertical"  
    android:id="@+id/ly_inflate" >  
  
    <TextView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="我是Java代码加载的布局" />  
  
    <Button  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="我是布局里的一个小按钮" />  
  
</LinearLayout> 
```

**MainActivity.java**
```java
public class MainActivity extends Activity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        
        /*1. 获得LayoutInflater对象*/  
        final LayoutInflater inflater = LayoutInflater.from(this);    
        //获得外部容器对象  
        final RelativeLayout rly = (RelativeLayout) findViewById(R.id.RelativeLayout1);  
        Button btnLoad = (Button) findViewById(R.id.btnLoad);  
        btnLoad.setOnClickListener(new OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                //2. 加载要添加的布局对象  
                LinearLayout ly = (LinearLayout) inflater.inflate(  
                        R.layout.inflate, null, false).findViewById(  
                        R.id.ly_inflate);  
                //3. 设置加载布局的大小与位置  
                RelativeLayout.LayoutParams lp = new RelativeLayout.LayoutParams(    
                        LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);    
                lp.addRule(RelativeLayout.CENTER_IN_PARENT);    
                //4. 添加到外层容器中
                rly.addView(ly,lp);  
            }  
        });  
    }  
} 
```

