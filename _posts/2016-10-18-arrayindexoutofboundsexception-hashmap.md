---
layout: post
title: 解决ArrayIndexOutOfBoundsException in HashMap
date: 2016-10-18 23:10:22
---
在平时开发中，我们经常采用HashMap来作为本地缓存的一种实现方式，但是HashMap并不是线程安全的，如果你在多个线程中同事做put操作而没有对象锁的控制，就可能出现异常。
为什么会出现问题呢，假如在默认情况下，一个HashMap的容量为10，当我们往HashMap中put的值到达10时，它将进行自动扩容，假如每次扩容为10个，如果两个线程同时遇到HashMap的大小达到10的倍数时，就很有可能会出现在将oldTable转移到newTable的过程中遇到问题，从而导致最终的HashMap的值存储异常。

举个栗子：
```java
public class Test {

    public static final HashMap<String, String> hashMap = new HashMap<String, String>();

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread() {
            public void run() {
                for (int i = 0; i < 50; i++) {
                    hashMap.put(String.valueOf(i), String.valueOf(i));
                }
            }
        };

        Thread t2 = new Thread() {
            public void run() {
                for (int i = 50; i < 100; i++) {
                    hashMap.put(String.valueOf(i), String.valueOf(i));
                }
            }
        };

        t1.start();
        t2.start();

        // 主线程休眠1秒钟，以便t1和t2两个线程将hashMap填充完毕。
        Thread.currentThread().sleep(1000);

        for (int i = 0; i < 50; i++) {
            // 如果key和value不同，说明在两个线程put的过程中出现异常。
            if (!String.valueOf(i).equals(hashMap.get(String.valueOf(i)))) {
                System.out.println(String.valueOf(i) + ":" + hashMap.get(String.valueOf(i)));
            }
        }

    }
}
  
```

多试几次可能会打印出如下结果说明出现异常了：
0:null
3:null
9:null

解决办法很简单:
1. 可以使用ConcurrentHashMap就不会出现这个问题
2. 使用锁
```java
synchronized(locker){
        hashMap.put(String.valueOf(i), String.valueOf(i));
}
```