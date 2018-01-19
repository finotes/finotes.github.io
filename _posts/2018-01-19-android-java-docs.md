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

##### Volley
##### Retrofit
Inorder to find out how you can integrate OkHttp3Client in Volley, Retrofit, do check out [OkHttp3Client in Volley and Retrofit](https://www.google.com)  


Optionaly, you may add @Observe annotation to the same activity class (launcher Activity) where Fn.init() function was called.  
If launcher activity is annotated with @Observe, then only API calls to hosts listed in @Observe will be monitored.

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
    }
}
```
Optionaly, You may provide a custom timeout for hostnames. This will raise an Issue for any network call to the specified hosts, that takes more than the specified time for completion.  
Do note that specifing timeout will not interfere in any way with your network calls.


### Activity/Fragment monitoring

When an issue is raised, inorder to get user screen flow for the current session, you need to extend your Activities and Fragments from ObservableActivity or ObservableFragment.

```Java
public class MainActivity extends AppCompatActivity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

Changes to,


public class MainActivity extends ObservableAppCompatActivity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

```
You may extend from the list below.
```
    ObservableAppCompatActivity 
    ObservableActivity 
    ObservableFragmentActivity 
    ObservableFragment
```
Once all Activities and Fragments are extended from corresponding Observables,  
Once an issue is raised, on finotes dashboard, you will be able to view the screen activity for 3 minutes before that issue was raised. 
```
Activity Trail

    ActivityWelcome:onCreate     11:19:24:469
    MapActivity:onCreate         11:19:24:708
    MapActivity:onStart          11:19:26:983
    MapActivity:onResume         11:19:27:012
    ActivityWelcome:onDestroy    11:19:28:515
    MapActivity:onPause          11:20:17:806
```

### Memory monitoring

Optionaly, You may extend your Application class from ObservableApplication. This will report any app level memory issues that may arise in your application.

```Java
public class BlogApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
    }
}

Change to,

public class BlogApplication extends ObservableApplication {
    @Override
    public void onCreate() {
        super.onCreate();
    }
}
```
### Function call
You may monitor function calls for return value, exceptions and execution delays using Fn.call() and @Observe annotation.  
A regular function call will be,

```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    String userName = getUserNameFromDb("123-sd-12");

}

public String getUserNameFromDb(String userId){
    String userName = Db.findById(userId).getName();
    return userName;
}
```
Function “getUserNameFromDb()” can be monitored by changing the function call to,
```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    String userName = (String) Fn.call(“getUserNameFromDb", this, "123-sd-12");
}

@Observe
public String getUserNameFromDb(String userId){
    String userName = Db.findById(userId).getName();
    return userName;
}
```
You need to tag the function that needs to be monitored using @Observe annotation.

This will allow Fi.notes to raise issue incase the function take more than normal time to execute, or if the function return a NULL value, or throws an exception.

You can control all the above said parameters in @Observe annotation.
```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    String userName = (String) Fn.call(“getUserNameFromDb", this, "123-sd-12");
}

@Observe(expectedExecutionTime = 1000)
public String getUserNameFromDb(String userId){
    String userName = Db.findById(userId).getName();
    return userName;
}
```
Here the expectedExecutionTime for the function "getUserNameFromDb" has been overriden to 1000 milliseconds. If the database query takes more than 1000 milliseconds to return the value or if the returned "userName" is NULL, then corresponding issues will be raised.














