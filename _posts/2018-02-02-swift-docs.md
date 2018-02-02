---
layout: post
title: "iOS Swift Documentation"
---

# Swift framework Version: 2.2.0

## Pre requisites

fi.notes framework supports iOS projects with minimum deployment target version 8 or above.

## Integration

You need to add cocoa pods to your project. You can find more information [here](https://guides.cocoapods.org/using/using-cocoapods.html).  
After integrating cocoa pods, add FinotesCore to your Podfile.

```bash
pod 'FinotesCoreSwift', '2.2.0'
```

Then install the same by executing the following command from terminal where your Podfile resides.  
Here the —repo-update is added incase your local cocoa pods repository is not up to date.
```bash
pod install --repo-update
```
###### The --repo-update option should be used the first time pod install is run from terminal. 

You can import the FinotesCoreSwift to your code using the import statement below.

```swift
import FinotesCoreSwift
```


## Initialize
You need to call the Fn.initialize(application) function in your appDelegate didFinishLaunchingWithOptions:

```swift
import FinotesCoreSwift

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application)
        return true
    }
    
```

#### DryRun
During development, you can set the dryRun mode, so that the issues raised will not be sent to the server. Every other feature except the issue sync to server will work as same.
```swift
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: true, withVerbose: false)
        return true
    }
```
#### VerboseLog
There are two variations of logging available in fi.notes, Verbose and Error. You can toggle them using corresponding APIs.  
Activating verbose will print all logs in LogCat including error and warning logs.
```swift
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: false, withVerbose: true)
        return true
    }
```
##### ErrorLog
If only error and warning logs needs to be printed,
```swift
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: false, withVerbose: false)
        Fn.logError(true)
        return true
    }
```
## Test
Now that the basic integration of fi.notes framework is complete,
Lets make sure that the dashboard and framework are in sync. 
```swift
import FinotesCoreSwift

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: false, withVerbose: true)
        Fn.reportIssue(atTarget: self, withDescription: "iOS Test Issue", withSeverity: MINOR)
        return true
    }
```

Now run the application in a simulator or real iOS device (with network connection).  
Once the application opens up, open [fi.notes dash](http://app.finotes.com/FinotesRS/#/tickets).    
The issue that we raised should be reported.   
In-case the issue is not listed, make sure the right app is selected at the top of the dashboard.  
Also, do refer to the logs in console, it will list any errors/warnings that might have occured during the integration.  

If the error still persists, do contact us at [fi.notes contact email](mailto:support@finotes.com) or you may use the chat support at the bottom right corner.  

#### You should remove the Fn.reportIssue() call, else every time the app is run, an issue will be reported.

```swift
import FinotesCoreSwift

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: false, withVerbose: true)
//        Fn.reportIssue(atTarget: self, withDescription: "iOS Test Issue", withSeverity: MINOR)
        return true
    }
```

### Listen for Issue
You can listen for and access every issue in realtime using the Fn.listenForIssues() API.  
You need to add the listener in your AppDelegate file.
```swift

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        Fn.initialize(application, withDryRun: false, withVerbose: true)
        Fn.listenForIssues(forSelector: #selector(issueReported(_:)), atTarget: self)
        return true
    }

    @objc func issueReported(_ issue:Issue){

    }
```
You will be provided with an Issue object that contains issue properties that are being synced to the server, making the whole process transparent.  
As this callback will be made right after an issue occurrence, you will be able to provide a positive message to the user depending on the severity of the issue.

## Reporting Issues

### Report Network Call Failure

Issues like status code errors, timeout issues and other failures will be reported for all network calls by fi.notes, from your application.  

If you are using URLSession with shared.dataTask then issues in all REST api calls will be reported automatically.
```swift
        let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
	    ...
        }
        task.resume()
```

##### Alamofire
Incase you are using Alamofire, then you need to add the below code in your NSURLSessionConfiguration.
The part we are interested is 
```objc
        let todoEndpoint: String = "https://hostname.com/path/to/resource"
        let configuration = URLSessionConfiguration.default
        
        configuration.protocolClasses = Fn.getProtocols(configuration.protocolClasses)
	
        let manager = Alamofire.SessionManager(configuration: configuration)
        manager.request(todoEndpoint, method: .get)
            .responseJSON { response in
        }
```

#### Whitelist Hosts
Add ObservableDomains array to the project info.plist with the list of domains to be observed, if needed, you may provide a timeout for the api calls along with the host names separated by a coma.

![Info.plist observable domains](/Screen%20Shot%202018-02-02%20at%2011.06.25%20AM.png)

Incase you are directly editing the info.plist as source-code

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ObservableDomains</key>
	<array>
		<string>host.com,5000</string>
		<string>another.host.com</string>
	</array>
```

Optionally, You may provide a custom timeout for hostnames. This will raise an Issue for any network call to the hosts, that takes more than the specified time for completion.  
Do note that specifying timeout will not interfere in any way with your network calls.

#### Categorize Tickets (Optional)

You will be able to configure how API issues are categorized into tickets using a custom header "X-URLID". You can set this header in your API calls and when they are reported to fi.notes dashboard, issues from API calls with same "X-URLID" will be categorized into a single ticket.  
```swift
    nsMutableURLRequest.setValue("loginapi", forHTTPHeaderField: "X-URLID")
```
Any issue in API calls with same X-URLID will be shown as a single ticket in fi.notes dashboard.


### Controller trail
When an issue is raised, inorder to get user screen flow for the current session, you need to extend your Controller from ObservableViewController.

```swift
class ViewController: UIViewController {

}
Changes to,

class ViewController: ObservableViewController {

}
```

##### Low Memory Reporting 
Extending Controller from ObservableViewController will also allow fi.notes to report any memory related issues that may occur in your application.

You may use App functions to report different app states along with Controller trail.

```swift
import FinotesCoreSwift


class AppDelegate: UIResponder, UIApplicationDelegate {

    func applicationWillResignActive(_ application: UIApplication) {
        App.applicationWillResignActive(application)
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
        App.applicationDidEnterBackground(application)
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        App.applicationWillEnterForeground(application)
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        App.applicationDidBecomeActive(application)
    }

    func applicationWillTerminate(_ application: UIApplication) {
        App.applicationWillTerminate(application)
    }

}
```
Once Controllers are extended from ObservableViewController and App functions are implemented, if an issue is raised, on fi.notes dashboard, you will be able to view screen activity for 3 minutes prior to the issue was raised. 
```
Activity Trail
	LoginController:viewDidLoad                  09:21:53:923
	LoginController:viewWillAppear               09:21:53:923
	LoginController:viewDidAppear                09:21:53:927
	UIApplication:applicationDidBecomeActive     09:21:54:081
	LoginController:viewDidLoad                  09:22:09:670
```

### Custom Breadcrumbs

You may add custom breadcrumbs to activity trail, which will be timestamped and shown in their execution order on fi.notes dashboard when an issue is raised.

```swift
        Fn.setBreadCrumb(atTarget: self, forCrumb: "completed login")
	...
        Fn.setBreadCrumb(atTarget: self, forCrumb: "sending a message")
```

### Function call
fi.notes will report any return value issues, exceptions and execution delays that may arise in functions using Fn.call().  
A regular function call will be,

```swift

    getUserNameFromDb("123-sd-12")
}

func getUserNameFromDb(_ userId:String) -> String?{
    return User.findById(userId).name
}

```
Function "getUserNameFromDb()" call needs to be changed to,  
You need to add @objc annoation before function definition.
```swift
import FinotesCoreSwift

    Fn.call(withSelector: #selector(getUserNameFromDb(_:)), withTarget: self, 
    							withParameters:"123-sd-12")
}

@objc func getUserNameFromDb(_ userId:String) -> String?{
    return User.findById(userId).name
}
```

This will allow fi.notes to raise issue incase the function take more than normal time to execute, or if the function return a nil value, or throws an exception.

You can control all the above said parameters in Observe object.

##### expectedExecutionTime

```swift
import FinotesCoreSwift

    let observer : Observer = Fn.observe()
    observer.expectedExecutionTime(1400)
    Fn.call(withSelector: #selector(getUserNameFromDb(_:)), withTarget: self, 
    					withObserver:observer, withParameters:"123-sd-12")
}

@objc func getUserNameFromDb(_ userId:String) -> String?{
    return User.findById(userId).name
}

```
##### Returns nil
##### Exception in function
Here the expectedExecutionTime for the function "getUserNameFromDb()" has been overriden to 1400 milliseconds (default was 1000 milliseconds). If the database query takes more than 1400 milliseconds to return the value or if the returned "userName" is nil, then corresponding issues will be raised.  
An issue will be raised when exception occurs inside a function that is called using Fn.call().


#### Function in Separate Class file

If the function is defined in a separate file and needs to be invoked, then instead of passing "self" you can pass the corresponding object in Fn.call()
```swift
import FinotesCoreSwift


    let dbUtils = DBUtils()

    let observer : Observer = Fn.observe()
    observer.expectedExecutionTime(1400)
    let userName :String? = Fn.call(withSelector: #selector(customObject.getUserNameFromDb(_:)), 
    					 withTarget: customObject,
                                         withObserver:observer ,
                                         withParameters:"hello") as! String?
}

public class DBUtils: NSObject {
    @objc public func getUserNameFromDb(_ userId:String) -> String?{
        return User.findById(userId).name
    }
}
```

#### Static Function calls

For static functions, pass corresponding class instead of object in Fn.call() to invoke static method.

```swift
import FinotesCoreSwift


        let userTimeStamp :CLongLong = Fn.call(withSelector: #selector(ViewController.getUserTimestampFromJSON),
                withTarget: ViewController.classForCoder()) as! CLongLong
}

@objc static func getUserTimestampFromJSON() -> CLongLong{
	return processJsonResponse()
}
```

### Chained Function calls

You can connect multiple functions using 'nextFunctionSignature' and 'inClass' properties in Observe object.

```swift
import FinotesCoreSwift

    @objc func sendChatClicked(sender: UITapGestureRecognizer) {
    
        let observer : Observer = Fn.observe()
        observer.expectedChainedExecutionTime(2000)
        observer.nextFunctionSignature(#selector(onChatSent(_:)), in: ViewController.classForCoder())

	Fn.call(withSelector: #selector(sendChat(_:)), withTarget: self, withObserver:observer, 
								withParameters: chatMessage)
    }
    
    @objc func sendChat(_ message:String) -> Bool {
    	if(isValid(message)){
	      return true
	}
        return false
    }

    @objc func onChatSent(_ chatId:String) {
    	chatSyncConfirmed(chatId)
    }
```

Here 'onChatSent()' should be called within 2000 milliseconds after execution of 'sendChat()'. If the function 'onChatSent()' is not called or is delayed then corresponding issues will be raised and reported.

##### boolean returns false
Here in 'sendChat()' function, if it returns false, an issue will be raised with the function parameters, which will help you dig more into what could have gone wrong.

#### Test
In-order to check if you have implemented function chaining correctly, set break points inside the chained functions then run application in your simulator or device, and use your application, if you have implemented correctly, You will hit these breakpoints correctly.

### Catch Global Exceptions.
In-order to catch uncaught exceptions, You may use Fn.catchExceptions().
```swift
import FinotesCoreSwift

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
	Fn.initialize(application, withDryRun: false, withVerbose: true)
        Fn.catchExceptions()
        return true
}
```

### Custom Issue
You can report custom issues using the Fn.reportIssue() API.
```swift
//Payment gateway delegate methods.
func paymentCompleted(_ userIdentifier:String, _ type:String){

}

func paymentFailed(_ reason:String, _ userId:String){
	Fn.reportIssue(atTarget: self, withDescription: String(format: "Payment failed for %@",reason)
										, withSeverity: FATAL)
}
```

























