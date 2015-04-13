---
layout: post
category: android-opensource
title: Eclipse导入gradle项目
description: ""
modified: 2014-03-01
tags: [android]
comments: true
share: true

comments: true
---



原文链接[Importing gradle project to eclipse](http://stackoverflow.com/questions/20805715/importing-gradle-project-to-eclipse)

打开build.gradle文件,在第一行添加以下代码


{% highlight java %}

    apply plugin: 'eclipse'

{% endhighlight %}



然后在命令行中项目所在目录运行以下命令

Windows下

{% highlight java %}

    gradlew.bat eclipse

{% endhighlight %}

mac或linux下

{% highlight java %}

    ./gradlew eclipse

{% endhighlight %}


然后再Eclipse中正常导入该项目即可