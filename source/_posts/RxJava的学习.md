title: RxJava的学习
date: 2016-03-14 11:38:54
tags: RxJava
---


###RxJava的学习

RxJava去年学过，实际使用的不多，对使用的方法也开始变得模糊。所以还是把学过的记个大概吧😋。

<!--more-->

首先是源码主要类的结构图：

![chart]()

* Observer/Subscriber是观察者
* Action0 是 RxJava 的一个接口，它只有一个方法 void call()，这个方法是**无参无返回值**的，Action1 也是一个接口，它同样只有一个方法 void call(T param)，这个方法**有参无返回值**。


参考链接：

[Speaker Deck]

[给 Android 开发者的 RxJava 详解]

[RxJava Wiki]

[ReactiveX文档中文翻译]

[RxJava Essentials 中文翻译版]

[Bookmarks--RxJava]

[Speaker Deck]:https://speakerdeck.com/jakewharton/demystifying-rxjava-subscribers-oredev-2015

[给 Android 开发者的 RxJava 详解]: http://gank.io/post/560e15be2dca930e00da1083

[RxJava Wiki]:https://github.com/ReactiveX/RxJava/wiki

[ReactiveX文档中文翻译]:https://mcxiaoke.gitbooks.io/rxdocs/content/index.html

[RxJava Essentials 中文翻译版]:https://yuxingxin.gitbooks.io/rxjava-essentials-cn/content/index.html

[Bookmarks--RxJava]:http://www.jianshu.com/p/de4cc3c14ff0
