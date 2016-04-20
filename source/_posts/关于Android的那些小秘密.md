title: 关于Android的那些小秘密
date: 2016-03-11 10:49:30
tags: cardView
---

总结一些Android中比较使用的技巧，或者是比较容易被忽略的东西。

<!--more-->

#### 1. CardView
使用cardView的时候，Lollipop之前的机子出现了**白边**（下图左侧即为白边的设备）。

附上截图：
![white border cardView](http://7xrr0t.com1.z0.glb.clouddn.com/img_cardView_white_border.png)

我的解决方式是在把你的XML文件中CardView的**cardPreventCornerOverlap**设置成 false。

```
 <android.support.v7.widget.CardView
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:cardCornerRadius="8dp"
        app:cardElevation="8dp"
        app:cardPreventCornerOverlap="false"
        >
```
之后的效果就是这个样子了：

![effect cardView](http://7xrr0t.com1.z0.glb.clouddn.com/img_cardView_white_border_fixed.png)

如此，白边的问题就解决了，只需把子控件改成圆角即可。

#### 2.

---- 未完待续 ----







