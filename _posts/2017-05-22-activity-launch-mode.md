---
layout: post
title: activity launch mode详解
date: 2017-05-22 14:50:20
---
### Activity有四种加载模式：standard(默认)， singleTop， singleTask和 singleInstance。以下逐一举例说明他们的区别：

1. standard：Activity的默认加载方法，即使某个Activity在Task栈中已经存在，另一个activity通过Intent跳转到该activity，同样会新创建一个实例压入栈中。例如：现在栈的情况为：A B C D，在D这个Activity中通过Intent跳转到D，那么现在的栈情况为： A B C D D 。此时如果栈顶的D通过Intent跳转到B，则栈情况为：A B C D D B。此时如果依次按返回键，D  D C B A将会依次弹出栈而显示在界面上。

2. singleTop：如果某个Activity的Launch mode设置成singleTop，那么当该Activity位于栈顶的时候，再通过Intent跳转到本身这个Activity，则将不会创建一个新的实例压入栈中。例如：现在栈的情况为：A B C D。D的Launch mode设置成了singleTop，那么在D中启动Intent跳转到D，那么将不会新创建一个D的实例压入栈中，此时栈的情况依然为：A B C D。但是如果此时B的模式也是singleTop，D跳转到B，那么则会新建一个B的实例压入栈中，因为此时B不是位于栈顶，此时栈的情况就变成了：A B C D B。
3. singleTask：如果某个Activity是singleTask模式，那么Task栈中将会只有一个该Activity的实例。例如：现在栈的情况为：A B C D。B的Launch mode为singleTask，此时D通过Intent跳转到B，则栈的情况变成了：A B。而C和D被弹出销毁了，也就是说位于B之上的实例都被销毁了。

关于singleTask这个网上颇有争议，google api说singTask模式只能启动一个task，且总是位于栈底，这个也不是完全正确分2种情况：

> * 如果在同一个应用(apk)中使用singleTask，假设ActivityA是程序的入口，是默认的模式（standard）,ActivityB是singleTask 模式，由ActivityA启动，刚ActivityB不会位于栈底，不是根元素，不会启动新的task，此种情况ActivityB会和ActivityA在一个栈中，位于ActivityA上面

> * 如果从不同应用启动一个singleTask的activity，刚依赖于此activity所在的栈，如果之前没有运行过，则新建栈处于栈底，如果有运行过，则有可能不在栈底。假设apkA是情况一中的应用，apkB是另外一个测试程序，在apkB中启动apkA中的ActivityB，再分两种情况，如果ActivityB未启动过，刚ActivityB会位于栈底，是根元素，会启动新的task；如果ActivityB启动过，则ActivityB保持原来的位置不变，在栈底或者栈顶，移除掉ActivityB之上所有的activity(如果有)

<img src="/assets/images/post/doze_2.jpg"/>  

此图就是保存了activitY所在的栈的情况，按返回键的时候，会首先依次移除掉activitY所在的栈的activity，然后才是activity2的

`注意：singleTask模式的Activity不管是位于栈顶还是栈底，再次运行这个Activity时，都会destory掉它上面的Activity来保证整个栈中只有一个自己，切记切记`

4. singleInstance：将Activity压入一个新建的任务栈中。例如：Task栈1的情况为：A B C。C通过Intent跳转到D，而D的Launch mode为singleInstance，则将会新建一个Task栈2。此时Task栈1的情况还是为：A B C。Task栈2的情况为：D。此时屏幕界面显示D的内容，如果这时D又通过Intent跳转到D，则Task栈2中也不会新建一个D的实例，所以两个栈的情况也不会变化。而如果D跳转到C，则栈1的情况变成了：A B C C，因为C的Launch mode为standard，此时如果再按返回键，则栈1变成：A B C。也就是说现在界面还显示C的内容，不是D。

好了，现在有一个问题就是这时这种情况下如果用户点击了Home键，则再也回不到D的即时界面了。如果想解决这个问题，可以为D在Manifest.xml文件中的声明加上：
```java
<intent-filter>

       <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />

 </intent-filter>
```
加上这段之后，也就是说该程序中有两个这种声明，另一个就是那个正常的根activity，在打成apk包安装之后，在程序列表中能看到两个图标，但是如果都运行的话，在任务管理器中其实也只有一个。上面的情况点击D的那个图标就能回到它的即时界面（比如一个EditText，以前输入的内容，现在回到之后依然存在）。

 

PS：intent-filter中 <action android:name="android.intent.action.MAIN" />和 <category android:name="android.intent.category.LAUNCHER" />两个过滤条件缺一不可才会在程序列表中添加一个图标，图标下的显示文字是android:label设定的字符串。