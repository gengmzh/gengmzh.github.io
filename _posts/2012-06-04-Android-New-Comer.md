---
layout: post
title: 简洁版天气预报：my first android app
category : Android
tags : [Android]
---
{% include JB/setup %}

`Android New Comer`  

[android developers](http://developer.android.com/index.html)，任何Android开发问题都可以在这里找到解答，本文只是简单记录我的第一个安卓app开发过程中遇到的问题。

## 准备开发环境
**安装SDK**  
在[Android SDK](http://developer.android.com/sdk/index.html)上下载SDK后，直接解压到本地目录。  
然后点击打开`SDK Manager.exe`，选中需要的版本点击install进行更新。  

**安装ADT**  
update site： `ADT - https://dl-ssl.google.com/android/eclipse/`  
使用Eclipse的Install New Software安装即可。  

至此开发环境即搭建完成。 

## 使用Maven创建Android Project
参照[Android Configurator for M2E Maven Integration for Eclipse](http://rgladwell.github.com/m2e-android/)一文，简单可行。

## 如何创建桌面快捷方式
网上倒是能直接搜到代码，例如[Android桌面快捷方式图标生成与删除，使用Intent与launcher交互](http://dingran.iteye.com/blog/779510)一文。  
但是这些代码放哪呢，放在app的启动类里每次都会发送创建快捷方式的广播，最好能在安装应用时发送这个广播。接着在stackoverflow上搜到[How to add app's shortcut to the home screen](http://stackoverflow.com/questions/6988511/how-to-add-apps-shortcut-to-the-home-screen)。
看到评论`Do. Not. Automatically. Create. App. Shortcuts.`，I agree了，还是别污染用户的桌面了~~

## SQLITE优化
有3000条中国城市信息，就是code、name、parent及其简单，在app第一次启动时初始化到SQLITE里，最初版本的代码如下：

	//示例啊，源码是从配置文件里读取城市信息
	City city=null;
	while ((city=getNextCity()) != null) {
		ContentValues values = new ContentValues();
		values.put(Weather.City.CODE, city.getId());
		values.put(Weather.City.NAME, city.getName());
		values.put(Weather.City.PARENT, city.getParent());
		contentResolver.insert(Weather.City.CONTENT_URI, values);
	}

巨慢无比，3000条初始完大概需要95s，只能优化了。  
首先是业务逻辑上的，最初`contentResolver.insert`里先判断city是否存在，存在则update否则insert，由于是初始化所以这个判断没有必要，改为直接insert，效果很大改进，但仍然需要10来秒。  
接着优化，发现ContentProvider还有bulkInsert接口，于是将单个insert改为批量insert；又发现SQLITE也可以对sql进行预编译，于是乎最终代码长成下面模样了：

	public int bulkInsert(Uri uri, ContentValues[] values) {
		switch (URI_MATCHER.match(uri)) {
		case Weather.City.TYPE:
			int result = 0;
			SQLiteDatabase sqlite = databaseSupport.getWritableDatabase();
			String sql = "insert into " + Weather.City.TABLE_NAME + "(" + Weather.City.CODE + "," + Weather.City.NAME + ","
					+ Weather.City.PARENT + ") values(?,?,?) ";
			SQLiteStatement stat = sqlite.compileStatement(sql);
			sqlite.beginTransaction();
			try {
				for (ContentValues values : valuesArray) {
					stat.bindString(1, values.getAsString(Weather.City.CODE));
					stat.bindString(2, values.getAsString(Weather.City.NAME));
					String p = values.getAsString(Weather.City.PARENT);
					if (p != null && p.length() > 0) {
						stat.bindString(3, p);
					} else {
						stat.bindNull(3);
					}
					stat.executeInsert();
					result++;
				}
				sqlite.setTransactionSuccessful();
			} finally {
				sqlite.endTransaction();
			}
			return result;
		default:
			throw new IllegalArgumentException("unknown Uri " + uri);
		}
	}

批量insert3000条城市信息只需要5s左右。  
网上有说SQLITE insert5000条信息时只需要0.几秒，经过实际测试在我的安卓上没有这么高效。

## TabHost疑惑

+ TabWidget存放的是标签，FrameLayout存放的是内容，此外还可视需要添加更多的组件，如下：

	<?xml version="1.0" encoding="utf-8"?>
	<TabHost xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@android:id/tabhost"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent" >
	
	    <LinearLayout
	        android:layout_width="fill_parent"
	        android:layout_height="fill_parent"
	        android:orientation="vertical"
	        android:padding="0dp" >
	
	        <TabWidget
	            android:id="@android:id/tabs"
	            android:layout_width="fill_parent"
	            android:layout_height="wrap_content"
	            android:padding="0dp" />
	
	        <RelativeLayout
	            android:layout_width="fill_parent"
	            android:layout_height="50dp"
	            android:orientation="horizontal"
	            android:padding="3dp" >
	
	            <TextView
	                android:id="@+id/city"
	                android:layout_width="wrap_content"
	                android:layout_height="wrap_content"
	                android:hint="@string/city_hint"
	                android:textSize="28dp"
	                android:textStyle="bold" />
	
	            <ProgressBar
	                android:id="@+id/progressBar"
	                style="?android:attr/progressBarStyleSmall"
	                android:layout_width="wrap_content"
	                android:layout_height="wrap_content"
	                android:layout_alignParentRight="true"
	                android:layout_alignTop="@id/city"
	                android:max="100" />
	        </RelativeLayout>
	
	        <FrameLayout
	            android:id="@android:id/tabcontent"
	            android:layout_width="fill_parent"
	            android:layout_height="fill_parent"
	            android:padding="0dp" />
	    </LinearLayout>
	
	</TabHost>

+ 在每个标签对应的Activity里如何获取到`@id/city`呢？简单：

	getParent().findViewById(R.id.city)

## AsyncTask应用
通过AsyncTask可以在后台线程里执行一些费时的操作，并把最终结果展现在UI里，简单易用。  例如我的天气预报app里就有：

	class RealtimeTask extends AsyncTask<String, Integer, Cursor> {

		@Override
		protected Cursor doInBackground(String... params) {
			onProgressUpdate(0);
			……
			//联网获取天气预报信息
			Cursor realtime = ……
			……
			onProgressUpdate(60);
			return realtime;
		}

		@Override
		protected void onProgressUpdate(Integer... values) {
			super.onProgressUpdate(values);
			……
			//更新进度条
			progressBar.setProgress(progress);
		}

		@Override
		protected void onPostExecute(Cursor realtime) {
			super.onPostExecute(realtime);
			onProgressUpdate(80);
			// 显示天气预报
			if (realtime != null && realtime.moveToFirst()) {
				// temperature
				String text = realtime.getString(realtime.getColumnIndex(Weather.RealtimeWeather.TEMPERATURE));
				TextView view = (TextView) findViewById(R.id.temperatue);
				view.setText(text);
				// wind
				view = (TextView) findViewById(R.id.wind);
				text = realtime.getString(realtime.getColumnIndex(Weather.RealtimeWeather.WINDDIRECTION));
				view.setText(text);
				text = realtime.getString(realtime.getColumnIndex(Weather.RealtimeWeather.WINDFORCE));
				if (text != null && !text.equals(view.getText())) {
					view.setText(view.getText() + text);
				}
				// humidity
				text = realtime.getString(realtime.getColumnIndex(Weather.RealtimeWeather.HUMIDITY));
				view = (TextView) findViewById(R.id.humidity);
				view.setText("湿度" + text);
				// updateTime
				text = realtime.getString(realtime.getColumnIndex(Weather.RealtimeWeather.TIME));
				view = (TextView) findViewById(R.id.updateTime);
				view.setText(DATE_FORMAT.format(new Date()) + " " + text + "更新");
			} else {
				Log.e(RealtimeActivity.class.getName(), "can't get realtime weather");
			}
			……
			onProgressUpdate(100);
		}
	}

需要注意的是，对UI的更新一定要在onPostExecute里进行，否则会报权限错误。


## references
+ [Android DevGuide](http://developer.android.com/guide/index.html)
+ [Android Configurator for M2E Maven Integration for Eclipse](http://rgladwell.github.com/m2e-android/)
+ [Android桌面快捷方式图标生成与删除，使用Intent与launcher交互](http://dingran.iteye.com/blog/779510)

