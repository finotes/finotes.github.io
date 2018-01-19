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

### Initialize
You need to call the Fn.init() function in your launcher activity onCreate() function.

```Java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this);
}
```
### Catch Global Exceptions.
In-order to catch uncaught exceptions, You may use Fn.catchUnCaughtExceptions().
```Java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this);
    Fn.catchUnCaughtExceptions();
}

```

### Monitoring network calls

You need to use the custom OkHttp3Client() provided by Fi.notes in your network calls.


```Java
    import com.finotes.android.finotescore.OkHttp3Client;
    ...
    ...
    ...

    client = new OkHttp3Client(new OkHttpClient.Builder()).build();
```


All network calls using custom OkHttp3Client() provided by Fi.notes, from your application will be automatically monitored for issues like status code errors, timeout issues, exceptions and other failures.


Add @Observe annotation to the same activity class (launcher Activity) where Fn.init() function was called.
If needed, you may provide a timeout for the api calls along with the host names.


```Java
@Observe(domains = {
        @Domain(hostName = "host.com"),
        @Domain(hostName = "p.anotherhost.com", timeOut = 8000)
        })
public class MainActivity extends AppCompatActivity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    	   Fn.init(this);
```





