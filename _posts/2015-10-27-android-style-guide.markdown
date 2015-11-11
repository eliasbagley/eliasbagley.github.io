---
layout: post
title:  "Android Style Guide"
date:   2015-10-27 12:00:00
comments: true
categories: android
---

Note: This guide is a work in progress.

# Android Studio

Recommended plugins: TODO

## Always keep Android Studio up to date



# use camelCase

# public static final int CONSTANTS\_SHOULD\_BE\_IN\_SHOUTING\_SNAKE\_CASE = 42

# Put constants into a Constants.java class

# Align fields and variables into columns

In Android Studio, navigate to:

```
Preferences | Editor | Code Style | Java | Wrapping and Braces | Group declarations | Align fields in columns
```

# Use Lombok for data classes

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


# Prefer interface types over implementation types

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

# Trailing /> in XML on a separate line


YES:

```
<Button
    android:id="@+id/activity_login_twitter"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Login with Twitter"
    />
```

NO:

```
<Button
    android:id="@+id/activity_login_twitter"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Login with Twitter"/>
```

This makes it easy to add new properties at the end of the property list.

# When in doubt, make custom views a subclass of RelativeLayout

It offers the most flexibility

# Setup App Architecture Using Dagger

# Keep all Dagger modules together in a daggermodules package

# Subclass all app objects from a base Object

Subclass models from BaseModel
Subclass custom views from BaseView
Subclass activites from BaseActivity
Subclass fragments from BaseFragment

# Follow the Android Studio convention for naming layout and view xml files

example..

view_my_button.xml
layout_main_activity.xml

Android only has a single resources folder, and subfolers aren't allowed. Naming them this way visually groups related xml files together.

# Add the Debug Drawer for every project

# Retrofit API interfaces in a separate class

# All API interfaces together in a single folder

# Use @PATCH instead of @PUT for API updating

# Open braces on the same line

example:

```java
public static void main(String[] args) {
}
```

# Reference all custom colors from a colors.xml class. Don't hard code colors into XML

_colors.xml_

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="white">#FFFFFF</color>
    <color name="black">#000000</color>
</resources>
```

# Reference all strings from strings.xml. Don't hard code strings.

# Reference constants (margins, font sizes) from dimens.xml

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

# Prefer RecyclerView for lists

# Use Timber library for logging

# Add all dependencies through gradle. Don't add standalone jars into the project.

# Always have CI setup

# Prefer Activities over Fragments, unless they're necessary

# Always pair custom view xml with a custom view subclass. Have `populate` methods on the subclass to bind model to view

```java
public class MyView extends RelativeLayout {
  @InjectView(R.id.my_text_view) TextView _textView;
  ...

  public void populate(MyModel model) {
    _myTextView.setText(model.getText());
  }
}
```

# Prefix private variables in activities

```java
@Inject Manager _manager;
```

# Use ButterKnife for View Injection

yes:

```java
@InjectView(R.id.my_view) TextView _myView;
```

no:

```
_myView = (TextView)findViewById(R.id.my_view);
```

# Use ButterKnife `@OnClick` annotations instead of setting an `OnClickListener`

yes:

```java
@OnClick(R.id.my_button)
public void myButtonClicked(Button b) {
}
```

no:

```java
_myButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        myButtonClicked();
    }
});
```


# Generate assets only for xxdpi and let Android down sample as needed


# Add utility code to a Util class

# Use Otto event bus where extreme decoupling is needed. Pair with Otto plugin. Don't overuse otto.

# Have a ServiceRequest subclass for every request

# Have a ServiceResponse subclass for every response

# If find that you have a class that has multiple convenience constructors to aid in initializing optional variables, use the Builder pattern (Lombok can generate it)

# Avoid 'naked' variables

yes:

```java
private String myString;
public String getMyString() {
  return this.myString
}
```

no:

```java
public String myString;
```

# TODO .staging, .debug etc

# TODO structure of build.gradle

# Use github pull requests for workflow/code reviews. Use hub to do this from the command line.

# Feel free to use Kotlin for a util class. Keep code that interacts heavily with Android in Java for now.

# Put @Inject, @InjectView annotations on the same line

# When ButterKnife @InjectView isn't available (Like in Android library projects) use ButterKnife's `findById` method instead

yes:
import static butterknife.ButterKnife.findById;

_myButton = findById(this, R.id.my_button);

no:
_myButton = (Button)findViewById(R.id.my_button)

# Don't litter activities with intent code. Have each activity instead present a public static show(Activity activity) method

# Use `Snackbar` instead of `Toast`

# Use `CoordinatorLayout` instead of `FrameLayout`

# Use `dp` for dimentions, and `sp` for fonts

# Include layouts in xml from a single place, rather than duplciating the code across two different layout files

# Use editor config (enabled by default in Android Studio.. just don't turn it off)

TODO: paste a cample editor config to use for android projects

# Field annotations should not wrap. Change in Android Setudio settings: set field annotations to `do not wrap`

yes:
```java
class MyClass {
  @Inject Manager _myManager;
}

```

no:
```java
class MyClass {
  @Inject
  Manager _myManager;
}
```

# Method annotations should wrap (Android Studio Default)

yes:

```java
@Override
public void myMethod() {
}
  ```

no:
```java
@Override public void myMethod() {
}
```

`Preferences -> Editor -> Code Style -> Java -> Wrapping and Braces tab -> Field Annotations`

# Consistent id names and variable names. snake_case id names to camelCase variable names

```java
_googleButton = findById(this, R.id.google_button);
```


# Instead of using IntroActivity.this in an anonymous inner class, use getActivity() (put this method in a superclass)

This makes the code easily portable across activities

# Colors in colors.xml should be uppercase

<color name="colorPrimary">#FF9B00</color>      <!-- Main Toolbar color -->

# Use a color pallete, not a new color for everything.

It's a code smell to have dozens of colors in colors.xml!

# Have a "layout/spacing pallete" button spacing, margins, etc

Be consistent layout margins, button heights, font sizes, etc

# Use `match_parent` instead of `fill_parent` in layouts

They do the same thing, but `match_parent` is the new version, `fill_parent` has been deprecated.

# Break up the res/layouts folder into multiple layouts

# For gradle, avoid using + to specify version in the dependencies block. Use exact version numbers instead.

Using + can lead to build reproducibility problems down the road.

yes:
`compile 'com.google.code.gson:gson:2.2.3'`

no:
`compile 'com.google.code.gson:gson:2.2.+'`

# Project Guidelines

# Use Android Studio's default package and folder structure

Put code into a release/ subfolder if it will only ever be used for release
Put code into a debug/ subfolder if it will ever only be used for debug builds
Everything else goes in the main subfolder
Release mode can override classes in the main folder. Do this if you have classes with methods that will change depending on which build flavor you're using


# You can override classes by


# TODO: Naming convention for layouts, drawables, icons

# TODO: Don't swallow exceptions

# TODO: Class member ordering (constants, fields, constructors, override methods and callbacks, public public methods, privaet methods, inner classes or interfaces

# Use //region description to mark the beginning of a local section of code. Use //endregion to close it.

Android Studio understands these tags to be able to collapse sections of the file


# Have order of Android Acitivty callbacks match the lifecycle

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

# Provide a private construtor for non-instantiable Util classes, so they don't generate a default constructor

# Use type inference for generics

yes:

`List<Integer> list = new ArrayList<>();`

no:

`List<Integer> list = new ArrayList<Integer>();`

# BUILD / GRADLE SECTION

# Use build type strings to define the user-visble project name

TODO: or do this in the build.gradle buildTypes like in Fortify?

```
  src
    ├── debug
    │   └── res
    │       └── buildtype_strings.xml
    └── release
      └── res
            └── buildtype_strings.xml
```

# Set the app's version in `build.gradle`, NOT in `AndroidManifest.xml`

Put the version information right at the top of the file

# Put `dependency{}` and `compile` statements in the module build.config, NOT in the top level project build.config

# RELEASE SECTION

Put keystore information in a keystore.properties file and read the property file at buildtime

# JAVA SECTION

Parameter ordering - if a `Context` object is being passed in, it comes first.
if a `Callback` object is being passed in, it comes last.

yes:

```java
public void getUser(Context context, Callback<User> callback) {
  ...
}
```

# Don't suffix model names with Model

yes:
`User user = new User();`

no:
`UserModel user = new UserModel();`

# Use styles.xml rather than specifying lots of duplicated attributes in xml files

# Use retrolambda and lambda syntax for single method interfaces
