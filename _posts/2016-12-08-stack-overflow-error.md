---
layout: post
title: StackOverflowError布局嵌套太深
date: 2016-12-08 12:10:42
---
最近项目中测试在三星的一款低端机是出现了java.lang.StackOverflowError错误，但是在一些性能高点的机器上从来没有出现过。经过万能的google，得知是布局嵌套层次嵌套太深。

#### 解决方法：
1. Android官方SDK提供了一种XML标签， 在官方文档里的标注就是通过merge标签来减少视图层级结构。
2. 用RelativeLayout的性质来减少布局层次（本人就把之前用LinearLayout布局实现的5层改为RelativeLayout用2层实现）