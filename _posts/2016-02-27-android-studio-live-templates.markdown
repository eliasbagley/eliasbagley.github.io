---
layout: post
title:  "Useful Live Templates for Android Development"
date:   2016-02-27 12:00:00
comments: true
categories: Android
---

For each of these, naviate to `Preferences -> Live Templates`, and create a new Live Template. Define the context as `Java`.

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

```
@Rule
public ActivityTestRule<$ACTIVITY$> activityRule = new ActivityTestRule($ACTIVITY$.class);
```

# Add MAIN, DEFAULT, and LAUNCHER intent filters to quickly switch which activity gets launched for development

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
```

# Quickly add custom annotation

```xml
import java.lang.annotation.Retention;
import javax.inject.Qualifier;

import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Qualifier
@Retention(RUNTIME)
public @interface $FILENAME$ {
}
```

Click `Edit variables` on the right hand side, and enter `fileNameWithoutExtension()` for the `FILENAME` variable, to have the filename prepopulated.
