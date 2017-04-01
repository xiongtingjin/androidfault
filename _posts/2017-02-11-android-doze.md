---
layout: post
title: 使Android进入到Doze模式
date: 2017-02-11 15:23:22
---
1. 运行应用程序
2. 关闭设备的屏幕
3. 使用如下命令强制系统进入Doze模式  
$ adb shell dumpsys battery unplug  
$ adb shell dumpsys deviceidle step

第二条命令需要多次运行,直到设备进入到空闲状态  
<img src="/assets/images/post/doze_1.jpg"/>  
如果遇到下面提示就是手机没有这种服务，就无法用命令进入Doze模式了  
<img src="/assets/images/post/doze_2.jpg"/>  