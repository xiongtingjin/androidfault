---
layout: post
title: AsyncTask的缺陷和问题
date: 2016-09-21 19:23:11
---



### 1、first step
我们要在父类的FragmentActivity的onActivityResult中进行一步操作
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        /*在这里，我们通过碎片管理器中的Tag，就是每个碎片的名称，来获取对应的fragment*/
        Fragment f = fragmentManager.findFragmentByTag(curFragmentTag);
        /*然后在碎片中调用重写的onActivityResult方法*/
        f.onActivityResult(requestCode, resultCode, data);
    }
}
```
### 1、second step
在Fragment的onActivityResult获取返回值
```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
        case1:
            if (resultCode != Activity.RESULT_OK) {
                return;
            }
            break;

        default:
            break;
        }

    }
```
