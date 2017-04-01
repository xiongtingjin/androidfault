---
layout: post
title: Activity has leaked window com.android.internal.policy.impl.PhoneWindow$DecorView that was originally added here
date: 2017-04-01 14:45:11
---
### 异常场景：
经常在应用中需要处理一些耗时的工作，诸如读取大文件、访问网络资源等。为了避免因程序假死而带来的糟糕用户体验，通常我们可以通过线程+Handler或者Android提供的AsyncTask来解决该问题，并一般以ProgressDialog等提示性控件来告知用户当前的程序进度。而标题中描述的异常则会常常出现在这样的场景中，并且往往掩盖了导致异常的真正的罪魁祸首。

### 问题原因：
从异常描述中，大致的意思是存在窗口句柄泄露，即未能及时销毁某个PhoneWindow。而这往往误导了我们，把过多的精力放在查找所谓的内存泄露上了。其实存在这么一种情况，即因我们在非主线程中的某些操作不当而产生了一个严重的异常，从而强制当前Activity被关闭。而在关闭的同时，却没能及时的调用dismiss来解除对ProgressDialog等的引用，从而系统抛出了标题中的错误，而掩盖了真正导致这个错误的异常信息。

### 解决方法：

重写Activity的onDestroy方法，在方法中调用dismiss来解除对ProgressDialog等的引用。