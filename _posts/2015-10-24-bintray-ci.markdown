---
layout: post
title:  "Publish to Bintray with Continuous Integration"
date:   2015-10-24 12:00:00
comments: true
categories: Android
---

Set up Gradle for pushing to Bintray. See the instructions [here][bintray-setup-link]

Add the following to the bottom of `build.gradle`:

```
task('gitTag') {
    "git tag ${version_name}".execute([], project.rootDir)
    "git push --tags".execute([], project.rootDir)
}
```

```bash
# Fill this in with the path to your Android SDK. May not be necessary if this environment variable is already populated
export ANDROID_HOME="/PATH/TO/ANDROID/HOME/DIRECTORY"
export JAVA_HOME=$(/usr/libexec/java_home)

export BINTRAY_APIKEY=FILL_IN
export BINTRAY_USERNAME=FILL_IN
export BINTRAY_GPG_PASSPHRASE=FILL_IN

./gradlew install
./gradlew bintrayUpload
./gradlew gitTag
```

Set it to read from the master branch, and to publish on pushes to this branch. Then, whenever you merge to master, an artifact with the same version specified in `build.gradle` will be published to JCenter. Please make sure that the version number is updated before you push to master, or else the upload will fail because Bintray rejects artifacts with duplicate names.

[bintray-setup-link]: http://eliasbagley.github.io/android/2015/10/23/gradle-script-for-bintray-upload.html
