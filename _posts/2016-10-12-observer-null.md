---
layout: post
title: The observer is null错误解决方案
date: 2016-10-12 13:14:12
---
今天看后台crash的时候看到这个IllegalArgumentException:The observer is null这个崩溃，都发生在4.2的机器上。
问题出在ViewPager在移除View时会调用AbsListView的unregisterDataSetObserver方法，而AbsListView本身也会调用该方法，所以在第二次调用时就会报“The observer is null”错误。
解决方案是在Adapter中覆盖下面方法:
```java
@Override  
public void unregisterDataSetObserver(DataSetObserver observer) {  
    if (observer != null) {  
        super.unregisterDataSetObserver(observer);  
    }  
}  
```