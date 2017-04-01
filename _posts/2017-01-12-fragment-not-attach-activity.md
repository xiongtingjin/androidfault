---
layout: post
title: 解决Fragment not attach to Acitivity异常
date: 2017-01-12 09:50:32
---
后台莫名奇妙crash了个Fragment not attach to Acitivity ，经过排查我们出现改异常的情况是当网络不好，打开fragment后去请求网络，这时网络还没有返回数据，直接finish，当网络超时的时候会调用failed回调方法，在回调方法中写了fragment.getString()，获取错误的提示消息。
出现该异常，是因为Fragment的还没有Attach到Activity时，调用了如getResource()等。
#### 解决办法：
1. 可以在onCreate或onStart方法中把需要是数据提前初始化。
2. 在之前增加一个isAdded()。