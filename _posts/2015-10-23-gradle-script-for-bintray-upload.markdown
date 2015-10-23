---
layout: post
title:  "Gradle Scripts for Uploading Android Artifacts to Bintray"
date:   2015-10-23 12:00:00
comments: true
categories: Android
---

#1. Create a [Bintray][bintray-link] account and repository.

Follow the on screen instructions.

#2. Create a Bintray repository to host the artifact.

Make note of the repo name, publiser group id, and artifact name. You will plug these values into the script below.

#2. Add code to `build.gradle`.

Fill in any `FILL_IN` tags under Bintray Configuration to include the repository information you create in step 1. above.

```
def versionMajor = 1
def versionMinor = 0
def versionPatch = 0
def versionBuild = 0

def version_code = versionMajor * 10000 + versionMinor * 1000 + versionPatch * 100 + versionBuild
def version_name = "${versionMajor}.${versionMinor}.${versionPatch}"

////////////////////////
// Bintray configuration
////////////////////////

ext {
    bintrayRepo = FILL_IN
    bintrayName = FILL_IN

    publishedGroupId = FILL_IN
    libraryName = FILL_IN
    artifact = FILL_IN

    libraryDescription = FILL_IN

    siteUrl = FILL_IN
    gitUrl = FILL_IN

    libraryVersion = version_name

    developerId = FILL_IN
    developerName = FILL_IN
    developerEmail = FILL_IN

    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]
}


//////////////////
// Maven
//////////////////

apply plugin: 'com.github.dcendents.android-maven'

group = publishedGroupId                               // Maven Group ID for the artifact

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                groupId publishedGroupId
                artifactId artifact

                // Add your description here
                name libraryName
                description libraryDescription
                url siteUrl

                // Set your license
                licenses {
                    license {
                        name licenseName
                        url licenseUrl
                    }
                }
                developers {
                    developer {
                        id developerId
                        name developerName
                        email developerEmail
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl

                }
            }
        }
    }
}


/////////////////
// Bintray upload
/////////////////

apply plugin: 'com.jfrog.bintray'

version = libraryVersion

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
artifacts {
    archives javadocJar
    archives sourcesJar
}

// Bintray
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

bintray {
    // First try to read from local.properties
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    def gpgpassphrase = properties.getProperty("bintray.gpg.password")

    // If the properties aren't set in a properties file, load from environment (needed for CI)
    if (user == null) {
        user = System.getenv('BINTRAY_USERNAME')
    }
    if (key == null) {
        key = System.getenv('BINTRAY_APIKEY')
    }
    if (gpgpassphrase == null) {
        gpgpassphrase = System.getenv('BINTRAY_GPG_PASSPHRASE')
    }



    configurations = ['archives']
    pkg {
        repo = bintrayRepo
        name = bintrayName
        desc = libraryDescription
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = allLicenses
        publish = true
        publicDownloadNumbers = true
        version {
            desc = libraryDescription
            gpg {
                sign = true //Determines whether to GPG sign the files. The default is false
                passphrase = gpgpassphrase
                //Optional. The passphrase for GPG signing'
            }
        }
    }
}
```

Then, you can keep configuration info in `local.properties`. Replace the placeholders with the real values.

```
bintray.apikey=KEY
bintray.user=USER
bintray.gpg.password=PASSWORD
```

Or, in a `.env` file:

```
export BINTRAY_APIKEY=KEY
export BINTRAY_USERNAME=USER
export BINTRAY_GPG_PASSPHRASE=PASSWORD
```

`build.gradle` will first read `local.properties` for the bintray information, and if it's not present, it will check the environment. If you want a CI build script to push to Bintray, then use the environment variables.

[bintray-link]: https://bintray.com
