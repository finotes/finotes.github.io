---
layout: post
title: "Android SDK Documentation"
---

## Android SDK Version: 2.1.0 (Outdated)

### To check [Latest Version Documentation:](https://finotes.github.io/2018/02/02/java-docs)

## Pre requisites

fi.notes SDK supports android projects with minimum SDK version 14 (Ice-cream Sandwich) or
above.

## Integration

In-order to integrate fi.notes in your Android project, add the code below to project level build.gradle

```gradle

allprojects {
    repositories {
        jcenter()
        maven {
            url "s3://finotescore-android/release"
            credentials(AwsCredentials) {
                accessKey = <AWS_ACCESS_KEY>
                secretKey = <AWS_SECRET_KEY>
            }
        }
   }
}
```
You will be able to get AWS_ACCESS_KEY, AWS_SECRET_KEY from 'Apps' section in fi.notes. dashboard.

Then in app level build.gradle
```gradle
compile('com.finotes:finotescore:2.1.0@aar') {
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

## Initialize
You need to call the Fn.init() function in your launcher activity onCreate() function.

```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this);
}
```

#### DryRun
During development, you can set the dryRun mode, so that the issues raised will not be sent to the server. Every other feature except the issue sync to server will work as same.
```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    //Second parameter in Fn.init() toggles dryRun flag, which is false by default. 
    Fn.init(this, true , false);
}
```
#### VerboseLog
There are two variations of logging available in fi.notes, Verbose and Error. You can toggle them using corresponding APIs.  
Activating verbose will print all logs in LogCat including error and warning logs.
```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    //Third parameter in Fn.init() toggles the verbose mode.
    Fn.init(this, true , true);
}
```
##### ErrorLog
If only error and warning logs needs to be printed,
```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this, true , false);
    Fn.logError(true);
}
```
## Test
Now that the basic integration of fi.notes SDK is complete,
Lets make sure that the dashboard and SDK are in sync. 
```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this, false , true);
    //Fn.issue allows you to raise custom issues.
    //Refer Custom Issue section by the end of this documentation for more details.
    Fn.issue(this, "Test Issue", Severity.MINOR);

}
```
Now run the application in a simulator or real android device (with network connection).  
Once the application opens up, open [fi.notes dash](https://app.finotes.com/FinotesRS/#/tickets).    
The issue that we raised should be reported.   
In-case the issue is not listed, make sure the right app is selected at the top of the dashboard.  
Also, do refer to the logs in LogCat, it will list any errors/warnings that might have occured during the integration.  

If the error still persists, do contact us at [fi.notes contact email](mailto:support@finotes.com) or you may use the chat support at the bottom right corner.

<span style="color:red">*You should remove the Fn.issue() call, else every time the app is run, an issue will be reported.*</span>

```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this, false , true);
//  Fn.issue(this, "Test Issue", Severity.MINOR);

}
```

### Listen for Issue
You can listen for and access every issue in realtime using the Fn.listenForIssue() API.  
You need to add the listener in your Application class.
```java
Fn.listenForIssue(new IssueFoundListener() {
    @Override
    public void issueFound(IssueView issue) {
	  ….
	  ….
	  ….	
    }
});
```
You will be provided with an Issue object that contains all the issue properties that are being synced to the server, making the whole process transparent.    
As this callback will be made right after an issue occurrence, you will be able to provide a positive message to the user.


## Reporting Issues

### Report Network Call Failure

You need to use the custom OkHttp3Client() provided by fi.notes in your network calls.


```java
    import com.finotes.android.finotescore.OkHttp3Client;
    ...
    ...
    
    client = new OkHttp3Client(new OkHttpClient.Builder()).build();
```

Issues like status code errors, timeout issues, exceptions and other failures will be reported for all network calls using custom OkHttp3Client() provided by fi.notes, from your application .

##### Volley
In order to add OkHttp3Client to Volley, set the client to OkHttpStack().
```java
OkHttp3Client client = new OkHttp3Client(new OkHttpClient.Builder()).build();
RequestQueue mRequestQueue = 
	Volley.newRequestQueue(context,new OkHttpStack(client));
mRequestQueue.add(jsonObjReq);
```

##### Retrofit
Adding OkHttp3Client to Retrofit is straight forward, Just call .client() in Retrofit.Builder()
```java
OkHttp3Client client = new OkHttp3Client(new OkHttpClient.Builder()).build();
Retrofit retrofit = new Retrofit.Builder()
	.baseUrl(API_URL)
	.client(client)
	.addConverterFactory(GsonConverterFactory.create())
	.build();
```

##### retryOnConnectionFailure 
##### connectTimeout
```java
OkHttpClient.Builder builder = new OkHttpClient.Builder()
	.connectTimeout(15, TimeUnit.SECONDS)
	.retryOnConnectionFailure(false);
OkHttp3Client  client = new OkHttp3Client(builder).build();
```

#### Whitelist Hosts
Optionally, you may add @Observe annotation to the same activity class (launcher Activity) where Fn.init() function was called.  
If launcher activity is annotated with @Observe, then, only issues from API calls to hosts listed in @Observe will be raised.

```java
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
Optionally, You may provide a custom timeout for hostnames. This will raise an Issue for any network call to the 
ied hosts, that takes more than the specified time for completion.  
Do note that specifying timeout will not interfere in any way with your network calls.

#### Categorize Tickets (Optional)

You will be able to configure how API issues are categorized into tickets using a custom header "X-URLID". You can set this header in your API calls and when they are reported to fi.notes dashboard, issues from API calls with same "X-URLID" will be categorized into a single ticket.  
```java
       @Headers("X-URLID: loginapi")
```
or
```java
       .addHeader("X-URLID","registrationapi")
```
Any issue in API calls with same X-URLID will be shown as a single ticket in fi.notes dashboard.


### Activity/Fragment trail

When an issue is raised, inorder to get user screen flow for the current session, you need to extend your Activities and Fragments from ObservableActivity or ObservableFragment.

```java
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
Once Activities and Fragments are extended from corresponding Observables, when an issue is raised, on fi.notes dashboard, you will be able to view screen activity for 3 minutes before the issue was raised. 
```
Activity Trail

    ActivityWelcome:onCreate     11:19:24:469
    MapActivity:onCreate         11:19:24:708
    MapActivity:onStart          11:19:26:983
    MapActivity:onResume         11:19:27:012
    ActivityWelcome:onDestroy    11:19:28:515
    MapActivity:onPause          11:20:17:806
```

### Low Memory Reporting 

Optionally, You may extend your Application class from ObservableApplication. This will report any app level memory issues that may arise in your application.

```java
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
fi.notes will report any return value issues, exceptions and execution delays that may arise in functions using Fn.call() and @Observe annotation.  
A regular function call will be,

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    String userName = getUserNameFromDb("123-sd-12");

}

public String getUserNameFromDb(String userId){
    String userName = User.findById(userId).getName();
    return userName;
}
```
Function “getUserNameFromDb()” call needs to be changed to,
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    String userName = (String) Fn.call("getUserNameFromDb", this, "123-sd-12");
}

@Observe
public String getUserNameFromDb(String userId){
    String userName = User.findById(userId).getName();
    return userName;
}
```
You need to tag the function using @Observe annotation and the function should to be 'public'.

This will allow fi.notes to raise issue incase the function take more than normal time to execute, or if the function return a NULL value, or throws an exception.

You can control all the above said parameters in @Observe annotation.

##### expectedExecutionTime

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    String userName = (String) Fn.call("getUserNameFromDb", this, "123-sd-12");
}

@Observe(expectedExecutionTime = 1400)
public String getUserNameFromDb(String userId){
    String userName = User.findById(userId).getName();
    return userName;
}
```
##### Returns NULL
##### Exception in function
Here the expectedExecutionTime for the function "getUserNameFromDb" has been overriden to 1400 milliseconds (default was 1000 milliseconds). If the database query takes more than 1400 milliseconds to return the value or if the returned "userName" is NULL, then corresponding issues will be raised.  
An issue will be raised when exception occurs inside a function that is annotated with @Observe and is called using Fn.call().


#### Function overloading

You need to provide an explicit ID in @Observe annotation to the function incase of function overloading.  
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // Here the function name "getUserNameFromDb" is passed in Fn.call().
    // This will trigger the getUserNameFromDb(String userId) function.    
    String userName = (String) Fn.call("getUserNameFromDb", this, "123-sd-12");

    // Here the id "getUserNameFromDBWithEmailAndToken" specified in 
    // @Observe annotation is passed in Fn.call().
    // This will trigger the getUserNameFromDb(String email, String token) function.    
    String userName = (String) Fn.call("getUserNameFromDBWithEmailAndToken", 
						this, "support@finotes.com", "RENKDS123S");
}

@Observe(expectedExecutionTime = 1400)
public String getUserNameFromDb(String userId){
    String userName = User.findById(userId).getName();
    return userName;
}

@Observe(id = "getUserNameFromDBWithEmailAndToken", expectedExecutionTime = 1400)
public String getUserNameFromDb(String email, String token){
    String userName = User.findUniqueByProperties(email,token).getName();
    return userName;
}
```

##### ID
If a custom id is specified in @Observe annotation, then while calling that function, you need mandatorily to pass the id and not the function name.  
If no custom id is specified in @Observe annoatation, then you can pass the function name itself as the id.
```java
@Override
protected void onCreate(Bundle savedInstanceState) {

//  Function name itself is provided as function getUserNameFromDb(String userId) 
//	doesnt have any explicit id in its @Observe annotation
    Fn.call("getUserNameFromDb", this, "123-sd-12");

//  ID 'getUserNameFromDBWithEmailAndToken' is provided as function 
//      getUserNameFromDb(String email, String token) 
//	does have an explicit id in its @Observe annotation
    Fn.call("getUserNameFromDBWithEmailAndToken", 
						this, "support@finotes.com", "RENKDS123S");
}

@Observe
public String getUserNameFromDb(String userId){
}

@Observe(id = "getUserNameFromDBWithEmailAndToken")
public String getUserNameFromDb(String email, String token){
}
```


#### Function in Separate Class file

If the function is defined in a separate class file and needs to be invoked, then instead of passing "this" you can pass the corresponding object in Fn.call()
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    DBUtils dbUtils = new DbUtils();
    String userName = (String) Fn.call("getUserNameFromDb", dbUtils, "123-sd-12");

}

public class DBUtils {

    @Observe(expectedExecutionTime = 1400)
    public String getUserNameFromDb(String userId){
        String userName = User.findById(userId).getName();
        return userName;
    }
}
```

#### Static Function calls

For static functions, pass corresponding .class instead of object in Fn.call() to invoke static method.

```java
public void onHttpCallCompleted(JSONObject httpResponseJSONObject) {
    
    long userTimestamp = (long) Fn.call("getUserTimestampFromJSON", DBUtils.class,
    							httpResponseJSONObject);

}

public class DBUtils {

    @Observe(expectedExecutionTime = 2000)
    public static long getUserTimestampFromJSON(JSONObject response){
        long userTimestamp = processJSONAndFindUserTimestamp(response);
        return userTimestamp;
    }

    @Observe(expectedExecutionTime = 1400)
    public String getUserNameFromDb(String userId){
        String userName = User.findById(userId).getName();
        return userName;
    }
}
```

### Chained Function calls

You can connect multiple functions using 'nextFunctionId' and 'nextFunctionClass' properties in @Observe annotation.
```java
    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat);


	sendButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
		Fn.call("sendChat", this, chatBox.getText().toString());
            }
        });

    }
    
    // Here function 'onChatSent' is expected to be called in under '2000' milliseconds.
    @Observe(nextFunctionId = "onChatSent",
            nextFunctionClass = MainActivity.class)
    public boolean sendChat(String message){
	if(isValid(message)){
	     syncMessage(message);
	     return true;
	}
	return false;
    }
    
    @Observe
    public void onChatSent(String chatMessageId){
	chatSyncConfirmed(chatMessageId)
    }
```
Here 'onChatSent(String chatMessageId)' should be called within 2000 milliseconds after execution of 'sendChat(String message)'.  
If the function 'onChatSent' is not called or is delayed then corresponding issues will be raised and reported.

##### nextFunctionId
##### nextFunctionClass
##### expectedChainedExecutionTime

##### boolean returns false
Here in 'sendChat' function, if it returns false, an issue will be raised with the function parameters, which will help you dig more into what could have gone wrong.


#### Test
In-order to check if you have implemented function chaining correctly, set break points inside the chained functions then run application in your simulator or device, and use your application, if you have implemented correctly, You will hit these breakpoints correctly.


### Catch Global Exceptions.
In-order to catch uncaught exceptions, You may use Fn.catchUnCaughtExceptions().
```java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Fn.init(this);
    Fn.catchUnCaughtExceptions();
}

```

### Custom Exceptions
You can report custom exceptions using the Fn.exception() API.
```java
try {
    JSONObject jsonObject = new JSONObject();
    jsonObject.put("key","value");
} catch (JSONException exception) {
    Fn.exception(this, exception);
}
```

### Custom Issue
You can report custom issues using the Fn.issue() API.
```java
@Override
private void paymentCompleted(String userIdentifier, int type){
    //Handle post payment.
}

@Override
private void paymentFailed(String userIdentifier, String reason){
    Fn.issue(this, "payment failed for "+reason);
    //Handle payment failure.
}
```

























