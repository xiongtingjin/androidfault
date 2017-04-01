---
layout: post
title: 解决TransactionTooLargeException异常
date: 2016-11-18 23:10:22
---
今天查看后台Crash突然出现了看到了这个异常，次数还不少：

```java
android.os.TransactionTooLargeException
at android.os.BinderProxy.transactNative(Native Method)
at android.os.BinderProxy.transact(Binder.java:504)
at android.app.ActivityManagerProxy.startActivity(ActivityManagerNative.java:2487)
at java.lang.reflect.Method.invoke(Native Method)
at java.lang.reflect.Method.invoke(Method.java:372)
at androidx.pluginmgr.hook.AMSHook$HookHandler.invoke(AMSHook.java:68)
at java.lang.reflect.Proxy.invoke(Proxy.java:397)
at $Proxy1.startActivity(Unknown Source)
at android.app.Instrumentation.execStartActivity(Instrumentation.java:1491)
at android.app.Activity.startActivityForResult(Activity.java:3780)
  
```

经用户反馈是打开详情页就崩，google了一下，在Stack Overflow上找到原因，总结下就是Intent的传输的数据过大。
经过测试2个手机大概数据大小极限是300K-400k之间，也可能环境不同这个数值会有所不同。

#### 话不多说，上解决方案：
1. 用静态变量，定义一个全局的缓存HashMap，然后把对应的key通过Intent传输。
2. 使用本地文件缓存。

大家根据具体情况来使用，或者有更好的方案可以交流下，这里推荐第一种。