---
layout: post
title: 部分Android手机拍照后照片被旋转的解决方案
date: 2017-03-28 19:15:32
---
在部分Android手机（如MT788、Note2）上，使用Camera拍照以后，得到的照片会被自动旋转（90°、180°、270°），这个情况很不符合预期。仔细分析了一下，因为照片属性中是存储了旋转信息的，所以要解决这个问题，可以在onActivityResult方法中，获取到照片数据后，读取它的旋转信息，如果不是0，说明这个照片已经被旋转过了，那么再使用android.graphics.Matrix将照片旋转回去即可。
### 1、读取图片的旋转属性
```java
/**
 * 读取图片的旋转的角度
 *
 * @param path
 * 图片绝对路径
 * @return 图片的旋转角度
 */
private int getBitmapDegree(String path) {
     int degree = 0;
     try {
         // 从指定路径下读取图片，并获取其EXIF信息
         ExifInterface exifInterface = new ExifInterface(path);
         // 获取图片的旋转信息
         int orientation = exifInterface.getAttributeInt(ExifInterface.TAG_ORIENTATION,
         ExifInterface.ORIENTATION_NORMAL);
         switch (orientation) {
             case ExifInterface.ORIENTATION_ROTATE_90:
             degree = 90;
             break;
             case ExifInterface.ORIENTATION_ROTATE_180:
             degree = 180;
             break;
             case ExifInterface.ORIENTATION_ROTATE_270:
             degree = 270;
             break;
            }
        } catch (IOException e) {
            e.printStackTrace();
     }
     return degree;
}
```
很显然，设计者把Fragment的下标+1左移16位来标记这个request是不是Fragment的，拿到result再解码出下标，直接取对应的Fragment，这样并没有去考虑对Fragment嵌套Fragment做一个Map映射，所以出现了这种BUG。

但是如果我们需要在OnActivityResult的时候处理一些事情的话，我们可以通过在子Fragment中以getParentFragment.startActivityForResult的方式来启动，然后在父Fragment中去接收数据，我们需要在子Fragment中提供一个方法，如：getResultData（Object obj），通过父Fragment中的子Fragment实例去调用这个方法，把相应的数据传过去，然后去更新子Fragment。

以上是在使用Fragment去嵌套Fragment的时候可能会遇到的BUG，了解了BUG存在的原因之后，就可以完美的解决问题。