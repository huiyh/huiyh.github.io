---
layout: post
category: android-development
title: Android DownloadManager 
description: ""
modified: 2014-03-01
tags: [android]
comments: true
share: true
comments: true
---
[DownloadManager](http://developer.android.com/reference/android/app/DownloadManager.html)是系统开放给第三方应用使用的类，包含两个静态内部类DownloadManager.Query和DownloadManager.Request。

* [DownloadManager.Request](http://developer.android.com/reference/android/app/DownloadManager.Query.html)用来请求一个下载
* [DownloadManager.Query](http://developer.android.com/reference/android/app/DownloadManager.Request.html) 用来查询下载信息

DownloadManager主要提供了一下主要方法

* enqueue(Request request)：执行下载，返回downloadId,downloadId可用于查询下载信息。
* remove(long ids)：删除下载，若下载中取消下载。会同时删除下载文件和记录。
* query(Query query)查询下载信息
* getMaxBytesOverMobile(Context context)通过移动网络下载的最大字节数
* getMimeTypeForDownloadedFile(long id)得到下载的mineType

通过查看代码我们可以发现还有个CursorTranslator私有静态内部类。这个类主要对Query做了一层代理。将DownloadProvider和DownloadManager之间做个映射。将DownloadProvider中的十几种状态对应到了DownloadManager中的五种状态，DownloadProvider中的失败、暂停原因转换为了DownloadManager的原因。

###使用DownloadManager的一般步骤

####1.调用DownloadManager.Request开始下载

在开始之前，在AndroidManifest.xml中添加网络访问权限和sdcard写入权限。

{% highlight xml %}

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

{% endhighlight %}

接着，调用DownloadManager.Request开始下载

{% highlight java %}

DownloadManager downloadManager = (DownloadManager)getSystemService(DOWNLOAD_SERVICE);
String url = "http://v.yingshibao.chuanke.com/cet4/CET4_reading_video/1_zongshu/001_zongshu.mp4";
DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
request.setDestinationInExternalPublicDir("itbox", "zongshu.mp4");
long downloadId = downloadManager.enqueue(request);

{% endhighlight %}

[DownloadManager.Request](http://developer.android.com/reference/android/app/DownloadManager.Query.html)一些常用方法：

* setDestinationInExternalFilesDir 设置文件下载路径
* allowScanningByMediaScanner() 表示允许MediaScanner扫描到这个文件夹，默认不允许。
* setTitle()设置下载中通知栏提示的标题
* setDescription()设置下载中通知栏提示的介绍。
* setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED)表示下载进行中和下载完成的通知栏是否显示。默认只显示下载中通知。VISIBILITY_VISIBLE_NOTIFY_COMPLETED表示下载完成后显示通知栏提示。VISIBILITY_HIDDEN表示不显示任何通知栏提示，这个需要在AndroidMainfest中添加权限android.permission.DOWNLOAD_WITHOUT_NOTIFICATION.
* request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI) 表示下载允许的网络类型，默认在任何网络下都允许下载。有NETWORK_MOBILE、NETWORK_WIFI、NETWORK_BLUETOOTH三种及其组合可供选择。如果只允许wifi下载，而当前网络为3g，则下载会等待。
* request.setAllowedOverRoaming(boolean allow)移动网络情况下是否允许漫游。
* request.setMimeType() 设置下载文件的mineType。因为下载管理Ui中点击某个已下载完成文件及下载完成点击通知栏提示都会根据mimeType去打开文件。
* request.addRequestHeader(String header, String value)添加请求下载的网络链接的http头，比如User-Agent，gzip压缩等

####2.下载进度查询

DownloadManager下载过程中，会将下载的数据和下载的状态插入ContentProvider中，所以我们可以通过注册一个ContentObserver，通过ContentObserver不断获取数据，对UI进行更新。

我们主要调用DownloadManager.Query()进行查询，DownloadManager.Query为下载管理对外开放的信息查询类，主要包括以下方法：
* setFilterById(long… ids)根据下载id进行过滤
* setFilterByStatus(int flags)根据下载状态进行过滤
* setOnlyIncludeVisibleInDownloadsUi(boolean value)根据是否在download ui中可见进行过滤。
* orderBy(String column, int direction)根据列进行排序，不过目前仅支持DownloadManager.COLUMN_LAST_MODIFIED_TIMESTAMP和DownloadManager.COLUMN_TOTAL_SIZE_BYTES排序。


{% highlight java %}

// 观察
	class DownloadChangeObserver extends ContentObserver {
		private Handler handler;
		private long downloadId;

		public DownloadChangeObserver(Handler handler, long downloadId) {
			super(handler);
			this.handler = handler;
			this.downloadId = downloadId;
		}

		@Override
		public void onChange(boolean selfChange) {
			updateView(handler, downloadId);
		}

	}

	mContext.getContentResolver().registerContentObserver(
				Uri.parse("content://downloads/my_downloads"), true,
				new DownloadChangeObserver(handler, downloadId));

{% endhighlight %}

注册ContentResolver

{% highlight java %}

	mContext.getContentResolver().registerContentObserver(
				Uri.parse("content://downloads/my_downloads"), true,
				new DownloadChangeObserver(handler, downloadId));

{% endhighlight %}
updateView()方法，获取进度，通过handle发送消息，更新UI

{% highlight java %}

public void updateView(Handler handler, long downloadId) {
		// 获取状态和字节
		int[] bytesAndStatus = getBytesAndStatus(downloadId);
		//
		handler.sendMessage(handler.obtainMessage(0, bytesAndStatus[0],
				bytesAndStatus[1], bytesAndStatus[2]));
}
{% endhighlight %}

getBytesAndStatus(long downloadId)方法用于获取数据和状态

{% highlight java %}
public int[] getBytesAndStatus(long downloadId) {
		int[] bytesAndStatus = new int[] { -1, -1, 0 };
		DownloadManager.Query query = new DownloadManager.Query()
				.setFilterById(downloadId);
		Cursor c = null;
		try {
			c = downloadManager.query(query);
			if (c != null && c.moveToFirst()) {
				// 当前下载的字节
				bytesAndStatus[0] = c
						.getInt(c
								.getColumnIndexOrThrow(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR));
				// 总字节数
				bytesAndStatus[1] = c
						.getInt(c
								.getColumnIndexOrThrow(DownloadManager.COLUMN_TOTAL_SIZE_BYTES));
				// 状态
				bytesAndStatus[2] = c.getInt(c
						.getColumnIndex(DownloadManager.COLUMN_STATUS));
			}
		} finally {
			if (c != null) {
				c.close();
			}
		}
		return bytesAndStatus;
	}

{% endhighlight %}

handle接收到消息，更新UI

{% highlight java %}

private class MyHandler extends Handler {
		private ViewHolder holder;

		public MyHandler(ViewHolder holder) {
			// TODO Auto-generated constructor stub
			this.holder = holder;
		}

		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);

			switch (msg.what) {
			case 0:
				int status = (Integer) msg.obj;
				if (isDownloading(status)) {
					//
					holder.progress.setVisibility(View.VISIBLE);
					holder.progress.setMax(0);
					holder.progress.setProgress(0);
					holder.download.setText("暂停");
					holder.downloadSize.setVisibility(View.VISIBLE);
					holder.downloadPrecent.setVisibility(View.VISIBLE);
					if (msg.arg2 < 0) {
						holder.progress.setIndeterminate(true);
						holder.downloadSize.setText("0M/0M");
					} else {
						holder.progress.setIndeterminate(false);
						holder.progress.setMax(msg.arg2);
						holder.progress.setProgress(msg.arg1);
						holder.downloadPrecent.setText(getPercent(msg.arg1,
								msg.arg2));
						//
						holder.downloadSize.setText(getSize(msg.arg1) + "/"
								+ getSize(msg.arg2));
					}
				} else {
					// 停止下载
					holder.progress.setVisibility(View.GONE);
					holder.progress.setMax(0);
					holder.progress.setProgress(0);
					holder.download.setText("下载");
					holder.downloadSize.setVisibility(View.GONE);
					holder.downloadPrecent.setVisibility(View.GONE);
					if (status == DownloadManager.STATUS_FAILED) {

					} else if (status == DownloadManager.STATUS_SUCCESSFUL) {

					} else {

					}
				}
				break;
			}
		}
	}

{% endhighlight %}


####3.下载成功监听
下载完成后，下载管理会发出DownloadManager.ACTION_DOWNLOAD_COMPLETE这个广播，并传递downloadId作为参数。通过接受广播我们可以打开对下载完成的内容进行操作。

####4.响应通知栏点击

* 下载中通知栏点击

点击下载中通知栏提示，系统会对下载的应用单独发送Action为DownloadManager.ACTION_NOTIFICATION_CLICKED广播。intent.getData为content://downloads/all_downloads/29669，最后一位为downloadId。如果同时下载多个应用，intent会包含DownloadManager.EXTRA_NOTIFICATION_CLICK_DOWNLOAD_IDS这个key，表示下载的的downloadId数组。

* 响应下载完成通知栏点击

下载完后会调用下面代码进行处理，从中我们可以发现系统会调用View action根据mimeType去查询。所以可以利用我们在介绍的DownloadManager.Request的setMimeType函数。

这样我们就可以利用DownloadManager进行下载操作，不过DownloadManager没有暂停操作。

<div markdown="0"><a href="https://github.com/malinkang/DownloadManagerSample" class="btn btn-info">示例下载</a></div>

###扩展阅读

[Android系统下载管理DownloadManager功能介绍及使用示例](http://www.trinea.cn/android/android-downloadmanager/)


[Android下载管理DownloadManager功能扩展和bug修改](http://www.trinea.cn/android/android-downloadmanager-pro/)




