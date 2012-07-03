---
layout: post
title: Android初探：简洁版天气预报
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

## log missing
在一个AsyncTask里使用如下语句打印log

	Log.i(ImageTask.class.getSimpleName(), result.getCookie());

但在LogCat里始终看不到该条日志，log去哪儿了？重试了无数次终于发现玄机，原来`result.getCookie()`返回了null，而Log并不输出null日志，无语啊，android调试的一大陷阱！

## 画图
[AChartEngine](http://code.google.com/p/achartengine/)是Android下比较好用的画图工具，支持折线图、柱状图、饼图等等，具体怎么用可参见其docs和demo。这里记录下几个特殊问题：  
**折线图X/Y轴如何自定义刻度？**  
默认情况下，X/Y轴只显示`XYSeries`中添加的数值，double类型，如何显示日期/温度呢？代码如下  

	// XLabels设为0后，不再显示XYSeries中加入的x值
	renderer.setXLabels(0);
	for (Double x : xlabels.keySet()) {
		renderer.addXTextLabel(x, xlabels.get(x));
	}
	// YLabels设为0后，不再显示XYSeries中加入的y值
	renderer.setYLabels(0);
	double ymax = Math.max(daySeries.getMaxY(), nightSeries.getMaxY());
	for (int t = 0; t <= ymax + 2; t += 2) {
		renderer.addYTextLabel(t, t + "℃");
	}

**有些数据点被遮挡**  
默认情况下，X/Y轴只显示到`XYSeries`添加的数值范围，即X轴范围为[XYSeries,getMinX, XYSeries.getMaxX]，Y轴范围为[XYSeries。getMinY, XYSeries.getMaxY]，因此边界点会被遮挡。
增大X/Y轴范围即可完整显示所有数值点，代码如下

	renderer.setXAxisMin(Math.min(daySeries.getMinX(), nightSeries.getMinX()) - 0.5);
	renderer.setXAxisMax(Math.max(daySeries.getMaxX(), nightSeries.getMaxX()) + 0.5);

**为何有些店不现实数值？**  
如果数据点较多，前后间隔较近，就会发现有些店的值不现实了，即使`dayRender.setDisplayChartValues(true);`了。
查看源码发现`XYChart.drawChartValuesText`方法里判断了前后两个点X轴间距大于100像素或者Y轴间距大于100像素时才显示ChartValue，而且这个100不能设置，是hardcode进去的。
通过子类重写这个方法，设置间距为10即可显示全部ChartValue。  

**如何自定义数值点显示的值？**  
默认折线图中的点显示的y轴的值，如何自定义呢？问题也在`XYChart.drawChartValuesText`方法里。可以自定义一个XYSeries子类，在添加数值点时同时加上这个点要显示的label，在drawChartValuesText时再从XYSeries取出label显示即可，代码示例如下

	public class MyXYSeries extends XYSeries {
		private Map<String, String> labels = new HashMap<String, String>();
	
		public synchronized void add(double x, double y, String label) {
			add(x, y);
			labels.put(normalize(x) + seperator + normalize(y), label);
		}
	
		public String getLabel(double x, double y) {
			String label = labels.get(normalize(x) + seperator + normalize(y));
			if (label == null) {
				label = normalize(y);
			}
			return label;
		}
		……
	}
	
	public class MyLineChart extends LineChart {
		private float displayPointDistance = 10f;
		……
		@Override
		protected void drawChartValuesText(Canvas canvas, XYSeries series, SimpleSeriesRenderer renderer, Paint paint,
				float[] points, int seriesIndex, int startIndex) {
			if (points.length > 1) { // there are more than one point
				// record the first point's position
				float distance = displayPointDistance;
				float previousPointX = points[0];
				float previousPointY = points[1];
				for (int k = 0; k < points.length; k += 2) {
					if (k == 2) { // decide whether to display first two points'
									// values or not
						if (Math.abs(points[2] - points[0]) > distance || Math.abs(points[3] - points[1]) > distance) {
							// first point
							drawText(canvas, getLabel(series, series.getX(startIndex), series.getY(startIndex)), points[0],
									points[1] - renderer.getChartValuesSpacing(), paint, 0);
							// second point
							drawText(canvas, getLabel(series, series.getX(startIndex + 1), series.getY(startIndex + 1)),
									points[2], points[3] - renderer.getChartValuesSpacing(), paint, 0);
	
							previousPointX = points[2];
							previousPointY = points[3];
						}
					} else if (k > 2) {
						// compare current point's position with the previous
						// point's, if they are not too close, display
						if (Math.abs(points[k] - previousPointX) > distance
								|| Math.abs(points[k + 1] - previousPointY) > distance) {
							drawText(canvas,
									getLabel(series, series.getX(startIndex + k / 2), series.getY(startIndex + k / 2)),
									points[k], points[k + 1] - renderer.getChartValuesSpacing(), paint, 0);
							previousPointX = points[k];
							previousPointY = points[k + 1];
						}
					}
				}
			} else { // if only one point, display it
				for (int k = 0; k < points.length; k += 2) {
					drawText(canvas, getLabel(series.getY(startIndex + k / 2)), points[k],
							points[k + 1] - renderer.getChartValuesSpacing(), paint, 0);
				}
			}
		}
	
		protected String getLabel(XYSeries series, double x, double y) {
			if (series instanceof MyXYSeries) {
				return ((MyXYSeries) series).getLabel(x, y);
			}
			return getLabel(y);
		}
	
	}


## references
+ [Android DevGuide](http://developer.android.com/guide/index.html)
+ [Android Configurator for M2E Maven Integration for Eclipse](http://rgladwell.github.com/m2e-android/)
+ [Android桌面快捷方式图标生成与删除，使用Intent与launcher交互](http://dingran.iteye.com/blog/779510)
+ [AChartEngine](http://code.google.com/p/achartengine/)

