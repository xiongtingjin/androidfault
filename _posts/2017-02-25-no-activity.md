---
layout: post
title: java.lang.IllegalStateException: No activity
date: 2017-02-25 08:12:42
---
当使用Fragment去嵌套另外一些子Fragment的时候，我们需要去管理子Fragment，这时候需要调用ChildFragmentManager去管理这些子Fragment，由此可能产生的Exception主要是：
java.lang.IllegalStateException: No activity

首先我们来分析一下Exception出现的原因：
通过DEBUG发现，当第一次从一个Activity启动Fragment，然后再去启动子Fragment的时候，存在指向Activity的变量，但当退出这些Fragment之后回到Activity，然后再进入Fragment的时候，这个变量变成null，这就很容易明了为什么抛出的异常是No activity

这个Exception是由什么原因造成的呢？如果想知道造成异常的原因，那就必须去看Fragment的相关代码，发现Fragment在detached之后都会被reset掉，但是它并没有对ChildFragmentManager做reset，所以会造成ChildFragmentManager的状态错误。

找到异常出现的原因后就可以很容易的去解决问题了，我们需要在Fragment被detached的时候去重置ChildFragmentManager，即：
```java
@Override
public void onDetach() {
    super.onDetach();
    try {
      Field childFragmentManager = Fragment.class
          .getDeclaredField("mChildFragmentManager");
      childFragmentManager.setAccessible(true);
      childFragmentManager.set(this, null);
    } catch (NoSuchFieldException e) {
      throw new RuntimeException(e);
    } catch (IllegalAccessException e) {
      throw new RuntimeException(e);
    }
}
```