---
layout: post
title:  "Android Style Guide"
date:   2015-10-27 12:00:00
comments: true
categories: android
---

# Android Studio

- Always keep Android Studio up to date. Coordinate with the team when updating major versions.

- Suggestion: Hide the navigation bar. `View -> Uncheck navigation bar`

- Suggestion: Hide the tabs (Double tap shift for search instead of relying on tabs).

- Suggestion: Learn the keyboard shortcuts

- Use `//region description` to mark the beginning of a local section of code. Use `//endregion` to close it. This is like `//MARK: description` in iOS land.

Android studio understands these markers and gives you the ability to collapse code inside of them.

# Essential Android Studio Hotkeys

- CMD CTRL UPARROW - toggle between Activites/Fragments and layout files.

- CMD SHIFT O - open quickly

- CMD SHIFT A - searches for editor actions. Useful if you forget the hotkey for something - you can just fuzzy search by name.

# Android Studio Plugins

- Lombok

- Otto plugin, if you're using the Otto library

- ADB Idea

# Project Structure

- Group classes by type of object, not by feature.

All activities go in `controllers/activities`, all fragments go in `controllers/fragments`

All models go in `models` folder

All data related stuff goes in `data` folder

All network related stuff goes in `network`. Add subfolders for `network/api`, `network/servicerequest`, `network/serviceresponse`, `network/managers`


- Use Android Studio's default package and folder structure

Put code into a `release/` subfolder if it will only ever be used for release

Release mode can override classes in the main folder. Do this if you have classes with methods that will change depending on which build flavor you're using

Put code into a `debug/` subfolder if it will ever only be used for debug builds

Everything else goes in the main subfolder

Code in `release` or `debug` will only be included in the APK if the app is being build for that mode.


# Android testing

- Use `Do Not Keep Activites` setting to discover `saveInstanceState` and `restoreInstanceState` bugs

- Use `adb shell am kill app-id` to force process death to test restoring state after process death

- Test with Espresso, Truth, JUnit4, and Mockito

- Test UI on variety of screen sizes and API levels (Genymotion helps here)

- Use Genymotion instead of the default Emulator


# Libraries

Java needs all the help it can get. Use these excellent 3rd party libs to make it suck less.

Essential:

- Dagger

- Retrofit

- RxJava

- Retrolambda

- SqlBrite

- Leak Canary

- Butterknife

- Google Support libs

Nice to have:

- Picasso

- Lombok

- RxBindings

- Timber. Use instead of `Log.d()`


Situational:

- Otto. Otto is an event bus - use where extreme decoupling is needed. Use the Otto plugin to prevent frustration. Don't overuse Otto.

# Gradle style

- TODO .staging, .debug etc

- TODO structure of build.gradle

- Add all dependencies through gradle. Don't add standalone jars into the project unless necessary

- Protip: turn on Offline Mode in settings to prevent Gradle from pinging the jar repository on every build. This will decrease build times slightly.

- Split out a large .gradle file into smaller reusable build files that can be included in the main build script

- If multiple dependencies share the same version, pull them out into an `ext{}` block and use String interpolation to resolve the version.

Example:

```
ext {
    supportLibVersion = '23.0.1'
}

dependencies {
    compile "com.android.support:appcompat-v7:${supportLibVersion}"
    compile "com.android.support:design:${supportLibVersion}"
    compile "com.android.support:support-annotations:${supportLibVersion}"
    compile "com.android.support:support-v4:${supportLibVersion}"
}

```


- Set the app's version in `build.gradle`, NOT in `AndroidManifest.xml`. Put this information right at the top of the file for easy access

- Put `dependency{}` and `compile` statements in the app's `build.gradle`, NOT in the top level project `build.gradle`

- Put keystore information directly in `build.gradle`

-  Avoid using + to specify version in the dependencies block. Use exact version numbers instead.

Using + can lead to build reproducibility problems down the road.

YES:
`compile 'com.google.code.gson:gson:2.2.3'`

NO:
`compile 'com.google.code.gson:gson:2.2.+'`



## Android Style

- Integrate the Debug Drawer into the project!

- Use the support versions of Android components, rather than the version directly bundled with the platform (support.v4.app.Fragment, etc)

- Going forward, prefer to use the new `RecyclerView` for lists over `ListView`

- Don't litter activities with intent setup code. Have each activity instead present a `public static show(Activity activity)` method

- Prefer Android specific classes like `ArrayMap` over `HashMap` for better performance on mobile devices

- Prefer Activities over Fragments, unless Fragments are necessary. Activities have a simpler lifecycle, and can be easily refactored into a Fragment if necessary.

- Use ButterKnife for injecting views

YES:

```java
@InjectView(R.id.my_view) TextView myView;
```

NO:

```
this.myView = (TextView)findViewById(R.id.my_view);
```

- Use ButterKnife `@OnClick` annotations instead of setting an `OnClickListener`


YES:

```java
@OnClick(R.id.my_button)
public void myButtonClicked(Button b) {
}
```

NO:

```java
this.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        myButtonClicked();
    }
});
```


-  Order of Android Acitivty callbacks match the lifecycle

```java
public class MainActivity extends Activity {
    //Order matches Activity lifecycle
    @Override
    public void onCreate() {}

    @Override
    public void onStart() {}

    @Override
    public void onStop() {}

    @Override
    public void onResume() {}

    @Override
    public void onPause() {}

    @Override
    public void onDestory() {}
}
```

- Parameter ordering - if a `Context` object is being passed in, it comes first. If a `Callback` object is being passed in, it comes last.

```java
public void getUser(Context context, Callback<User> callback) {
  ...
}
```

## Java Style

- Use retrolambda and lambda syntax for single method interfaces

```java
articleStore.getArticle(id)
        .subscribe(article -> {
            doSomethingWithArticle(article);
        });
```

- Use primitive types over Object types when possible. This avoids allocations and memory issues on constrained mobile devices.

- Don't prefix your instance variables with any nonsense. mFriends don't let sFriends use hungarian notation. http://jakewharton.com/just-say-no-to-hungarian-notation/

GOOD:
```java
public class MyClass {
  public String variable;
}
```

BAD:

```java
public class MyClass {
  public String mVariable;
}
```

- Open braces on the same line

```java
public static void main(String[] args) {
}
```

- Flatten uninteresting inner methods

Some callbacks in anonymous classes, like  `onDown()` in GestureDetector, are just boilerplate. Flatten it to make it visually stay out of the way.

```java
    private void setupGestureDetector() {
        this.gestureDetector = new GestureDetectorCompat(this, new GestureDetector.SimpleOnGestureListener() {
            @Override public boolean onDown(MotionEvent e) { return true; }

            @Override
            public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
                // Interesting stuff
                foo()
                bar()
            }
        });
    }

```

- Consider extracting methods from Overriden anonymous inner classes if they are doing too much work

Example:

```java
new Callback() {
  @Override
  public void success(Response response) {
    ...
  }

  @Override
  public void failure(RetrofitError error) {
    MyError e = parseErrorBody(error)
  }

  private void parseErrorBody(RetrofitError error) {
    ...
  }
}
```



- Put constants into a Constants.java class

- Prefer interface types over implementation types

YES:

```java
public List<Integer> getList() {
}
```

NO:

```java
public ArrayList<Integer> getList() {
}
```

- Use type inference for generics

YES:

`List<Integer> list = new ArrayList<>();`

NO:

`List<Integer> list = new ArrayList<Integer>();`


- Add static utility code to a Util class. Break it out into logical subgroups, like `StringUtils.java`, `ImageUtils.java` etc

- Make these Util classes have a private constructor to prevent anyone from ever instantiating them


- Method annotations should wrap (Android Studio Default)

YES:

```java
@Override
public void myMethod() {
}
```

NO:
```java
@Override public void myMethod() {
}
```

- Field annotations should not wrap. Change in Android Setudio settings: set field annotations to `do not wrap`

YES:
```java
class MyClass {
  @Inject Manager myManager;
}

```

NO:
```java
class MyClass {
  @Inject
  Manager myManager;
}
```

`Preferences -> Editor -> Code Style -> Java -> Wrapping and Braces tab -> Field Annotations`


- Don't swallow exceptions

```java
try {
  someDangerousCode();
} catch (Exception e) {
  // lol yolo
}
```

- If find that you have a class that has multiple convenience constructors to aid in initializing optional variables, use the Builder pattern.

YES:
```java
  return Person.builder()
    .setFirstName("Elias")
    .setLastName("Bagley")
    .setAge(27)
    .setOccupation("Engineer")
    .setCatchphrase("Hello world!")
    .build();
```

- Create a custom annotation rather than using `@Named`

YES:
```
@Qualifier
@Retention(RUNTIME)
public @interface MyAnnotation {
}
```

NO:
```
@Named("MyAnnotation")
```

This avoids String typing.


- TODO: Class member ordering (constants, fields, constructors, override methods and callbacks, public public methods, private methods, inner classes or interfaces

# General Style

- Align fields and variables into columns

In Android Studio, navigate to:

```
Preferences | Editor | Code Style | Java | Wrapping and Braces | Group declarations | Align fields in columns
```

- Always have CI setup. Do this at the beginning of the project!

- Add a script to the project root to initiate a CI build, so you can release a build with a single command.


# Naming Conventions

- Don't suffix model names with Model. It's redundant

YES:
`User user = new User();`

NO:
`UserModel user = new UserModel();`

- Use camelCase for variables

- public static final int CONSTANTS\_SHOULD\_BE\_IN\_SHOUTING\_SNAKE\_CASE = 42

- Activity classes should be suffixed with `Activity` - `MainActivity.java`

- Fragment classes should be suffixed with `Fragment` - `TestFragment.java`

- Adapter classes should be suffixed with `Adapter` - `MyAdapter.java`

- Dagger modules should be suffixed with `Module` - `AppModule.java`

- Util classes should be suffixed with `Utils` - `StringUtils.java`

- Retrofit API interfaces should be suffixed with `API` - `UserAPI.java`

# Architecture Style

- Use Dagger to provide dependencies. Split out dagger modules into logical subgroupings, such as AppModule, UIModule, DataModule, NetworkModule

- Use Lombok for data classes to fight Java's verbosity

before:

```java
public class SocialAccessToken {
    String token;
    String secret;

    public SocialAccessToken(String token, String secret) {
        this.token = token;
        this.secret = secret;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public void setSecret(String secret) {
        this.secret = secret;
    }

    public String getToken() {
        return this.token;
    }

    public String getSecret() {
        return this.secret;
    }
}
```

after:

```java
@Data
public class SocialAccessToken {
    String token;
    String secret;
}
```

- Subclass all app objects from a base Object. You can add useful helper code here, like performing injection etc. Java doesn't have protocol extensions, so this is the next best thing.

    Subclass models from BaseModel

    Subclass custom views from BaseView

    Subclass activites from BaseActivity

    Subclass fragments from BaseFragment


## Network Layer Architecture

- Have a ServiceRequest subclass for every request

- Have a ServiceResponse subclass for every response

- Put Retrofit API interfaces in a separate class

- Put all Retrofit API interfaces together in a single folder

# UI/Layout Style

- Follow Material design

- Use `match_parent` instead of `fill_parent` in layouts

They do the same thing, but `match_parent` is the new version, `fill_parent` has been deprecated.

- Put trailing /> in XML on a separate line

YES:

```xml
<Button
    android:id="@+id/activity_login_twitter"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Login with Twitter"
    />
```

NO:

```xml
<Button
    android:id="@+id/activity_login_twitter"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Login with Twitter"/>
```

This makes it easy to add new properties at the end of the property list.

- By default, prefer `FrameLayout` (or `CoordinatorLayout`) as the parent view group. It is more flexible than `LinearLayout`, and more performant than `RelativeLayout`

`RelativeLayout` requires two layout passes to correctly lay out subviews, which is can get slow. It's especially bad when you have a `RelativeLayout` nested inside another `RelativeLayout`, in which case you'll have 4 layout passes!


- Follow the Android Studio default convention for naming layout and view xml files

`view_my_button.xml`

`layout_main_activity.xml`

Android only has a single resources folder, and subfolers aren't allowed. Naming them this way visually groups related xml files together.



- Reference all custom colors from a colors.xml class. Don't hard code colors into XML

_colors.xml_
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="white">#FFFFFF</color>
    <color name="black">#000000</color>
</resources>
```

- Reference all strings from `strings.xml`. Don't hard code strings into views or layout files!

- Reference constants (margins, font sizes) from dimens.xml

example:

_dimens.xml_

```xml
<resources>
    <dimen name="action_bar_text_size">20sp</dimen>
    <dimen name="action_bar_icon_size">28sp</dimen>

    <dimen name="very_large_text_size">40sp</dimen>
    <dimen name="larger_text_size">24sp</dimen>
    <dimen name="large_text_size">20sp</dimen>
    <dimen name="medium_text_size">16sp</dimen>
    <dimen name="medium_small_text_size">14sp</dimen>
    <dimen name="small_text_size">12sp</dimen>
    <dimen name="very_small_text_size">10sp</dimen>

    <!-- Default screen margins, per the Android Design guidelines. -->
    <dimen name="activity_horizontal_margin">16dp</dimen>
    <dimen name="activity_vertical_margin">16dp</dimen>

</resources>
```

- Prefer using `layout_marginStart` and `layout_marginEnd` over `layout_marginLeft` and `layout_marginRight`. For a left-to-right layout, start == left, and end == right, but for a right to left layout, start == right and end == left. Only if min API level >= 17.

- Use `styles.xml` rather than specifying lots of duplicated attributes in xml files. This doesn't need to be applied religiously - use it when you find yourself duplicating lots of attributes.

- Always pair custom view xml with a custom view subclass. Have `populate` methods on the subclass to bind model to view


```java
public class MyView extends RelativeLayout {
  @InjectView(R.id.my_text_view) TextView _textView;
  ...

  public void populate(MyModel model) {
    _myTextView.setText(model.getText());
  }
}
```

- Use `dp` for dimentions, and `sp` for fonts

- Break out shared layout code into a single .xml file, and include it rather than duplicating it.


