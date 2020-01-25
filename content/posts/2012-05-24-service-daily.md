---
layout: post
title: Execute Android Service daily when Internet connection is available
author: Dominik Schürmann
date: 2012-05-24
slug: service-daily
aliases:
    - "/2012/05/24/service-daily.html"
---

__This tutorial is a solution for the following scenario:__ 
Your application needs a service that is executed daily and polls a server for updates. The user should only have the simple preferences to enable/disable that background service and to restrict it to Wifi connections. In my opinion you shouldn't overload the user with detailed options. Mobile applications are popular because they are mostly easier to use than desktop applications. Thus keep it simple!

The first seemingly easy solution that comes to mind is to use Androids AlarmManager to set up a repeating event that executes this service when triggered. There are several pitfalls with this approach:

  * __What to do if Android is currently in sleeping mode (screen is black)?__  
      Solution: You need a wakelock that locks the CPU before the service is executed!
  * __When to register the Alarm with Androids AlarmManager?__
      Solution: On enabling of the preference for the background service, after booting Android (because Android forgets about all events after rebooting), and (to ensure that it really registers at some time) on the start of your application.
  * __What happens when there is no Internet available at the time when my event is triggered?__  
      This is a very problematic situation. My solution to this problem is to enable a BroadcastReceiver which is triggered when the connectivity in Android changes. Then when Internet is available the background service is started and the receiver disables itself. See the following code examples for details.

## Implement the background service
To simplify the handling of the wakelock and the registering of the event on boot, I propose using [CommonsWare's WakefulIntentService](https://github.com/commonsguy/cwac-wakeful). So copy the src folder into your project to get AlarmReceiver.java and WakefulIntentService.java.

Now create a new class as follows:

```java
public class BackgroundService extends WakefulIntentService {
    
    public BackgroundService() {
        super("BackgroundService");
    }

    /**
     * Asynchronous background operations of service, with wakelock
     */
    @Override
    public void doWakefulWork(Intent intent) {
       // your code here...
    }
}
```

## AlarmListener
Now we need to implement CommonsWare's AlarmListener:

```java
public class DailyListener implements AlarmListener {
    public void scheduleAlarms(AlarmManager mgr, PendingIntent pi, Context context) {
        // register when enabled in preferences
        if (PreferenceHelper.getUpdateCheckDaily(context)) {
            Log.i("DailyListener", "Schedule update check...");

            // every day at 9 am
            Calendar calendar = Calendar.getInstance();
            // if it's after or equal 9 am schedule for next day
            if (Calendar.getInstance().get(Calendar.HOUR_OF_DAY) &gt;= 9) {
                calendar.add(Calendar.DAY_OF_YEAR, 1); // add, not set!
            }
            calendar.set(Calendar.HOUR_OF_DAY, 9);
            calendar.set(Calendar.MINUTE, 0);
            calendar.set(Calendar.SECOND, 0);

            mgr.setInexactRepeating(AlarmManager.RTC, calendar.getTimeInMillis(),
                        AlarmManager.INTERVAL_DAY, pi);
        }
    }

    public void sendWakefulWork(Context context) {
        ConnectivityManager cm = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo netInfo = cm.getActiveNetworkInfo();

        // only when connected or while connecting...
        if (netInfo != null && netInfo.isConnectedOrConnecting()) {

            boolean updateOnlyOnWifi = PreferenceHelper.getUpdateOnlyOnWifi(context);

            // if we have mobile or wifi connectivity...
            if (((netInfo.getType() == ConnectivityManager.TYPE_MOBILE) && updateOnlyOnWifi == false)
                    || (netInfo.getType() == ConnectivityManager.TYPE_WIFI)) {
                Log.d("DailyListener", "We have internet, start update check directly now!");

                Intent backgroundIntent = new Intent(context, BackgroundService.class);
                WakefulIntentService.sendWakefulWork(context, backgroundIntent);
            } else {
                Log.d("DailyListener", "We have no internet, enable ConnectivityReceiver!");

                // enable receiver to schedule update when internet is available!
                ConnectivityReceiver.enableReceiver(context);
            }
        } else {
            Log.d("DailyListener", "We have no internet, enable ConnectivityReceiver!");

            // enable receiver to schedule update when internet is available!
            ConnectivityReceiver.enableReceiver(context);
        }
    }

    public long getMaxAge() {
        return (AlarmManager.INTERVAL_DAY + 60 * 1000);
    }
}
```
You need to implement a method to query your preferences by yourself. I have a helper class named PreferenceHelper for this. Replace that code with your methods.

## ConnectivityReceiver
What is also missing is the ConnectivityReceiver:

```java
public class ConnectivityReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(ConnectivityManager.CONNECTIVITY_ACTION)) {
            Log.d("ConnectivityReceiver", "ConnectivityReceiver invoked...");

            // only when background update is enabled in prefs
            if (PreferenceHelper.getUpdateCheckDaily(context)) {
                Log.d("ConnectivityReceiver", "Update check daily is enabled!");

                boolean noConnectivity = intent.getBooleanExtra(
                        ConnectivityManager.EXTRA_NO_CONNECTIVITY, false);

                if (!noConnectivity) {

                    ConnectivityManager cm = (ConnectivityManager) context
                            .getSystemService(Context.CONNECTIVITY_SERVICE);
                    NetworkInfo netInfo = cm.getActiveNetworkInfo();

                    // only when connected or while connecting...
                    if (netInfo != null && netInfo.isConnectedOrConnecting()) {

                        boolean updateOnlyOnWifi = PreferenceHelper.getUpdateOnlyOnWifi(context);

                        // if we have mobile or wifi connectivity...
                        if (((netInfo.getType() == ConnectivityManager.TYPE_MOBILE) && updateOnlyOnWifi == false)
                                || (netInfo.getType() == ConnectivityManager.TYPE_WIFI)) {
                            Log.d("ConnectivityReceiver", "We have internet, start update check and disable receiver!");

                            // Start service with wakelock by using WakefulIntentService
                            Intent backgroundIntent = new Intent(context, BackgroundService.class);
                            WakefulIntentService.sendWakefulWork(context, backgroundIntent);

                            // disable receiver after we started the service
                            disableReceiver(context);
                        }
                    }
                }
            }
        }
    }

    /**
     * Enables ConnectivityReceiver
     * 
     * @param context
     */
    public static void enableReceiver(Context context) {
        ComponentName component = new ComponentName(context, ConnectivityReceiver.class);

        context.getPackageManager().setComponentEnabledSetting(component,
                PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP);
    }

    /**
     * Disables ConnectivityReceiver
     * 
     * @param context
     */
    public static void disableReceiver(Context context) {
        ComponentName component = new ComponentName(context, ConnectivityReceiver.class);

        context.getPackageManager().setComponentEnabledSetting(component,
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED, PackageManager.DONT_KILL_APP);
    }
}
```

## XML configurations
Now we need to register all that in your AndroidManifest:

```xml
<service android:name=".BackgroundService" />
 
<receiver android:name="com.commonsware.cwac.wakeful.AlarmReceiver" >
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
 
    <meta-data
        android:name="com.commonsware.cwac.wakeful"
        android:resource="@xml/wakeful" />
</receiver>
<receiver
    android:name=".ConnectivityReceiver"
    android:enabled="false" >
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
```

You need the following permissions in your Manifest:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

The following is needed for CommonsWare's method to register the right Listener. Save it as wakeful.xml in res/xml and adopt it to your package name.

```xml
<WakefulIntentService listener="your.packagename.DailyListener" />
```

In the last step you need to place the following code in onCreate of your default Activity and also execute it when the background service is enabled by the user in the preference screen.

```java
WakefulIntentService.scheduleAlarms(new DailyListener(), this, false);
```

This method of building a background service is somewhat complicated but energy efficient. The BackgroundService is executed once a day, but if no Internet connection is available the ConnectivityReceiver gets activated. By using this method there is no need to check regularly for Internet connection from your service. Additionally the ConnectivityReceiver is not active all the time. It gets only activated on purpose.
