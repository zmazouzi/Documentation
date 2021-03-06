
---
title: 音频相关
description: 
platform: 音频相关
updatedAt: Mon Dec 17 2018 08:52:55 GMT+0000 (UTC)
---
# 音频相关
### iOS 端集成 H5 游戏音量低

**背景信息：**iOS SDK 2.2.0 集成 H5 游戏；

**问题现象：**用 layabox 自带的引擎播放 WkWebview，主播加入声网的频道，游戏里面的声音很低。该问题仅在 iOS 端发现，Android 端表现正常。

**问题原因：**WKWebView 加载的 H5 是另外一个进程通过 AVAudioSession 播放声音，不是通过 SDK 播放的。

**解决方案：**
`self.agoraEngine setAudioProfile:(AgoraRtc_AudioProfile_Default) scenario:(AgoraRtc_AudioScenario_GameStreaming);`  //选择 `GameStreaming` 模式。

**方案缺点：**

* 采用 `GameStreaming` 模式可能会出现一点回声，理论不影响使用，如有必现或大概率复现的情况可联系技术支持。
* H5 的声音不是通过 SDK 播放的，无法消除回声。

### Android 9 设备上应用退到后台或锁屏后，采集不到声音

**背景信息：** Android 9 系统行为变更。

**问题现象：** Android 9 设备锁屏 1 分钟内，音频无声

**问题原因：** 从 Anroid 官网来看，这是系统强制限制。原文如下：

> **Limited access to sensors in background**
> Android 9 limits the ability for background apps to access user input and sensor data. If your app is running in the background on a device running Android 9, the system applies the following restrictions to your app:
> * Your app cannot access the microphone or camera.
> * Sensors that use the continuous reporting mode, such as accelerometers and gyroscopes, don't receive events.
> * Sensors that use the on-change or one-shot reporting modes don't receive events.
> If your app needs to detect sensor events on devices running Android 9, use a foreground service.


详见 [Android 行为变更](https://developer.android.com/about/versions/pie/android-9.0-changes-all)。


**解决方案：** 目前 Android 官网没有明确说明后台采集声音应如何处理，但使用**前台服务**可以让应用正常工作。

如果 Android 9 设备用户有锁屏后采集音频的需求，可以在锁屏前起一个 Service，并在退出锁屏前终止 Service。你可以参考下面的代码片段进行实现：

- 开启前台服务

	```java
	Intent intent = new Intent(this, MyForegroundService.class);
	intent.setAction(MyForegroundService.ACTION_START_FOREGROUND_SERVICE);
	this.startService(intent);
	```

- 停止前台服务

	```java
	Intent intent = new Intent(this, MyForegroundService.class);
	intent.setAction(MyForegroundService.ACTION_STOP_FOREGROUND_SERVICE);
	stopService(intent);
	```
	
- `MyForegroundService.class` 示例代码

	```java
	package io.agora.tutorials1v1acall;

	import android.app.Notification;
	import android.app.PendingIntent;
	import android.app.Service;
	import android.content.Intent;
	import android.os.IBinder;
	import android.support.v4.app.NotificationCompat;
	import android.util.Log;
	import android.widget.Toast;

	public class MyForegroundService extends Service {
			private static final String TAG_FOREGROUND_SERVICE = "FOREGROUND_SERVICE";

			public static final String ACTION_START_FOREGROUND_SERVICE = "ACTION_START_FOREGROUND_SERVICE";

			public static final String ACTION_STOP_FOREGROUND_SERVICE = "ACTION_STOP_FOREGROUND_SERVICE";

			public MyForegroundService() {
			}

			@Override
			public IBinder onBind(Intent intent) {
					throw new UnsupportedOperationException("Not yet implemented");
			}

			@Override
			public void onCreate() {
					super.onCreate();
					Log.d(TAG_FOREGROUND_SERVICE, "My foreground service onCreate().");
			}

			@Override
			public int onStartCommand(Intent intent, int flags, int startId) {

					Log.d(TAG_FOREGROUND_SERVICE, "onStartCommand " + intent);

					if (intent != null) {
							String action = intent.getAction();

							switch (action) {
									case ACTION_START_FOREGROUND_SERVICE:
											startForegroundService();
											Toast.makeText(getApplicationContext(), "Foreground service is started.", Toast.LENGTH_LONG).show();
											break;
									case ACTION_STOP_FOREGROUND_SERVICE:
											stopForegroundService();
											Toast.makeText(getApplicationContext(), "Foreground service is stopped.", Toast.LENGTH_LONG).show();
											break;
							}
					}
					return super.onStartCommand(intent, flags, startId);
			}

			/* Used to build and start foreground service. */
			private void startForegroundService() {
					Log.d(TAG_FOREGROUND_SERVICE, "Start foreground service.");

					// Create notification default intent.
					Intent intent = new Intent();
					PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);

					// Create notification builder.
					NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "sfdsfds");

					// Make notification show big text.
					NotificationCompat.BigTextStyle bigTextStyle = new NotificationCompat.BigTextStyle();
					bigTextStyle.setBigContentTitle("Music player implemented by foreground service.");
					bigTextStyle.bigText("Android foreground service is a android service which can run in foreground always, it can be controlled by user via notification.");
					// Set big text style.
					builder.setStyle(bigTextStyle);

					builder.setWhen(System.currentTimeMillis());

					// Make the notification max priority.
					builder.setPriority(Notification.PRIORITY_MAX);
					// Make head-up notification.
					builder.setFullScreenIntent(pendingIntent, true);

					// Build the notification.
					Notification notification = builder.build();

					// Start foreground service.
					startForeground(1, notification);
			}

			private void stopForegroundService() {
					Log.d(TAG_FOREGROUND_SERVICE, "Stop foreground service.");

					// Stop foreground service and remove the notification.
					stopForeground(true);

					// Stop the foreground service.
					stopSelf();
			}
	}
	```
