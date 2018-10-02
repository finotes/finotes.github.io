---
layout: post
title: "Android Kotlin SDK Documentation"
---

# Android Kotlin SDK Version: 2.5.2
##### [Change log](https://finotes.github.io/2018/01/22/android-change-log) 

## Pre requisites

Finotes SDK supports android projects with minimum SDK version 14 (Ice-cream Sandwich) or
above.

## Integration

In-order to integrate Finotes in your Android project, add the code below to project level build.gradle

```gradle

allprojects {
    repositories {
        jcenter()
        maven {
            url "s3://finotescore-android/release"
            credentials(AwsCredentials) {
                accessKey = <Access Token>
                secretKey = <Secret>
            }
        }
   }
}
```
You will be able to get Access Token, Secret from 'Apps' section in Finotes dashboard.

Then in app level build.gradle
```gradle
implementation('com.finotes:finotescore:2.5.2@aar') {
    transitive = true;
}
```

#### Proguard
If you are using proguard in your release build, you need to add the following to your proguard-rules.pro file.

```
-keep class com.finotes.android.finotescore.* { *; }

-keepclassmembers class * {
    @com.finotes.android.finotescore.annotation.Observe *;
}
```
Add the below line incase you want to view the exact line number and source file name in your stacktrace.
```
-keepattributes SourceFile,LineNumberTable
```
Please make sure that the mapping file of your production build (each build) is backed up, inorder to deobfuscate the stacktrace from finotes dashboard.  
Location of mapping file: your-project-folder/app/build/outputs/proguard/release/mapping.txt

## Initialize
You need to call the Fn.init() function in your application onCreate() function.

```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
        Fn.init(this)
    }
}
```

#### DryRun
During development, you can set the dryRun mode, so that the issues raised will not be sent to the server. Every other feature except the issue sync to server will work as same.
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	//Second parameter in Fn.init() toggles dryRun flag, which is false by default.
        Fn.init(this, true, false)
    }
}
```
##### When preparing for production release, you need to unset the DryRun flag.

#### VerboseLog
There are two variations of logging available in Finotes, Verbose and Error. You can toggle them using corresponding APIs.  
Activating verbose will print all logs in LogCat including error and warning logs.
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	//Third parameter in Fn.init() toggles the verbose mode.
	Fn.init(this, false , true)
    }
}
```
##### ErrorLog
If only error and warning logs needs to be printed,
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, true , false)
        Fn.logError(true)
    }
}
```
## Test
Now that the basic integration of Finotes SDK is complete,  
Lets make sure that the dashboard and SDK are in sync. 
##### Step 1. 
Add **Fn.reportIssue(this, "Test Issue", Severity.MINOR)** under **Fn.init(this, false , true)**
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, false , true)
        //Fn.reportIssue allows you to raise custom issues.
        //Refer Custom Issue section by the end of this documentation for more details.
        Fn.reportIssue(this, "Test Issue", Severity.MINOR)
    }
}
```
##### Step 2. 
Now run the application in a simulator or real android device (with network connection).  

##### Step 3.
Once the application opens up, open [finotes dash](https://app.finotes.com/#/tickets).    
The issue that we raised should be reported.   

In-case the issue is not listed, make sure the right app and platform is selected at the top of the dashboard.    
##### If you still cant find the issue in fintoes dashboard, [Click Here](https://finotes.github.io/2018/02/28/test-issue-not-synced-java).  


<span style="color:red">*You should remove the Fn.reportIssue() call, else every time the app is run, an issue will be reported.*</span>

```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, false , true)
//        Fn.reportIssue(this, "Test Issue", Severity.MINOR)
    }
}
```

## ReleaseBuild
Make sure that the dryRun and verbose flags are off in your release build along with the [proguard rules specified above](#proguard).
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
        Fn.init(this) //By default the dryRun and verbose flags are off.
    }
}
```
Please make sure that the mapping file of your production build (each build) is backed up, inorder to deobfuscate the stacktrace from finotes dashboard.  
Location of mapping file: your-project-folder/app/build/outputs/proguard/release/mapping.txt


### Listen for Issue
You can listen for and access every issue in realtime using the Fn.listenForIssue() API.  
You need to add the listener in your Application class.

```kotlin
Fn.listenForIssue(IssueFoundListener { 
    issueView: IssueView? ->
    ...
    ...
    ...
})
```
You will be provided with an Issue object that contains all the issue properties that are being synced to the server, making the whole process transparent.    
As this callback will be made right after an issue occurrence, you will be able to provide a positive message to the user.


### Report Network Call Failure

You need to use the custom OkHttp3Client() provided by Finotes in your network calls.


```kotlin
    import com.finotes.android.finotescore.OkHttp3Client
    ...
    ...
    
    var client = OkHttp3Client(OkHttpClient.Builder()).build()
```

Issues like status code errors, timeout issues, exceptions and other failures will be reported for all network calls using custom OkHttp3Client() provided by Finotes, from your application .

##### Volley
In order to add OkHttp3Client to Volley, set the client to OkHttpStack().
```kotlin
val client = OkHttp3Client(OkHttpClient.Builder()).build()
val requestQueue = Volley.newRequestQueue(context, OkHttpStack(client))
requestQueue.add(jsonObjReq)
```
You may use the OkHttpStack() from the following [github gist](https://gist.github.com/arvi/f1a0d2a812650c546223642856afe1e9)

Incase you are using a singleton or AppController (Application class) to manage volley objects.
```kotlin
	AppController.getInstance().addToRequestQueue(jsonObjReq, tag_json_obj);
```
Go to that singleton or Application class and find the function where you have called 
```kotlin
    val requestQueue: RequestQueue? = null
        get() {
            if (field == null) {
                return Volley.newRequestQueue(applicationContext)
            }
            return field
        }
```
Change this to,
```kotlin
    val requestQueue: RequestQueue? = null
        get() {
            if (field == null) {
    	        val client = OkHttp3Client(OkHttpClient.Builder()).build()
                return Volley.newRequestQueue(context, OkHttpStack(client))
            }
            return field
        }
```
You may use the OkHttpStack() from the following [github gist](https://gist.github.com/arvi/f1a0d2a812650c546223642856afe1e9)  
OkHttp3Client() is a custom class that finotes provides, if the Finotes SDK is connected then it will be auto imported.

##### Retrofit
Adding OkHttp3Client to Retrofit is straight forward, Just call .client() in Retrofit.Builder()
```kotlin
val client = OkHttp3Client(OkHttpClient.Builder()).build()
val retrofit = Retrofit.Builder()
        .client(client)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
 ```
 Incase you are already using a custom client,
```kotlin
	val currentBuilder = new OkHttpClient.Builder()
	... //Builder customization codes
	...
	val client = new OkHttp3Client(currentBuilder).build()	
```

##### retryOnConnectionFailure 
##### connectTimeout
```kotlin
val builder = OkHttpClient.Builder().connectTimeout(15, TimeUnit.SECONDS)
	.retryOnConnectionFailure(false)
val client = OkHttp3Client(builder).build()
```

#### Whitelist Hosts
Optionally, you may add @Observe annotation to the same Application class where Fn.init() function was called.  
If Application class is annotated with @Observe, then, only issues from API calls to hosts listed in @Observe will be raised.

```kotlin
@Observe(domains = arrayOf(Domain(hostName = "host.com"),
        Domain(hostName = "p.anotherhost.com", timeOut = 8000)))
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, false , true)
    }
}
```
Optionally, You may provide a custom timeout for hostnames. This will raise an Issue for any network call to the 
ied hosts, that takes more than the specified time for completion.  
Do note that specifying timeout will not interfere in any way with your network calls.

#### Mask Headers
You have an option to mask headers, incase they contain sensitive information.
```java
@Observe(maskHeaders = {"X-Key", "X-Me"})
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, false , true)
    }
}
```
or
```java
@Observe(maskHeaders = {"X-Key"})
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
	Fn.init(this, false , true)
    }
}
```
This will mask any headers with matching name in both request and response headers before raising the issue to Finotes dashboard.
The maskHeaders field is case insensitive.


#### Categorize Tickets (Optional)

You will be able to configure how API issues are categorized into tickets using a custom header "X-URLID". You can set this header in your API calls and when they are reported to Finotes dashboard, issues from API calls with same "X-URLID" will be categorized into a single ticket.  
```kotlin
       @Headers("X-URLID: loginapi")
```
or
```kotlin
       .addHeader("X-URLID","registrationapi")
```
Any issue in API calls with same X-URLID will be shown as a single ticket in Finotes dashboard.


### Activity trail

When an issue is raised, inorder to get user screen flow for the current session, you need to extend your Activities and Fragments from ObservableActivity or ObservableFragment.

```kotlin
class MapActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
}

Changes to,


class MapActivity : ObservableAppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
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
Once Activities and Fragments are extended from corresponding Observables, when an issue is raised, on Finotes dashboard, you will be able to view screen activity for 3 minutes before the issue was raised. 
```
Activity Trail

    ActivityWelcome:onCreate     11:19:24:469	1131900928 FREE MEMORY (BYTES)
    MapActivity:onCreate         11:19:24:708	1131900928
    MapActivity:onStart          11:19:26:983	1131003904
    MapActivity:onResume         11:19:27:012	1130999808
    ActivityWelcome:onDestroy    11:19:28:515	1130622976
    MapActivity:onPause          11:20:17:806	1129603072
```
### Custom Activity Trail
You can set custom activity markers in your android application using Fn.setActivityMarker(). These markers will be shown along with the activity trail when an issue is reported.

```kotlin
    Fn.setActivityMarker(this@PurchaseActivity, "clicked on payment_package_two");
```

```
Activity Trail

    ActivityWelcome:onCreate                            11:19:24:469	1131900928 FREE MEMORY (BYTES)
    MapActivity:onCreate                                11:19:24:708	1131458560
    MapActivity:onStart                                 11:19:26:983	1131003904
    MapActivity:onResume                                11:19:27:012	1130999808
    ActivityWelcome:onDestroy                           11:19:28:515	1130622976
    MapActivity:onPause                                 11:20:17:806	1129603072
    PurchaseActivity:onCreate                           11:20:18:106	1129230336
    PurchaseActivity:onStart                            11:20:18:404	1129222144
    PurchaseActivity:onResume                           11:20:18:906	1128136704
    PurchaseActivity:clicked on payment_package_two     11:20:24:235	1127759872
```


### Low Memory Reporting 

Optionally, You may extend your Application class from ObservableApplication. This will report any app level memory issues that may arise in your application.

```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
    }
}

Change to,

class BlogApp: ObservableApplication() {
    override fun onCreate() {
        super.onCreate()
    }
}
```


### Catch Global Exceptions.
In-order to catch uncaught exceptions, You may use Fn.catchUnCaughtExceptions().
```kotlin
    override fun onCreate() {
        super.onCreate()
        Fn.init(this)
        Fn.catchUnCaughtExceptions()
    }
```

### Custom Exceptions
You can report custom exceptions using the Fn.reportException() API.
```kotlin
        try {
            val jsonObject = JSONObject()
            jsonObject.put("key","value")
        } catch (exception: Exception) {
            Fn.reportException(this,exception,Severity.MINOR)
        }
```

### Custom Issue
You can report custom issues using the Fn.reportIssue() API.
```kotlin
    override fun paymentCompleted(userIdentifier:String, type:String){
        //Handle post payment
    }

    override fun paymentFailed(userIdentifier:String, reason:String){
        Fn.reportIssue(this@PaymentActivity, reason, Severity.MAJOR)
    }
```

### Function call
Finotes will report any return value issues, exceptions and execution delays that may arise in functions using Fn.call() and @Observe annotation.  
A regular function call will be,

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)
        val userName = getUserNameFromDb("123-sd-12")
    }

    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }
```
Function “getUserNameFromDb()” call needs to be changed to,
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)
        val userName = Fn.call("getUserNameFromDb", this@UserActivity, "123-sd-12") as String?
    }

    @Observe
    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }
```
###### You need to tag the function using @Observe annotation and should be public, in Kotlin functions are public by default.

This will allow Finotes to raise issue incase the function take more than normal time to execute, or if the function return a NULL value, or throws an exception.

## Asynchronous Chained Function calls

Using chained function calls, you can detect when a functionality in your app fails.  

All app features will have a start function and a success function, using Finotes SDK you will be able to chain both these functions.  
You can chain functions using 'nextFunctionId' and 'nextFunctionClass' properties in @Observe annotation.

Lets take a look at an "add to cart" functionality in an typical ecommerce app.

#### Use case: Add to cart
```kotlin
    @Override
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_product_listing)
	...
	...
	addToCartButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
	    	Fn.call("addItemToCart", this@ProductListingActivity, item.getId())
	    }
        })

    }
    
    // Here function 'onItemAddedToCart' is expected to be called in under '5000' milliseconds.
    @Observe(nextFunctionId = "onItemAddedToCart",
            nextFunctionClass = ProductListingActivity::class,
	    expectedChainedExecutionTime = 5000)
    fun addItemToCart(itemId :String): Boolean{
	if(isValid(itemId)){
	     ... 
	     ...
	     return true
	}
	return false
    }
    
    @Override
    fun onApiCallComplete(response :JSONObject){
	if(validResponse(response)){
	     Fn.call("onItemAddedToCart", this@ProductListingActivity, response.getString("id"))
	}
    }
    
    @Observe
    fun onItemAddedToCart(itemId :String){
	showMessage(Messages.CARTED_SUCCESS)
	...
	...
    }
```

##### nextFunctionId/nextFunctionClass
Here, we have connected functions "addItemToCart" to "onItemAddedToCart" using 'nextFunctionId'/'nextFunctionClass' in @Observe with expectedChainedExecutionTime set to 5000 milliseconds.   

##### expectedChainedExecutionTime
Now as soon as "addItemToCart" is executed, Finotes will listen for "onItemAddedToCart" to be executed. If the same is not called within 5000 milliseconds after the execution of "addItemToCart", an issue will be raised that says the function "onItemAddedToCart" is never called.  
Now if the function "onItemAddedToCart" is executed after 5000 milliseconds, another issue will be raised that says the function "onItemAddedToCart" is called with a delay.

##### Note,  
It is recomended to have no user activity when chaining 2 funtions. lets call functions, A and B. i.e after execution of function A, no user activity (like a click or input) should be required for execution of B.

#### Use case: Chat

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContentView(R.layout.activity_chat)
       val sendButton = findViewById<Button>(R.id.sendButton)
       sendButton.setOnClickListener(View.OnClickListener {
	    Fn.call("sendChat", this@ChatActivity, chatBox.getText().toString())
       })
    }

    // Here function 'onChatSent' is expected to be called in under '2000' milliseconds(default value).
    @Observe(nextFunctionId = "onChatSent",
            nextFunctionClass = ChatActivity::class)
    fun sendChat(message: String): Boolean{
	if(isValid(message)){
	     syncMessage(message);
	     return true;
	}
	return false;
    }
    
    @Observe
    override fun onChatSent(chatMessageId: String){
	chatSyncConfirmed(chatMessageId)
    }
```
Here 'onChatSent(String chatMessageId)' should be called within 2000 milliseconds(default value) after execution of 'sendChat(String message)'.  
If the function 'onChatSent' is not called or is delayed then corresponding issues will be raised and reported.


##### boolean returns false
Here in 'sendChat' function, if it returns false, an issue will be raised with the function parameters, which will help you dig more into what could have gone wrong.


#### Test
In-order to check if you have implemented function chaining correctly, set break points inside the chained functions then run application in your simulator or device, and use your application, if you have implemented correctly, You will hit these breakpoints correctly.

You can control multiple parameters in @Observe annotation.
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)
        val userName = Fn.call("getUserNameFromDb", this@UserActivity, "123-sd-12") as String?
    }

    @Observe(expectedExecutionTime = 1400)
    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }
```
##### expectedExecutionTime
Here the expectedExecutionTime for the function "getUserNameFromDb" has been overriden to 1400 milliseconds (default was 1000 milliseconds). If the database query takes more than 1400 milliseconds to return the value or if the returned "userName" is NULL, then corresponding issues will be raised.  

##### Exception in function
An issue will be raised when exception occurs inside a function that is annotated with @Observe and is called using Fn.call().

##### Returns NULL
If you expect a function to return NULL value then, to exempt that function from raising "returns NULL" issue, use "expectNull" field in @Observe annotation.
```kotlin
    @Observe(expectNull = true)
    fun getUserNameFromDb(userId: String): String?{
    }
```

##### Expected Boolean value
If the function annotated with @Observe returns a Boolean value, you may set a expceted value for the same and during execution if the function returns a value other than the expected value, an issue will be raised.
```kotlin
    @Observe(expectedBooleanValue = false)
    fun isValidUser(userId: String): Boolean?{
    }
```

##### Return value range
If the function annotated with @Observe returns a number or decimal value, you may set a range for the same and during execution if the function returns a value outside the specified range, an issue will be raised.
```kotlin
    @Observe(min = 10, max = 15)
    fun getUserCountFromJSON(response: JSONObject): long{
    }
    
    or
    
    @Observe(min = 1.1, max = 1.9)
    fun getUserCountFromJSON(response: JSONObject): float{
    }
```
##### Mask parameters
Inorder to mask function parameters before raising an issue, you may use 'mask' field in @Observe.
```java
    @Observe(mask = {0})
    fun getUserCountFromJSON(response: JSONObject): float{
    }
```
Here, incase an issue is raised in function 'getUserCountFromJSON(JSONObject)', the JSONObject value will be masked.  
You need to provide parameter position starting from 0 in the 'mask' field in @Observe.
```java
    @Observe(mask = {1})
    fun getUserNameFromDb(email: String, token: String): String?{
        return User.findUniqueByProperties(email,token).getName()
    }
```
Here the field 'String token' will be masked during an issue raise.

#### Function overloading
You need to provide an explicit ID in @Observe annotation to the function incase of function overloading.  
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)
        // Here the function name "getUserNameFromDb" is passed in Fn.call().
        // This will trigger the getUserNameFromDb(String userId) function.
        val userName = Fn.call("getUserNameFromDb", this@MainActivity, "123-sd-12") as String?

        // Here the id "getUserNameFromDBWithEmailAndToken" specified in
        // @Observe annotation is passed in Fn.call().
        // This will trigger the getUserNameFromDb(String email, String token) function.
        val userNameFromEmail = Fn.call("getUserNameFromDbWithEmailAndToken", this@MainActivity
                                            , "support@finotes.com", "RENKDS123S") as String?
    }

    @Observe(expectedExecutionTime = 1400)
    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }

    @Observe(id = "getUserNameFromDbWithEmailAndToken", expectedExecutionTime = 1400)
    fun getUserNameFromDb(email: String, token: String): String?{
        return User.findUniqueByProperties(email,token).getName()
    }
```

##### ID
If a custom id is specified in @Observe annotation, then while calling that function, you need mandatorily to pass the id and not the function name.  
If no custom id is specified in @Observe annoatation, then you can pass the function name itself as the id.
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)

	//  Function name itself is provided as function getUserNameFromDb(String userId) 
	//	doesnt have any explicit id in its @Observe annotation
        val userName = Fn.call("getUserNameFromDb", this@MainActivity, "123-sd-12") as String?

	//  ID 'getUserNameFromDBWithEmailAndToken' is provided as function 
	//      getUserNameFromDb(String email, String token) 
	//	does have an explicit id in its @Observe annotation
        val userNameFromEmail = Fn.call("getUserNameFromDbWithEmailAndToken", this@MainActivity
                                            , "support@finotes.com", "RENKDS123S") as String?
    }

    @Observe
    fun getUserNameFromDb(userId: String): String?{
    }

    @Observe(id = "getUserNameFromDbWithEmailAndToken")
    fun getUserNameFromDb(email: String, token: String): String?{
    }
```


#### Function in Separate Class file

If the function is defined in a separate class file and needs to be invoked, then instead of passing "this" you can pass the corresponding object in Fn.call()
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_user)
    
    val dbUtils = DbUtils()
    val userName = Fn.call("getUserNameFromDb", dbUtils, "123-sd-12") as String?

}

class DbUtils {
    @Observe
    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }
}
```

#### Static Function calls

For static functions, pass corresponding class instead of object in Fn.call() to invoke static method.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_user)
    
    var userTimeStamp = Fn.call("getUserTimestampFromJSON",DbUtils,JSONObject()) as Long
}

class DbUtils {
    // Static function in Kotlin
    companion object {
        @Observe(expectedExecutionTime = 2000)
        fun getUserTimestampFromJSON(response: JSONObject): Long {
            return processJSONAndFindUserTimestamp(response)
        }
    }

    @Observe
    fun getUserNameFromDb(userId: String): String?{
        return User.findById(userId).getName()
    }
}
```
























