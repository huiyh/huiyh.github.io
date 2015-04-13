---
layout: post
category: android
title: Eclipse ADT 中 libs jar 包关联源代码 和 javadoc
description: ""
modified: 2014-03-10
tags: [android]
comments: true
share: true

comments: true
---

####一. 操作如下
*      把库的 .jar 文件放到 libs  目录中，对应的 source .jar （或者 doc .jar）文件放到和 libs 同级的 libs-src 目录（可以为其他任意目录，但是不要放到 libs 目录中）。
*      在libs 目录中创建一个  .properties 文件，该文件的名字为 .jar 库的名字加上  .properties 后缀。例如 butterknife-4.0.1.jar 对应的 .properties 名字为 butterknife-4.0.1.jar.properties 
*      在 .properties 文件中设置  sources 和 javadoc .jar 包的路径。
*      关闭 并重新打开该 Android 项目！如果不可关联代码可以尝试用  F5 刷新下项目文件或者重启 eclipse。

[更详细的过程和截图可以参考这里](http://stackoverflow.com/questions/9873152/how-to-attach-javadoc-or-sources-to-jars-in-libs-folder)

