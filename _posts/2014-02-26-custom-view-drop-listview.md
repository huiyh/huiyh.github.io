---
layout: post
category: android-opensource
title: Android自定义控件-DragSortListview
description: ""
modified: 2014-02-26
tags: [android]
comments: true
share: true

comments: true
---
[drag-sort-listview](https://github.com/bauerca/drag-sort-listview)是一个支持拖拽排序和左右滑动删除功能的自定义ListView，重写了 TouchInterceptor (TI) 类来提供更加优美的拖拽动画效果。包含如下特性
* 完美的拖拽支持 (应该没有视觉问题)
* 在拖动的时候提供更平滑的滚动列表滚动
* 支持每个ListItem高度的多样性
* Public startDrag() and stopDrag() methods.
* 有公开的接口可以自定义拖动的View
DragSortListView 适用于带有任何优先级的列表:收藏夹、播放列表、备忘录等。笔者认为这是目前Android开源实现拖动排序操作最完美的方案。

####一. 如何使用
* (1) 数据重排. 拖拽排序重排ListView底层的数据顺序。由于DSLV 不知道您是如何组织您的数据的，所以重新组织数据必须由您自己通过实现相关的接口来实现。
* (2) 开始/停止拖动. 通过调用DSLV的 startDrag() 和 stopDrag() 函数来启动或者停止拖动操作。 DragSortController这个助手类，提供了所有常用的 开始/停止/删除 拖拽操作功能。
* (3) Floating View(拖动的View). 通过实现 FloatViewManager 接口可以控制 拖动的View 的视觉效果和行为。这样您可以显示任何内容作为 拖动的View，并且可以在拖动过程中更新其位置和显示状态。 DragSortController 助手类已经实现了该接口并提供了一些易用的实现方式。
* 第一条是必须的。如上所述 第二条和第三条可以通过 DragSortController 助手类实现。通过研究示例项目中的代码 可以更加深入的理解上述内容。


####二. XML 布局描述
* DragSortListView 可以和标准的View一样在XML布局文件中声明。 在示例项目中提供了几种演示 。新加的额外属性定义如下，描述的格式如下
* <xml attr名称>: (<datatype 数据类型>, <默认值>) <属性描述>.

### 1.XML 属性

* collapsed_height: (dimension, 1px) 拖动其实位置占位符的高度。不能为0.
* drag_scroll_start: (float, 0.3) 拖动时开始滚动ListView的区域（为DSLV 高度的分数值，在0到1之前）
* max_drag_scroll_speed: (float, 0.5) 默认线性加速的拖动时滚动的最大速度。单位：像素/毫秒。
* float_alpha: (float, 1.0) 拖动View的透明度。取值0到1 ， 1代表不透明。
* slide_shuffle_speed: (float, 0.7) 拖动View下方的View挤走的动画速度。
* drop_animation_duration: (int, 150) 拖动放下时候的动画时间。
* remove_animation_duration: (int, 150) 删除一个ListView的空白区域消失的动画时间。
* track_drag_sort: (bool, false) 调试的选项。
* use_default_controller: (bool, true) DSLV是否创建一个默认的DragSortController 对象，并且设置如下属性的值。如果该属性为false，则如下的属性忽略。
* float_background_color: (color, BLACK) 拖动View的北背景色。默认情况下拖动View是当前拖动的Item的图像缓存。
* drag_handle_id: (id, 0) ListItem中的一个View的资源id(或者ListItem的根view)。这个id用来识别“拖动的手柄”，只有当点击该view的时候才会触发拖动操作。使用默认DragSortController并且支持拖动操作的时候需要设置该属性。
* sort_enabled: (bool, true) 是否启用拖动排序功能（如果您只需要左右滑动删除，则无需启用排序功能）
* drag_start_mode: (enum, “onDown”) 设置启动拖动的手势。
* “onDown”:当用户按下拖动手柄的时候启动拖动操作。 
* “onDrag”: 当用户按下拖动手柄开始拖动的时候启动拖动操作。
* “onLongPress”:在拖动手柄上长按时候启动拖动操作。 
* remove_enabled: (bool, false) 是否启用拖动删除的功能。
* remove_mode: (enum, “flingRight”) 设置启用删除的手势。
* “clickRemove”:点击click_remove_id对应的View来删除。
* “flingRight”: 快速向右滑动。
* “flingLeft”: 快速向左滑动。
* “slideRight”:向左滑动的时候，Floating View会变得透明。透明后释放，删除Item。
* “slideLeft”: 同上，向右滑动。
* click_remove_id: (id, 0) 当 remove_mode="clickRemove"并且remove_enabled="true"时候指定的删除一个Item的View。DragSortController使用了该属性。

####三. Listeners
DragSortListView 是个 ListView，因此需要一个 ListAdapter 来提供每个List Item。拖动排序是通过DSLV中的一些接口添加到ListAdapter的条目中的。可以有两种方式来设置DSLV 的各种接口:
*(1)通过set*Listener() 函数来设置
*(2)让自定义的 ListAdapter 实现这些接口，当调用 DragSortListView.setAdapter()函数时候，会检测实现的接口并设置 set*Listener()。

###1.DragSortListView.DropListener
* DropListener接口有一个回调函数:
   public void drop(int from, int to);
* 该函数在拖动排序完成的时候调用；例如 拖动View被放下了。参数 from 是ListView开始拖动的位置，to 是被放下的位置。该接口非常重要，否则DSLV无法完成拖动操作。
* 如果 DSLV 对象的 android:choiceMode 不是 "none"，并且您的 ListAdapter 没有稳定的ID，您则必须在 drop(from, to)中调用 DragSortListView.moveCheckState(int from, int to)。

###2.DragSortListView.RemoveListener
* DSLV 提供了手势来删除拖动的View。 完成一个删除手势后，DSLV 调用RemoveListener 接口的函数:
   public void remove(int which);
* 在ListAdapter中位置为 which 的条目需要被删除。是否真的删除或者只是标记删除由您自己实现。
* 在拖动操作之外您也可以删除条目。在任何时候都可以调用 DragSortListView.removeItem(int position) .
* 如果 DSLV 对象的 android:choiceMode 不是 "none"，并且您的 ListAdapter 没有稳定的ID，您则必须在 remove(which)中调用 DragSortListView.moveCheckState(int from, int to)

###3.DragSortListView.DragListener
* DragListener 中的回调接口为
   public void drag(int from, int to);
* 当拖动的View位于一个新条目上方的时候 会调用该函数。 to 是当前潜在的放下位置， from是前一个位置。  TI 实现中提供了这个接口，目前还没使用场景举例。

###4.DragSortListView.DragSortListener
这只是一个继承了上面3个接口的便利接口。

####四.FloatViewManager
* 该接口用来创建、更新和销毁拖动的View。通过函数 setFloatViewManager() 来设置。 SimpleFloatViewManager演示了其用法，该类是个简单的实现。用拖动List Item的快照实现一个拖动的View。
* 如果您需要实现不同的拖动View则可以提供自己的FloatViewManager。 在 onCreateFloatView() 函数中，确保返回的View有个固定的尺寸 (不要使用 MATCH_PARENT! 虽然对于宽度来说用 MATCH_PARENT 也是可以的). DSLV 会根据您拖动View上的ViewGroup.LayoutParams 参数来测量该View。如果没有LayoutParams参数则会使用 WRAP_CONTENT 和 MATCH_PARENT 来测量器高度和宽度。

####五.Drag start/stop
* 在DragSortListView 0.3.0以后，拖动开始和停止完全取决于您。在DSLV中调用startDrag() 或者 stopDrag() 即可。需要注意的是，如果在调用startDrag() 函数的时候没有Touch事件，则拖动操作不会启动。在通常情况下您都无需关系开始拖动的操作，DragSortController 已经帮助你做好了。

###DragSortController
* DragSortController 类实现了一些常用的拖拽操作和删除List item的操作。该类实现了 View.OnTouchListener 接口来监听点击事件。同时也实现了FloatViewManager 接口。如果您在XML文件中没有使用默认的DragSortController，则您需要调用DSLV对象的 setFloatViewManager() 和 setOnTouchListener() 函数来设置一个DragSortController 。
* DragSortController的默认行为认为List Item启用了拖动操作并且具有一个“拖动手柄”View，在DragSortController 中设置了“拖动手柄”View的资源id(如果 use_default_controller 是 true的话，拖动手柄的ID可以在XML文件中设置)。如果点击事件发生在一个List Item中的拖动手柄View上并且检测到了启动拖动操作的手势，则会启动一个拖动操作。

#### 更多信息参考API
* [API](http://bauerca.github.com/drag-sort-listview)
