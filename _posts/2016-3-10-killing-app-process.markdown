---
layout: post
title:  "Kill All Androids - Reproducing Memory Pressure Bugs"
date:   2016-03-10 12:00:00
comments: true
categories: Android
---

I just fixed a bug that has been annoying me for a while. It only manifested when when the app was restored from the background, after being in the background for a long time, and would crash the app when the app was restored to the foreground. Turns out it was from me accessing some state inside of a singleton, and that state disappeared after a long time in the background since the OS killed the process due to memory pressure. I was perplexed for a while at why this was only happening after a long time, since I thought that setting "Do Not Keep Activities" in the developer settings would trigger this sort of bug, but Do Not Keep Activities only kills _activities_ once the app is backgrounded, not the entire app process - and the scope of singletons is bound to the entire process, not just the activity. Anyways, the way to recreate a bug like this is to put the app in the background, and run the command `adb shell am kill <package id>` (it has to be in the background, because the kill process command won't work on a foregrounded app). This is much cleaner than my earlier technique of backgrounding the app and then launching dozens of apps until the OS kills my process to reclaim memory!
