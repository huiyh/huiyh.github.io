---
layout: post
category: android-opensource
title: ListView 异常处理
description: ""
modified: 2014-03-20
tags: [android]
comments: true
share: true
comments: true
---
有的时候会出现莫名的溢出，用了这个会好，项目中用到的。


{% highlight java %}

public class ListViewEx extends ListView {
	public ListViewEx(Context context) {
		super(context);
	}
	
	public ListViewEx(Context context, AttributeSet attrs) {
		super(context, attrs);
	}
	
	public ListViewEx(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
	}
	
	@Override
	protected void layoutChildren() {
		try {
			super.layoutChildren();
		} catch (Exception e) {
		}
	}

}

{% endhighlight %}



