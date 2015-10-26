---
layout: post
title:  "Remove Noise from Android Studio Logcat Output"
date:   2015-10-26 12:00:00
comments: true
categories: Android
---

Tired of seeing a bunch of system events junking up the Android Studio logcat output? Click on `Android Monitor` on the bottom toolbar, then click `Edit Filter Configuration` in the top right of the popup. Paste the following into the `Log Tag` field:

```
^(?!(WifiMulticast|WifiHW|MtpService|PushClient|Tethering|SensorService|WifiStateMachine|hawaii.hwcomposer|AnyDo|PowerManagerService|Monitor|IconMerger|InputMethodManager|SignalClusterView_dual|StatusBar.NetworkController_dual|LocationManagerService|Provider|SurfaceTextureClient|ImageLoader|dalvikvm|OpenGLRenderer|skia|AbsListView|MediaPlayer|AudioManager|VelocityTracker|Drv|Jpeg|CdpDrv|IspDrv|TpipeDrv|iio|ImgScaler|IMG_MMU|ResMgrDrv|JpgDecComp|JpgDecPipe|mHalJpgDec|PipeMgstatrDrv|mHalJpgParser|jdwp|libEGL|Zygote|Trace|InputEventReceiver|SpannableStringBuilder|IInputConnectionWrapper|MotionRecognitionManager|LoadedApk|Settings|PhoneWindow|Choreographer|v_galz|SensorManager|Sensors|GC|LockPatternUtils|STATUSBAR*|SignalStrength|STATUSBAR-BatteryController|BatteryService|STATUSBAR-PhoneStatusBar|WifiP2pStateTracker|Watchdog|AlarmManager|BatteryStatsImpl|STATUSBAR-Clock)).*$
```

This is a regular expression which filters out any of the Logcat tags matching any of the above patterns. If there are still tags in your logcat you don't want, add them to the above list.

