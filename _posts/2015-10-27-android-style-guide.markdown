---
layout: post
title:  "Android Style Guide"
date:   2015-10-27 12:00:00
comments: true
categories: android
---

Note: This guide is a work in progress.

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

# Add the Debug Drawer

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
