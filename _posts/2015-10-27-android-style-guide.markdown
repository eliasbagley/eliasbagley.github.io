---
layout: post
title:  "Android Style Guide"
date:   2015-10-27 12:00:00
comments: true
categories: android
---

Note: This guide is a work in progress.

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



