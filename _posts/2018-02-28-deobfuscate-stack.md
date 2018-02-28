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
