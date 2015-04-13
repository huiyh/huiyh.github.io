---
layout: post
category: Java
title: 如何修改jar文件的包名
description: ""
modified: 2014-02-24
tags: [java]
comments: true
share: true
image:
  feature: abstract-2.jpg
  credit: 
  creditlink: 
comments: true
---
首先我们要说说为什么要修改jar文件的包名，jar包我们会非常高频率的使用，尤其是用一些其他三方库的时候，往往把java代码打包成jar文件，方便项目依赖使用。笔者在开发过程中遇到过一些问题，是由于jar文件里面的包名和项目或者运行时环境的引用文件包名上存在冲突导致的，这时候我们修改jar文件的包名是一个比较快速有效的解决办法。举两个例子，是笔者亲身经历的两个情况。

* Gson库：这个相信很多人都在使用，但是直接导入gson的jar包，在HTC Desire HD这款手机上会抛出异常（TypeNotFoundException）
* Jackson库：这也是个json解析的，我们在做苹果推送后台的时候，后台用java写的，使用java-apns库，依赖于Jackson，然后运行就会抛出异常，具体原因没有查，初步怀疑包名冲突，修改之后就解决的。

下面介绍如何修改jar文件的包名，需要用到一个小工具，叫做jarjar.jar

* 下载地址：[http://code.google.com/p/jarjar/downloads/list](http://code.google.com/p/jarjar/downloads/list)

这里我们以gson.jar为例，在gson.jar包目录下新建一个文本文件，名字随意，例如rule.txt，写入下面的内容。

{% highlight java %}

rule com.google.gson.** com.google.mygson.@1

{% endhighlight %}

上面写的就是修改规则，我们将包名中的gson修改为mygson。打开命令行，输入如下命令。


{% highlight java %}

java -jar jarjar.jar process rule.txt gson.jar mygson.jar

{% endhighlight %}

命令执行完毕，同目录下会多出一下mygson.jar，这个就是修改包名之后的jar文件，我们项目导入这个jar包使用即可。