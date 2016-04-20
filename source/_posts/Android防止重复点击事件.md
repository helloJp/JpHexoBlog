title: Android防止重复点击事件
date: 2016-04-19 13:46:52
tags: [重复点击,throttleFirst]
---
 > 在项目中我们是经常遇到**快速点击**同一个按钮或者是listView的item,导致**请求了多次网络**或者**同时打开了多次页面**。
 <!--more-->

#### 1.理解原理：

 ```
public class FastClickUtil {

    private static long mLastClickTime;
    private static int INTERVAL_TIME = 800;//允许的间隔最长事件小于这个值 单位:ms

    public static boolean isFastClick() {
        long time = System.currentTimeMillis();
        if (time - mLastClickTime < INTERVAL_TIME) {
            return true;
        }
        mLastClickTime = time;
        return false;
    }

    public static boolean isFastClick(long intervalTime) {
        long time = System.currentTimeMillis();
        if (time - mLastClickTime < intervalTime) {
            return true;
        }
        mLastClickTime = time;
        return false;
    }
}

 ```
  >是的，原理很简洁：
  >
  * 通过记录和保存下每次点击时刻的时间戳 
  * 将本次时间戳减去上一次时间戳，小于间隔时间INTERVAL_TIME,则是fastClick
 
#### 2.使用原理：
 对原理的直接的使用的如下：

```
   mButton.setOnClickListener((view) -> {
            if (FastClickUtil.isFastClick()) {
                return;
            }
            // TODO: 点击事件
        });

```
这个时候懒惰的程序猿们肯定在想了，我是不是疯了😱，要在每一个点击操作前面加入这么一个判断。

#### 3.简易封装

由 *步骤2* 的使用原理，很容易想到将这个isFastClick()的判断封装到接口中去：

```
public abstract class OnIntervalClickListener implements View.OnClickListener {

    private long mLastTime;
    private final int INTERVAL_TIME = 800;

    public abstract void onIntervalClick(View view);

    @Override
    public void onClick(View view) {
        if (isFastClick()) {
            return;
        }
        onIntervalClick(view);
    }

    private boolean isFastClick() {
        long time = System.currentTimeMillis();
        if (time - mLastTime < INTERVAL_TIME) {
            return true;
        }
        mLastTime = time;
        return false;
    }
}


```
这里的OnIntervalClickListener就已经过滤了fastClick的事件，使用方式就变为了：

```
 mButton.setOnClickListener(new OnIntervalClickListener() {
            @Override
            public void onIntervalClick(View view) {
                // TODO: 点击事件
            }
        });

```
是不是觉得这个世界，还是充满着美好的😇。

同理重写 AdapterView.OnItemClickListener 就可以避免 ListView 重复点击的问题。

#### 4.扩展方式
响应式编程：

* **Debounce** 

  函数过滤掉由Observable发射的速率过快的数据；如果在一个指定的时间间隔过去了仍旧没有发射一个，那么它将发射***最后***的那个。

```
  RxView.clicks(mButton).debounce(800, TimeUnit.MILLISECONDS).subscribe(new Action1<Object>() {
            @Override
            public void call(Object o) {
                // TODO: 点击事件 
            }
        });

```
>因此以上代码要响应事件的话，必须是 800ms后，无点击才会进入。**不适合做页面跳转等，立即性的响应**。


* **ThrottleFirst**

	发射在那段时间内的第一项数据。

```
        RxView.clicks(mButton).throttleFirst(800, TimeUnit.MILLISECONDS)
                .subscribe(new Action1<Object>() {
                    @Override
                    public void call(Object o) {
                        // TODO: 点击事件
                    }
                });

```
>以上代码就是响应800ms内的**第一个点击事件**。过滤这个时间段内的其他事件。

* **sample(别名throttleLast)**
	
	发射自上次采样以来它最近发射的数据。与ThrottleFirst采样的方式相反。

```
RxView.clicks(mButton).sample(800, TimeUnit.MILLISECONDS)
                .subscribe(new Action1<Object>() {
                    @Override
                    public void call(Object o) {
                        // TODO: 点击事件
                    }
                });
                
```
> 这里的sample相对应ThrottleFirst，达到与上面debounce，一同的效果。

 




