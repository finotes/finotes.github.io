## Decode ProGuard’s Obfuscated Code From Stack Trace

### Before

```bash
    com.finotes.android.be.u(Unknown Source)
    com.finotes.android.we.b(Unknown Source)
    android.app.Activity.performCreate(Activity.java:6975)
    android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1213)
    android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2770)
    android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2892)
    android.app.ActivityThread.-wrap11(Unknown Source:0)
```

### After

```bash
    com.finotes.android.MainActivity.getUserDetails(MainActivity.java:204)
    com.finotes.android.MainActivity.onCreate(MainActivity.java:21)
    android.app.Activity.performCreate(Activity.java:6975)
    android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1213)
    android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2770)
    android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2892)
    android.app.ActivityThread.-wrap11(Unknown Source:0)
```

### Steps to de-obfuscate stacktrace.

There are 2 possible methods to de-obfuscate stacktrace.

### Graphical User Interface

You need to open proguardgui.bat (for Windows users) or proguardgui.sh for (Mac or Linux users).  
proguardgui.bat is located in path-to-sdk/tools/proguard/bin/  
Execute ./proguardgui.sh for Linux and Mac users from the terminal.  

Go to the "retrace" section on the left side Proguard GUI tool.  
Set the path to your mapping file generated (automatically) during the release build, usually located at build/outputs/proguard/release/mapping.txt on your application module folder.  
Copy stacktrace from the fi.notes dashboard and paste it inside box thats titled "obfuscated stack trace".  

Click on "Retrace" button.  



