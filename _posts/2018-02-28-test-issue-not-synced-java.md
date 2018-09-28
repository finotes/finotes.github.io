

### Steps to rectify Finotes integration issue 

Incase the test issue created using the below snapshot is not reflected in finotes dashboard.

##### Java
```java
public class BlogApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        Fn.init(this, false , true);
        //Fn.reportIssue allows you to raise custom issues.
        //Refer Custom Issue section by the end of this documentation for more details.
        Fn.reportIssue(this, "Test Issue", Severity.MINOR);
    }
}
```
##### Kotlin
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

#### Step 1.
Make sure that you are using the latest SDK version.  
Make sure that the finotes logging is ON as shown below.  
##### Java
```java
public class BlogApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // Make sure that the second parameter is false and third parameter is true.
        // Second parameter if true, will not sync any issues raised with dashboard
        // Third parameters if true, prints the verbose logging.
        Fn.init(this, false , true); 
        Fn.reportIssue(this, "Test Issue", Severity.MINOR);
    }
}
```
##### Kotlin
```kotlin
class BlogApp: Application() {
    override fun onCreate() {
        super.onCreate()
        // Make sure that the second parameter is false and third parameter is true.
        // Second parameter if true, will not sync any issues raised with dashboard
        // Third parameters if true, prints the verbose logging.
        Fn.init(this, false , true)
        Fn.reportIssue(this, "Test Issue", Severity.MINOR)
    }
}
```
#### Step 2.
Please uninstall the existing application from your emulator or device, then install the application.

#### Step 3.
In your Android Studio logcat at the top type 'Observe' in the search bar.  
This will help us view only finotes logs.  

#### Step 4.
You should get a log that says.
###### 'About to start device registration'  

If not please make sure that Fn.init(this, false, true) is actually called in your Application class onCreate() function.  
Also that your application class is registered in your manifest file.  
If you are still not getting the above log printed, please chat with our agent on the support chat below, we will help you out.

#### Step 5.
Once you have the log **About to start device registration** printed in your logcat.
Check for any errors that are displayed.  
If you get a error that says  
###### 'Registration failed with status code 401'
Please make sure that you are you have created an app with same package name as in **applicationId** field in your **build.gradle** file.  
If you got an error log with any other status code then please do initate a chat with our agent on the support chat below, we will help you out. 

#### Step 6.
Incase you didnt get any status error, but got the below error.  
###### 'Please consult https://finotes.github.io for instructions'

#### Step 7.
Please recheck your network connection. 
If you are on wifi, please make sure that there are no restrictions in place on your wifi connection, especially if you are on a public or workplace wifi connection.  

Just switch your phone to a network connection (3G or 4G) and uninstall the existing app and reinstall the application you should be able to view the issue being reported in finotes dashboard.  

##### If none of the above steps worked, please do start a chat with our agent, we will fix the issue for you.

<span style="color:red">*Once the integration issue is fixed, you should remove the Fn.reportIssue() call, else every time the app is run, an issue will be reported.*</span>


