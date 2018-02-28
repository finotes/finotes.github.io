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

### Using Graphical User Interface

1. You need to open proguardgui.bat (for Windows users) or proguardgui.sh for (Mac or Linux users).  
2. proguardgui is located in path-to-sdk/tools/proguard/bin/  
3. Execute proguardgui.sh using './proguardgui.sh' for Linux and Mac users from the terminal.  

4. Go to the "retrace" section on the left side Proguard GUI tool.  
5. Set the path to your mapping file generated (automatically) during the release build, usually located at build/outputs/proguard/release/mapping.txt on your application module folder.  
6. Copy stacktrace from the fi.notes dashboard and paste it inside box thats titled "obfuscated stack trace".  

7. Click on "Retrace" button.  


### Using Terminal

1. Copy the stacktrace from fi.notes dashboard to a file "stacktrace.txt".
2. Execute retrace.bat /path/to/mapping.txt /path/to/stacktrace.txt  > deobfuscate.txt
3. Use ./retrace.sh for Linux and Mac users.
4. The de-obfuscated stacktrace will be written to the file deobfuscate.txt.

