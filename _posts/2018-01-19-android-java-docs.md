---
layout: post
title: "Android SDK Documentation"
---

## Pre requisites

Fi.notes SDK supports android projects with minimum SDK version 14 (Ice-cream Sandwich) or
above.

## Integration

In-order to integrate fi.notes in your Android project, add the code below to project level build.gradle

```gradle

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
```gradle
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
There are two variations of logging available in Fi.notes, Verbose and Error. You can toggle them using corresponding APIs.  
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

## Reporting Issues

### Report Network Call Failure

You need to use the custom OkHttp3Client() provided by Fi.notes in your network calls.


```java
    import com.finotes.android.finotescore.OkHttp3Client;
    ...
    ...
    
    client = new OkHttp3Client(new OkHttpClient.Builder()).build();
```

All network calls using custom OkHttp3Client() provided by Fi.notes, from your application will be automatically monitored for issues like status code errors, timeout issues, exceptions and other failures.

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
Optionaly, you may add @Observe annotation to the same activity class (launcher Activity) where Fn.init() function was called.  
If launcher activity is annotated with @Observe, then only API calls to hosts listed in @Observe will be monitored.

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
Optionaly, You may provide a custom timeout for hostnames. This will raise an Issue for any network call to the specified hosts, that takes more than the specified time for completion.  
Do note that specifing timeout will not interfere in any way with your network calls.


### Activity/Fragment monitoring

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

### Memory monitoring

Optionaly, You may extend your Application class from ObservableApplication. This will report any app level memory issues that may arise in your application.

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
You may monitor function calls for return value, exceptions and execution delays using Fn.call() and @Observe annotation.  
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
Function “getUserNameFromDb()” can be monitored by changing the function call to,
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
You need to tag the function that needs to be monitored using @Observe annotation and the function needs to be 'public'.

This will allow Fi.notes to raise issue incase the function take more than normal time to execute, or if the function return a NULL value, or throws an exception.

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
An issue will be raised if an exception is raised inside a function that is being monitored by Fi.notes.


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
						this, "email", "RENKDS123S");
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
    
    long userTimestamp = (long) Fn.call("getUserTimestampFromJSON", DBUtils.class, httpResponseJSONObject);

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

You can connect multiple functions using ‘nextFunctionId’ and 'nextFunctionClass' properties in @Observe annotation.

Lets consider Facebook LOGIN process in an application.

When user clicks on the facebook login button the login workflow will take the user to a custom facebook screen where the user authenticates and comes back with a list of permissions that the user has approved, then a login API call is initiated to the app backend and the result could be success or failure.

##### nextFunctionId
##### nextFunctionClass
##### expectedChainedExecutionTime

```java
public class LoginActivity extends ObservableAppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        fbLoginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //Inside a click listener or an interface function you need to specify the class object
                // as in this case LoginActivity.this and simply passing 'this' would not work.
                Fn.call("onFbLoginClicked", LoginActivity.this);
            }
        });
    }
    
    // Here @Observe monitors if 'makeLoginApiCall' function is called within '10000' milliseconds.
    @Observe(nextFunctionId = "makeLoginApiCall",  
                nextFunctionClass = LoginActivity.class, 
                expectedChainedExecutionTime = 10000)
    public void onFbLoginClicked() {
        fnLogin.initateFbLogin(LoginActivity.this, null, callbackManager, new FbLoginListener() {
            @Override
            public void onLoginComplete(LoginResult loginResult) {
                Fn.call("makeLoginApiCall", LoginActivity.this, 
				loginResult.getToken(), loginResult.getUserId(),
				AccessToken.getCurrentAccessToken().getPermissions());
            }

            @Override
            public void onLoginFailed() {
	    	// You can report custom issue using Fn.issue().
	    	// Check custom issue section for more details
	        Fn.issue(this, "Facebook login failed");
            }
        });
    }
    
    // Here @Observe monitors if 'processUserLogin' function is called within '5000' milliseconds.
    @Observe(nextFunctionId = "processUserLogin"
                nextFunctionClass = LoginActivity.class, 
                expectedChainedExecutionTime = 5000)
    public boolean makeLoginApiCall(String fbToken, String fbUserId, Set<String> permisisonSet){
        if (permissionSet.contains("publish_actions")){
            //On login API success 'onApiCompleted' function will be called.
            new LoginAPI().call(fbToken, fbUserId);
            return true;
        }else{
            //Toast: "Publish permission is required"
            return false;
        }
    }
    
    @Override
    public void onApiCompleted(JSONObject response) {
        Fn.call("processUserLogin", LoginActivity.this, response);
    }
    
    public void processUserLogin(JSONObject response) {
        if(response.isSuccess()){
            //Move to MainActivity
        }
    }
}
```

Here, incase the facebook Login is failed or is delayed by more than 10000 milliseconds, corresponding issues will be raised.  
Also, after facebook login, if the Login API call is not returned or is delayed by more than 5000 milliseconds then, corresponding issues will be raised.  
In case of function chaining you need to use 'expectedChainedExecutionTime' and not 'expectedExecutionTime' to specify the expected time required for chained function to be called.  

##### boolean returns false
Here in 'makeLoginApiCall' function, if it returns false, an issue will be raised with the function parameters, which will help you dig more into what could have gone wrong.


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
private void paymentCompleted(String userIdentifier, int type){
    //Handle post payment.
}

private void paymentFailed(String userIdentifier, String reason){
    Fn.issue(this, "payment failed for "+reason);
    //Handle payment failure.
}
```

### Listen for Issue
You can listen for and access every issue in realtime using the Fn.listenForIssue() API.
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
























