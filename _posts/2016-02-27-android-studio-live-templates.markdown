---
layout: post
title:  "Useful Live Templates for Android Development"
date:   2016-02-27 12:00:00
comments: true
categories: Android
---

For each of these, naviate to `Preferences -> Live Templates`, and create a new Live Template. Define the context as 'Java'.

# Static imports for tests

```java
import static org.mockito.Mockito.*;
import static com.google.common.truth.Truth.*;
```

# Static imports for espresso tests

```java
import static android.support.test.espresso.Espresso.*;
import static android.support.test.espresso.matcher.ViewMatchers.*;
import static android.support.test.espresso.assertion.ViewAssertions.*;
```

# Adding an ActivityTestRule for tests

```java
@Rule
public ActivityTestRule<$ACTIVITY$> activityRule = new ActivityTestRule($ACTIVITY$.class);
```
