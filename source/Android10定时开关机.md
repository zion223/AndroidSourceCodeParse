### Android10 定时开关机实现思路	-2020/11/19

Android10 中AlarmManager取消了RTC_POWEROFF_WAKEUP 不支持通过AlarmManager设置关机唤醒设备。  
Android7.1中AlarmManager代码中关于中AlarmManager取消了RTC_POWEROFF_WAKEUP的说明。
``` java
  	/**
     * Alarm time in {@link System#currentTimeMillis System.currentTimeMillis()}
     * (wall clock time in UTC), which will wake up the device when
     * it goes off. And it will power on the devices when it shuts down.
     * Set as 5 to make it be compatible with android_alarm_type.
     * @hide
     */
    public static final int RTC_POWEROFF_WAKEUP = 5;

```


在Android10中的时钟应用在设置了闹钟时间后，支持在关机状态下唤醒设备提醒闹钟，因此跟踪闹钟应用设置闹钟时间的流程，在设置闹钟时间时发送了一个广播。
这个广播由Android10原生应用接收 PowerOffAlarm, 应用路径 /vendor/qcom/proprietary/commonsys/qrdplus/Extension/apps/PowerOffAlarm。
PowerOffAlarm接收org.codeaurora.poweroffalarm.action.SET_ALARM广播，此广播需要系统权限才可以发送，在AndroidManifest.xml中设置shareUserId并且申请权限。
``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	android:shareUserId="android.uid.system">
	<uses-permission android:name="org.codeaurora.permission.POWER_OFF_ALARM">
```
设置后需要将此APK进行系统签名后才可安装。

``` java
//发送广播
Intent powerOnIntent = new Intent("org.codeaurora.poweroffalarm.action.SET_ALARM");
powerOnIntent.setPackage("com.qualcomm.qti.poweroffalarm");
powerOnIntent.putExtra("time", time);
sendBroadcast(powerOnIntent);
```

PowerOffAlarm应用中的PoweroffAlarmBroadcastReceiver接收到广播后，通过PowerOffAlarmUils中的setAlarmToRtc()方法解析时间并且计算出时间间隔后，将时间设置到通过hidl实现的Alarm.cpp中的函数setAlarm()，在这个函数中通过ioctl命令将时间写入到 /dev/rtc0 时钟上。
``` java
	//PowerOffAlarmUtils.java
	public static long setAlarmToRtc(long alarmTime){

		long currentTime = System.currentTimeMillis();
		long alarmInRtc = getAlarmFromRtc();
		long rtcTime = getRtcTime();
		// calculate the alarm to rtc
		long timeDelta = alarmTime - currentTime - MS_IN_ONE_MIN;
		if(timeDelta <= 0){
			Log.d(TAG, "setAlarmToRtc failed: alarm time is in one miunute");
			return FAILURE;
		}
		long alarmTimeToRtc = timeDelta/SEC_TO_MS + rtcTime;
		try{
			IAlarm mProxy = IAlarm.getService(true);
			int ret = mProxy.setAlarm(alarmTimeToRtc);
			if(ret == SUCCESS){
				return alarmTimeToRtc;
			}else{
				return FAILURE;
			}
		}catch{
			return FAILURE;
		}
	}
```
上面代码中的IAlarm是HIDL的形式，具体实现如下

```C++
Return<int32_t> Alarm::setAlarm(int64_t time){
	int fd, rc;
	struct tm alarm_tm;
	struct rtc_wkalrm rtc_alarm;
	time_t alarm_secs = 0;

	ALOGD("alarm hal setAlarm time");
	// #define DEFAULT_RTC_DEV_PATH = "dev/rtc0"
	fd = open(DEFAULT_RTC_DEV_PATH, O_RDONLY);
	if(fd < 0){
		ALOGE("Open rtc dev failed when set alarm!");
		return fd;
	}
	alarm_secs = time;
	gmtime_r(&alarm_secs, &alarm_tm);

	rtc_alarm.time.tm_sec = alarm_tm.tm_sec
	rtc_alarm.time.tm_min = alarm_tm.tm_min
	rtc_alarm.time.tm_hour = alarm_tm.tm_hour
	rtc_alarm.time.tm_mday = alarm_tm.tm_mday
	rtc_alarm.time.tm_mon = alarm_tm.tm_mon
	rtc_alarm.time.tm_year = alarm_tm.tm_year
	rtc_alarm.time.tm_wday = alarm_tm.tm_wday
	rtc_alarm.time.tm_yday = alarm_tm.tm_yday
	rtc_alarm.time.tm_isdst = alarm_tm.tm_isdst

	rtc_alarm.enabled = 1;
	// 设置定时时间
	rc = ioctl(fd, RTC_WKALM_SET, &rtc_alarm);
	close(fd);

	if(rc < 0){
		ALOGE("Set alarm to rtc failed!");
		return rc;
	}

	return 0;
}
```
因此在自己实现定时开关机的思路就是模仿时钟应用，在设置定时开机的时候给PowerOffAlarm应用发送广播,但是需要解决和时钟应用设置的时间冲突的问题。  
取消闹钟同理，发送org.codeaurora.poweroffalarm.action.CANCEL_ALARM即可，发送此广播时需要在extra中带有时间。



