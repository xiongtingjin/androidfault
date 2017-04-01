---
layout: post
title: ActivityNotFoundException错误
date: 2016-09-28 12:12:00
---

拨打电话时，手机没有相关应用程序导致的android.content.ActivityNotFoundException错误
错误日志：android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.DIAL dat=tel:xxxxxxxxxxxx }

错误原因：`因为手机没有安装可以拨打电话的应用程序`

解决办法:

直接try catch 捕获异常，可以跳转到android.intent.action.CALL这个Action或者提示用户没有相关的应用程序处理此操作。
```java
 try{
        Uri telUri = Uri.parse("tel:" + number);
        Intent telIntent = new Intent(Intent.ACTION_DIAL, telUri);
        startActivity(context, telIntent);
    } catch (ActivityNotFoundException e) {
        try{
            telIntent = new Intent(Intent.ACTION_CALL, telUri);
            startActivity(context, telIntent);
         } catch (ActivityNotFoundException e) {
            Toast.showToast("没有相关的应用程序处理此操作");
         }
    }

```

用浏览器打开网页链接时，若没有安装浏览器，发短信时，没有安装短信程序，发邮件时没有邮件处理程序，也会产生类似的错误，解决办法是一样的。


