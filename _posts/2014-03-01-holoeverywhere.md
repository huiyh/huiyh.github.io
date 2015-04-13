---
layout: post
category: android-opensource
title: HoloEverywhere
description: ""
modified: 2014-03-01
tags: [android]
comments: true
share: true

comments: true
---

![](https://github.com/ITBox/Picture/blob/master/holoeverywhere_logo.png?raw=true)

HoloEverywhere是一个能够让我们在2.1+的系统上实现Android4.0的Holo风格。

下面是我在Android2.3.7上运行的HoloEverywhere示例的几张截图:

![](https://github.com/ITBox/Picture/blob/master/holoeverywhere_1.jpg?raw=true)

![](https://github.com/ITBox/Picture/blob/master/holoeverywhere_2.jpg?raw=true)

![](https://github.com/ITBox/Picture/blob/master/holoeverywhere_3.jpg?raw=true)

![](https://github.com/ITBox/Picture/blob/master/holoeverywhere_4.jpg?raw=true)


###HoloEverywhere使用

HoloEverywhere中集成了android-support-v7-appcompat包，所以在低版本使用actionbar不需要引用任何包。

使用HoloEverywhere只需要以下简单步骤：

1. 所有的Activity继承org.holoeverywhere.app.***Activity
2. 所有的Fragment继承org.holoeverywhere.app.***Fragment
3. 为清单文件<application>标签添加android:name="org.holoeverywhere.app.Application"，或者自定义Application继承org.holoeverywhere.app.Application。
4. 使用org.holoeverywhere.widget包下的控件替代Android原生的控件。
<a href="https://github.com/Prototik/HoloEverywhere" class="btn btn-info">GitHub下载</a>




