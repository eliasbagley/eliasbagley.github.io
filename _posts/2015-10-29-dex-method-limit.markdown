---
layout: post
title:  "Overcoming Android Dex Method Limit"
date:   2015-10-29 12:00:00
comments: true
categories: Android
---

Google has officially addressed the 65k method limit in Android 5, and has included a support library for earlier platforms.

#1. Add multiDexEnabled = true in build.gradle

```
android {
    defaultConfig {
        multiDexEnabled = true
    }
}
```

#2. Add multidex library in build.gradle

```
dependencies {
  compile 'com.android.support:multidex:1.0.0'
}
```

#3. Install MultiDex in your Application subclass' attachBaseContext() method

```java
   @Override
   protected void attachBaseContext(Context context) {
       super.attachBaseContext(context);
       MultiDex.install(this);
   }
```
