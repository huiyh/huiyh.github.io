---
layout: post
category: android-development
title: NineOldAndroids动画库
description: ""
modified: 2014-05-13
tags: [android]
comments: true
share: true
image:
  feature: abstract-2.jpg
  credit: 
  creditlink: 
comments: true
---


##介绍

![](https://raw.githubusercontent.com/baoyongzhang/test_pages/gh-pages/NineOldAndroids.png)

Android3.0推出了全新的[AnimationAPI](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html)，使用起来很方便，但是不能在3.0以下版本使用，[NineOldAndroids](https://github.com/JakeWharton/NineOldAndroids)是一个可以在任意Android版本上使用的AnimationAPI，API和Android3.0中的类似。

###常用类
* ObjectAnimator
* ValueAnimator 
* AnimatorSet 
* ViewPropertyAnimator 

类名与官方的API是对应的，只是包名为`com.nineoldandroids.animation`。


![](https://raw.githubusercontent.com/baoyongzhang/test_pages/gh-pages/NineOldAndroid_demo.gif)

##使用方法
首先导入NineOldAndroids的jar包。在Android3.0中，`View`中有一个`animate`方法，NineOldAndroids中提供了`ViewPropertyAnimator.animate(View)`与其对应，可以选择静态导入。

{% highlight java %}

// 官方API（3.0以上）
mView.animate().setDuration(5000).rotationY(720).x(100).y(100).start();
		
// NineOldAndroids
ViewPropertyAnimator.animate(mView).setDuration(5000)
				.rotationY(720).x(100).y(100).start();
				
// 可以使用静态导入
import static com.nineoldandroids.view.ViewPropertyAnimator.animate;
// 直接调用animate方法
animate(mView).setDuration(5000).rotationY(720).x(100).y(100).start();

{% endhighlight %}

使用链式编程设置各种属性参数，最终调用`start()`来启动动画，还可以调用`setStartDelay()`设置动画延迟启动。

可以设置动画的监听器，在动画开始、结束等时候加一些处理。


{% highlight java %}


ViewPropertyAnimator
	.animate(mIView)
	.setDuration(5000)
	.rotationY(720)
	.x(100).y(100)
	.setListener(new AnimatorListenerAdapter() {
		@Override
		public void onAnimationStart(Animator animation) {
			super.onAnimationStart(animation);
			// 动画开始
		}
		@Override
		public void onAnimationCancel(Animator animation) {
			super.onAnimationCancel(animation);
			// 动画取消
		}
		@Override
		public void onAnimationEnd(Animator animation) {
			super.onAnimationEnd(animation);
			// 动画结束
		}
		@Override
		public void onAnimationRepeat(Animator animation) {
			super.onAnimationRepeat(animation);
			// 动画重复启动
		}
	}).start();


{% endhighlight %}

`ViewPropertyAnimator`对象提供了取消动画的方法

{% highlight java %}
ViewPropertyAnimator animate = ViewPropertyAnimator.animate(mDropTv);
/* ... */
animate.start();	// 开始动画
animate.cancel();	// 取消动画

{% endhighlight %}

简单的动画效果使用`ViewPropertyAnimator`一般可以满足，下面介绍一下高级玩法。核心是`ObjectAnimator`类。

###举例

* <a name="demo1">背景颜色从红色到蓝色，并反转回去，而且无限重复。</a>

{% highlight java %}


ValueAnimator colorAnim = ObjectAnimator.ofInt(mView, "backgroundColor", /*红色*/0xFFFF8080, /*蓝色*/0xFF8080FF);
colorAnim.setDuration(3000);
colorAnim.setEvaluator(new ArgbEvaluator());	// ARGB
colorAnim.setRepeatCount(ValueAnimator.INFINITE);   // 无限重复
colorAnim.setRepeatMode(ValueAnimator.REVERSE); // 反转回去
colorAnim.start();


{% endhighlight %}

* <a name="demo2">使用动画集合`AnimatorSet`，可以使用多个`ObjectAnimator`，实现复杂的动画效果。</a>

{% highlight java %}


AnimatorSet set = new AnimatorSet();
set.playTogether(
    ObjectAnimator.ofFloat(myView, "rotationX", 0, 360),
    ObjectAnimator.ofFloat(myView, "rotationY", 0, 180),
    ObjectAnimator.ofFloat(myView, "rotation", 0, -90),
    ObjectAnimator.ofFloat(myView, "translationX", 0, 90),
    ObjectAnimator.ofFloat(myView, "translationY", 0, 90),
    ObjectAnimator.ofFloat(myView, "scaleX", 1, 1.5f),
    ObjectAnimator.ofFloat(myView, "scaleY", 1, 0.5f),
    ObjectAnimator.ofFloat(myView, "alpha", 1, 0.25f, 1)
);
set.setDuration(5 * 1000).start();


{% endhighlight %}

`AnimatorSet`主要方法有两个，`playSequentially` 是创建按顺序执行的动画，`playTogether`是创建同时执行的动画。

###ObjectAnimator说明

`ObjectAnimator`是动画对象，通过ObjectAnimator提供的一系列of开头的静态方法创建。

创建一般需要传入三个参数

* `target`，Object类型，可不是View哦
* `PropertyName`，String类型或Property类型，用于描述target中的属性
* 数组，`ofInt()`就是int数组

`ObjectAnimator`原理是这样的：会调用`target`的`set`方法，设置`PropertyName`的值，这个值的计算方式是，根据Duration时长和第三个参数数组来计算出来当前时间的值。然后调用`set`方法设置进去。例如上面更改<a href="#demo1">背景颜色的实例</a>，`PropertyName`是`backgroundColor`，数组是两个颜色值，运行动画就会根据Duration计算当前的颜色值，调用`target`的`setBackgroundColor`方法设置进去，从而改变了背景颜色。

再看改<a href="#demo2">`AnimatorSet`的实例</a>，`PropertyName`是`rotationX`、`translationX`之类的，这几个属性是在Android3.0以上才有的，所以调用`set`方法会出错的，通过观察`ObjectAnimator`，发现对这几个属性做了特殊处理，提前预制了这几个属性值。

{% highlight java %}

static {
        PROXY_PROPERTIES.put("alpha", PreHoneycombCompat.ALPHA);
        PROXY_PROPERTIES.put("pivotX", PreHoneycombCompat.PIVOT_X);
        PROXY_PROPERTIES.put("pivotY", PreHoneycombCompat.PIVOT_Y);
        PROXY_PROPERTIES.put("translationX", PreHoneycombCompat.TRANSLATION_X);
        PROXY_PROPERTIES.put("translationY", PreHoneycombCompat.TRANSLATION_Y);
        PROXY_PROPERTIES.put("rotation", PreHoneycombCompat.ROTATION);
        PROXY_PROPERTIES.put("rotationX", PreHoneycombCompat.ROTATION_X);
        PROXY_PROPERTIES.put("rotationY", PreHoneycombCompat.ROTATION_Y);
        PROXY_PROPERTIES.put("scaleX", PreHoneycombCompat.SCALE_X);
        PROXY_PROPERTIES.put("scaleY", PreHoneycombCompat.SCALE_Y);
        PROXY_PROPERTIES.put("scrollX", PreHoneycombCompat.SCROLL_X);
        PROXY_PROPERTIES.put("scrollY", PreHoneycombCompat.SCROLL_Y);
        PROXY_PROPERTIES.put("x", PreHoneycombCompat.X);
        PROXY_PROPERTIES.put("y", PreHoneycombCompat.Y);
    }


{% endhighlight %}

##总结
NineOldAndroids的API与官方的API基本一致，使用很方便。能够轻松实现各种酷炫动画效果。

* 一般情况使用`ViewPropertyAnimator`就可以了，可以设置动画监听器，实现连贯动画，和其他处理。
* `ObjectAnimator`创建的`target`是`Object`，可以传入任何对象，原理是调用`set`方法，利用这个特性可以实现很多自定义的效果有点和`Scroller`类似。

##参考
* Github主页：[https://github.com/JakeWharton/NineOldAndroids](https://github.com/JakeWharton/NineOldAndroids)
* 官方网站：[http://nineoldandroids.com/](http://nineoldandroids.com/)
* ListView动画库：[https://github.com/nhaarman/ListViewAnimations](https://github.com/nhaarman/ListViewAnimations)
* Android3.0 API：[http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html](http://android-developers.blogspot.com/2011/02/animation-in-honeycomb.html)
* Android官方文档：[http://developer.android.com/reference/android/view/animation/package-summary.html](http://developer.android.com/reference/android/view/animation/package-summary.html)
