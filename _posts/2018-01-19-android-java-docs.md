---
layout: post
title: "Android SDK Documentation"
date: 2018-01-19
---

## Android Java Documentation

### Pre requisites

Finotes SDK supports android projects with minimum SDK version 14 (Ice-cream Sandwich) or
above.

### Integration

In-order to integrate finotes in your Android project, add the code below to project level build.gradle

```Gradle

allprojects {
    repositories {
        jcenter()
        maven {
            url "s3://finotes-core/releases"
            credentials(AwsCredentials) {
                accessKey = <AWS_ACCESS_KEY>
                secretKey = <AWS_SECRET_KEY>
            }
        }
   }
}
```

Then in app level build.gradle

```Gradle
compile('com.finotes:finotescore:1.0@aar') {
    transitive = true;
}
```
#### Progruard
If you are using proguard in your release build, you need to add the following to your proguard-rules.pro file.

```
-keep class com.finotes.android.finotescore.* { *; }

-keepclassmembers class * {
    @com.finotes.android.finotescore.annotation.Observe *;
}
```


