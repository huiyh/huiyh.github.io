---
layout: post
category: android-development
title: Activity跳转动画
description: ""
modified: 2014-02-11
tags: [actionbar]
comments: true
share: true
image:
  feature: abstract-2.jpg
  credit: 
  creditlink: 
comments: true
---
Activity跳转动画是指一个Activity跳转到另外一个Activity的动画效果，我们可以通过Acitivity的overridePendingTransition()方法实现跳转动画。

这个函数接受两个参数：

* 第一个参数为第一个Activity退出时的动画
* 第二个参数为第二个Activity进入时的动画

这个方法必须在startActivity方法或者finish方法之后调用。

示例：


{% highlight java %}

Intent intent = new Intent(MainActivity.this,OtherActivity.class);
startActivity(intent);
overridePendingTransition(R.anim.push_up_in,R.anim.push_up_out);

{% endhighlight %}

Android 4.1（API16）提供了一个新类[ActivityOptions](http://developer.android.com/reference/android/app/ActivityOptions.html),用来实现Activity的切换动画。

ActivityOptions类提供了三个方法

* makeScaleUpAnimation()
创建一个动画，能够从屏幕指定的位置和指定的大小拉伸一个活动窗口。例如，当打开一个应用时，Android 4.1的主屏幕使用了这个方法。
* makeThumbnailScaleUpAnimation()
创建一个动画，能够从屏幕指定的位置和提供的缩略图拉伸一个活动窗口。例如，在Android 4.1的最近使用程序窗口中，当往回一个应用程序时使用了这个动画。
* makeCustomAnimation()
创建一个动画，由你自己的资源所定义：一个用来定义活动开启的动画，一个用来定义活动被关闭的动画。

通过这三个方法中的一个就可以创建一个ActivityOptions示例，然后调用toBundle()方法获取一个Bundle对象，传递给startActivity方法。

示例：


{% highlight java %}

Intent intent = new Intent(MainActivity.this,OtherActivity.class);
ActivityOptions opts = ActivityOptions.makeCustomAnimation(MainActivity.this, R.anim.fade, R.anim.hold);
startActivity(intent, opts.toBundle());

{% endhighlight %}


<div markdown="0"><a href="https://github.com/ITBox/ActivityTransitionAnimationSample" class="btn btn-info">示例下载</a></div>



