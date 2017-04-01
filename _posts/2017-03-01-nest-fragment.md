---
layout: post
title: 嵌套的Fragment接收不到onActivityResult事件
date: 2017-03-01 18:25:32
---
当我们从一个Activity启动了一个Fragment，然后在这个Fragment中又去实例化了一些子Fragment，在子Fragment中去有返回的启动了另外一个Activity，即通过startActivityForResult方式去启动，这时候造成的现象会是，子Fragment接收不到OnActivityResult，如果在子Fragment中是以getActivity.startActivityForResult方式启动，那么只有Activity会接收到OnActivityResult，如果是以getParentFragment.startActivityForResult方式启动，那么只有父Fragment能接收（此时Activity也能接收），但无论如何子Fragment接收不到OnActivityResult。

这是一个非常奇怪的现象，按理说，应该是让子Fragment接收到OnActivityResult才对，究竟是什么造成的呢？这是由于某位写代码的员工抱怨没发奖金，稍稍偷懒了，少写了一部分代码，没有考虑到Fragment再去嵌套Fragment的情况。

我们来看看FragmentActivity中的代码：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    this.mFragments.noteStateNotSaved();
    int index = requestCode >> 16;
    if (index != 0) {
      index--;
      if ((this.mFragments.mActive == null) 
                || (index < 0) 
                || (index >= this.mFragments.mActive.size())) {
        Log.w("FragmentActivity", "Activity result fragment index out of range: 0x" 
            + Integer.toHexString(requestCode));
        return;
      }
      Fragment frag = (Fragment)this.mFragments.mActive.get(index);
      if (frag == null) {
        Log.w("FragmentActivity", "Activity result no fragment exists for index: 0x" 
        + Integer.toHexString(requestCode));
      }else {
            frag.onActivityResult(requestCode & 0xFFFF, resultCode, data);
      }
      return;
    }
    super.onActivityResult(requestCode, resultCode, data);
}
```
很显然，设计者把Fragment的下标+1左移16位来标记这个request是不是Fragment的，拿到result再解码出下标，直接取对应的Fragment，这样并没有去考虑对Fragment嵌套Fragment做一个Map映射，所以出现了这种BUG。

但是如果我们需要在OnActivityResult的时候处理一些事情的话，我们可以通过在子Fragment中以getParentFragment.startActivityForResult的方式来启动，然后在父Fragment中去接收数据，我们需要在子Fragment中提供一个方法，如：getResultData（Object obj），通过父Fragment中的子Fragment实例去调用这个方法，把相应的数据传过去，然后去更新子Fragment。

以上是在使用Fragment去嵌套Fragment的时候可能会遇到的BUG，了解了BUG存在的原因之后，就可以完美的解决问题。