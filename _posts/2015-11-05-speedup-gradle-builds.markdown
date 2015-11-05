---
layout: post
title:  "Speeding up Gradle Builds"
date:   2015-11-05 12:00:00
comments: true
categories: Android
---

I've lost years of my life to slow gradle builds, and I _finally_ found a set of build options which is drastically reducing my builds times. First, set `offline mode` in the gradle preferences. This will make it use the dependencies cached on disk first. I think this might give an exception if you're trying to use a dependency it doesn't have yet, so turn it off, sync, and back on again if this happens. Next, this goes in `build.gradle`:

```
android {
    dexOptions {
        incremental true
        preDexLibraries true
        javaMaxHeapSize "12g"
        threadCount 8
    }
}
```
My 1 minute builds are down to 15 s in a lot of cases, and 3-5 s if nothing changed. * _single tear rolls down cheek_ *
