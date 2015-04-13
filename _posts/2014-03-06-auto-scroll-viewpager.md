---
layout: post
category: android-opensource
title: Android自动滚动轮播循环的ViewPager
description: ""
modified: 2014-03-06
tags: [android]
comments: true
share: true

comments: true
---
[代码下载地址](http://pan.baidu.com/s/1kTyoCEr)

简单的通过handler发送消息去完成一次滚动，在完成一次滚动后发送另外一个delay的滚动消息，如此循环实现。

####一. 如何使用

{% highlight xml %}

<自己的包名.AutoScrollViewPager
android:id="@+id/view_pager"
android:layout_width="match_parent"
android:layout_height="wrap_content" />

{% endhighlight %}

{% highlight java %}

AutoScrollViewPager viewPager = (AutoScrollViewPager)findViewById(R.id.view_pager);
viewPager.setAdapter(new ImagePagerAdapter(context, imageIdList));
viewPager.setOnPageChangeListener(new MyOnPageChangeListener());
// 可以放自己的小圆点或字样 implements OnPageChangeListener viewPager.setInterval(2000);
viewPager.startAutoScroll();

{% endhighlight %}

####二. 属性介绍

*    startAutoScroll() 启动自动滚动
*    stopAutoScroll() 停止自动滚动
*    setInterval(long) 设置自动滚动的间隔时间，单位为毫秒
*    setDirection(int) 设置自动滚动的方向，默认向右
*    setCycle(boolean) 是否自动循环轮播，默认为true
*    setScrollDurationFactor(double) 设置ViewPager滑动动画间隔时间的倍率，达到减慢动画或改变动画速度的效果
*    setStopScrollWhenTouch(boolean) 当手指碰到ViewPager时是否停止自动滚动，默认为true
*    setSlideBorderMode(int) 滑动到第一个或最后一个Item的处理方式，支持没有任何操作、轮播以及传递到父View三种模式
*    setBorderAnimation(boolean) 设置循环滚动时滑动到从边缘滚动到下一个是否需要动画，默认为true



