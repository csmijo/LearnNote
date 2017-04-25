#Fragment学习

`Android` `Fragment`

--

##1. android.app.Fragment Vs support.v4.Fragment

　　Fragment是在Android3.0（API 11）提出的，所以如果AndroidManifest.xml 中的minSDK> 11，可以使用android.app.Fragment。
　　

　　如果要对API<11进行支持，可以使用support.v4.Fragment

##2. FragmentManager API

　　方法要和使用的包名相一致
###a. android.app.Fragment 
```java
FragmentManager  fm = getFragmentManager();
MyFragment mFragment = (MyFragment)fm.findFragmentByid(R.id.fragment1);
```

###b. support.v4.Fragment

　　宿主Activity要使用FragmentActivity
```java
FragmentManager fm = getSupportFragmentManager();
MyFragment mFragment = (MyFragment)fm.findFragmentByid(R.id.fragment1);
```
##3. FragmentTransaction API

```java
public void addFragment(View view){
    //1. 获得FragmentManager
    FragmentManager fm = getFragmentManager();
    //2. 通过FragmentManager得到FragmentTransaction
    FragmentTransaction ft = fm.beginTransaction();
    //3. 将Fragment添加到容器中
    MyFragment fragment = new MyFrament();
    //参数1：容器id；参数2：Fragment对象；参数3：Fragment的tag
    ft.add(R.id.layout,fragment,"test");
    //4. 提交
    ft.commit();
}
```

##4. Fragment 使用的一些注意事项

* [Android Fragment 的使用，一些你不可不知的注意事项](http://yifeng.studio/2016/12/15/android-fragment-attentions/)