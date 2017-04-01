---
layout: post
title: Can not perform this action after onSaveInstanceState
date: 2017-02-24 13:26:22
---
首先得了解一下我那项目的一些基本情况，UI结构是TabActivity包含着5个Tabs，每个tab又是一个独立的Activity。

异常是发生在android 4.03系统上，当我在某个Tab上按Back键时，就会报出java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState

从logout里发现了整个异常发生的过程：
```java
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1438)
at android.support.v4.app.FragmentManagerImpl.popBackStackImmediate(FragmentManager.java:549)
at android.support.v4.app.FragmentActivity.onBackPressed(FragmentActivity.java:166)
at android.app.Activity.onKeyUp(Activity.java:2204)
at android.view.KeyEvent.dispatch(KeyEvent.java:2664)
at android.app.Activity.dispatchKeyEvent(Activity.java:2434)
at com.android.internal.policy.impl.PhoneWindow$DecorView.dispatchKeyEvent(PhoneWindow.java:1962)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.widget.TabHost.dispatchKeyEvent(TabHost.java:324)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1408)
at com.android.internal.policy.impl.PhoneWindow$DecorView.superDispatchKeyEvent(PhoneWindow.java:2035)
at com.android.internal.policy.impl.PhoneWindow.superDispatchKeyEvent(PhoneWindow.java:1505)
at android.app.Activity.dispatchKeyEvent(Activity.java:2429)
...
```
上面的异常信息表示，我写的类不是异常的源头。根据异常信息Can not perform this action after onSaveInstanceState，可以了解到异常原因：在onSaveInstanceState行为之后，app执行某个不能响应的行为而导致异常发生。

在信息at android.app.Activity.onBackPressed(Activity.java:2066)，这一句表明异常是在响应返回键响应事件的行为上发生的。我们顺藤摸瓜，考究一下在我们按下返回键时，activity会执行的响应：onKeyDown–>onBackPressed–>onPause->onStop->onDestroy。

那导火索onSaveInstanceState又是在什么时候执行的？

我们先看android API的一段原文：
先看Application Fundamentals上的一段话：
Android calls onSaveInstanceState() before the activity becomes vulnerable to being destroyed by the system, but does not bother calling it when the instance is actually being destroyed by a user action
(such as pressing the BACK key)

从上面可以知道，当某个activity变得“容易”被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。

注意上面的双引号，何为“容易”？言下之意就是该activity还没有被销毁，而仅仅是一种可能性。

onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。

那为什么项目里头响应onBackPressed事件时会报出上面的异常呢，还表明是after onSaveInstanceState？

原因是我Tab里面的Activity响应了onBackPressed事件，得弹出task，作为它的父容器TabActivity当然也得弹出task，TabActivity 变得“容易”被系统销毁，于是就调用onSaveInstanceState保存状态。

现在整个流程都明白了，可是，这一切都很正常啊，这个流程也很符合Activity的生命周期啊，为什么还会报异常呢？还是在最新的android 4.03上出问题，难道是说，系统不兼容？

经过一番网上查阅，发现API 11 以上某些控件，包括 Fragment还有ActivityGroup，在调用saveInstanceState存在Bug，可能是google对saveInstanceState的实现做过修改。

直到隐藏在后面的原因，解决问题的思路就出来了：让父容器TabActivity在不调用saveInstanceState的情况下onDestroy

具体思路在tab上面的activity监听BACK键的事件，响应并拦截，再通过广播方式通知父容器TabActivity，主动销毁自己，达到原来响应onBackPressed退出App的效果。
具体代码如下：
```java
@Override  
    public boolean dispatchKeyEvent(KeyEvent event) {  
        if(event.getKeyCode() == KeyEvent.KEYCODE_BACK) {
            //TODO 代码  
            return true;//注意这儿返回值为true时该事件将不会继续往下传递，false时反之。根据程序的需要调整  
        }  
        return super.dispatchKeyEvent(event);  
    }  
```