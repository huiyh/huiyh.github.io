---
layout: post
category: android-opensource
title: WebView添加自定义JS函数
description: ""
modified: 2014-03-01
tags: [android]
comments: true
share: true

comments: true
---



原文连接[Listen to javascript function invocation from java - Android](http://stackoverflow.com/questions/9306801/listen-to-javascript-function-invocation-from-java-android)


假设我有一个托管服务器上的一些HTML5/javascript文件。当单击HTML5页面上的一个按钮一个JavaScript函数将被调用。我要监听被调用函数，并得到该函数返回的json。

注册您的接口，你需要调用addJavascriptInterface方法，您的WebView实例。你需要选择一个名称，并创建一个实现。你的界面应该拦截函数调用，并报告他们的成果，所以，让我们来描述它...

{% highlight java %}

    class FunctionCallInterceptor {
        public void reportCall(String functionName, String result) {
            // TODO some code, handling interception
        }
    }

{% endhighlight %}

并且注册

{% highlight java %}

    mWebView.addJavascriptInterface(new FunctionCallInterceptor(), 'Interceptor');

{% endhighlight %}

来包装你的函数使用这个代码

{% highlight java %}

    wrapFunc('myFunction'); // wraps myFunction in the source

{% endhighlight %}

另外，不要忘了启用JavaScript在所有

{% highlight java %}

    mWebView.getSettings().setJavaScriptEnabled(true);

{% endhighlight %}

时，如何将这个代码嵌入到您的外部网页（我暗示你是不是有什么访问外部JS代码）...要执行任意代码在页面上下文可以使用的WebView的使用loadURL方法

{% highlight java %}

    mWebView.getSettings().setJavaScriptEnabled(true);

{% endhighlight %}

这将不会触发页面重载，只是JavaScript的将被执行。请注意您需要执行此操作后页面完全加载。你可以通过下面的代码得到这样的：

{% highlight java %}

    mWebView.setWebViewClient(new WebViewClient {
        @Override
        public void onPageFinished (WebView view, String url) {
            // here page is loaded
        }
    });

{% endhighlight %}


见setWebViewClient和onPageFinished的细节

另外要注意，什么样的代码，通过使用loadURL电话转到页，不能包含换行符。所以，你需要摆脱他们（通过与string.replace左右）。

ADD因此，最终的解决办法是：

{% highlight java %}

    String wrapFuncCode = "function wrapFunc ...... ";
    // or maybe place it in resources?
    mWebView.addJavascriptInterface(new FunctionCallInterceptor(), 'Interceptor');
    mWebView.getSettings().setJavaScriptEnabled(true);
    mWebView.setWebViewClient(new WebViewClient {
        @Override
        public void onPageFinished (WebView view, String url) {
            // inject wrapper
            // don't forget to remove newline chars
            mWebView.loadUrl("javascript:" + wrapFuncCode.replace('\n', ''));

            // wrap all the functions needed
            String[] funcToWrap = new String[] { 'myFunc1', ... };
            for(String f : funcToWrap) {
                mWebView.loadUrl("javascript:wrapFunc('" + f + "myFunction');");  
            }  
        }
    });

{% endhighlight %}