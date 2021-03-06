# Flutter Local Notifications Plugin

[![pub package](https://img.shields.io/pub/v/flutter_local_notifications.svg)](https://pub.dartlang.org/packages/flutter_local_notifications)

A cross platform plugin for displaying local notifications. 

## Supported Platforms
* Android API 16+ (4.1+, the minimum version supported by Flutter). Uses the NotificationCompat APIs so it can be run older Android devices.
* iOS 8.0+ (the minimum version supported by Flutter). Supports the old and new iOS notification APIs (the User Notifications Framework introduced in iOS 10 but will use the UILocalNotification APIs for devices predating iOS 10)

## Features

* Mockable (plugin and API methods aren't static)
* Display basic notifications
* Scheduling when notifications should appear
* Periodically show a notification (interval based)
* Schedule a notification to be shown daily at a specified time
* Schedule a notification to be shown weekly on a specified day and time
* Cancelling/removing notification by id or all of them
* Specify a custom notification sound
* Ability to handle when a user has tapped on a notification, when the app is the foreground, background or terminated
* Determine if an app was launched due to tapping on a notification
* [Android] Configuring the importance level
* [Android] Configuring the priority
* [Android] Customising the vibration pattern for notifications
* [Android] Configure the default icon for all notifications
* [Android] Configure the icon for each notification (overrides the default when specified)
* [Android] Configure the large icon for each notification. The icon can be a drawable or a file on the device
* [Android] Formatting notification content via HTML markup (see https://developer.android.com/guide/topics/resources/string-resource.html#StylingWithHTML)
* [Android] Support for the following notification styles
    * Big picture
    * Big text
    * Inbox
* [Android] Group notifications
* [Android] Show progress notifications
* [iOS] Customise the permissions to be requested around displaying notifications

Note that this plugin aims to provide abstractions for all platforms as opposed to having methods that only work on specific platforms. However, each method allows passing in "platform-specifics" that contains data that is specific for customising notifications on each platform. It is still under development so expect the API surface to change over time.

**IMPORTANT**: Recurring notifications on Android use the [Alarm Manager](https://developer.android.com/reference/android/app/AlarmManager) API. This is standard practice but does mean the delivery of the notifications/alarms are inexact and this is documented Android behaviour as per the previous link.


## Acknowledgements

* [Javier Lecuona](https://github.com/javiercbk) for submitting the PR that added the ability to have notifications shown daily
* [Jeff Scaturro](https://github.com/JeffScaturro) for submitting the PR to fix the iOS issue around showing daily and weekly notifications
* [Ian Cavanaugh](https://github.com/icavanaugh95) for helping create a sample to reproduce the problem reported in [issue #88](https://github.com/MaikuB/flutter_local_notifications/issues/88)

## Raising issues and contributions

If you run into issues, please raise them on the GitHub repository. Please do not email them to me as I will be ignoring emails going forward as GitHub is the appropriate place for them. It would also be much appreciated if they could be limited to actual bugs or feature requests. If you're looking at how you could use the plugin to do a particular kind of notification, check the example app provides detailed code samples for each supported feature. Also try to check the README first in case you have missed something e.g. platform-specific setup.

Contributions are welcome by submitting a PR for me to review. If it's to add new features, appreciate it if you could try to maintain the architecture or try to improve on it. However, do note that I will not take PRs that add methods at the Dart level that don't work on all platforms. However, platform-specific configuration through the use parameters are fine as that's approach being taken via this plugin.

## Getting Started

The GitHub repository has an example app that should demonstrate of all the supported features of the plugin. Please check the example for more detailed code samples. If you only copy and paste the Dart code then this will not work as there's setup required for each platform. Pub also generates API docs for the latest version [here](https://pub.dartlang.org/documentation/flutter_local_notifications/latest/)

The following samples will demonstrate the more commonly used functionalities. The first step is to create a new instance of the plugin class and then initialise it with the settings to use for each platform

```
FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin = new FlutterLocalNotificationsPlugin();
var initializationSettingsAndroid =
    new AndroidInitializationSettings('app_icon');
var initializationSettingsIOS = new IOSInitializationSettings();
var initializationSettings = new InitializationSettings(
    initializationSettingsAndroid, initializationSettingsIOS);
flutterLocalNotificationsPlugin = new FlutterLocalNotificationsPlugin();
flutterLocalNotificationsPlugin.initialize(initializationSettings,
    selectNotification: onSelectNotification);
```

Here we specify we have specified the default icon to use for notifications on Android (refer to the Android Integration section) and designated the function (onSelectNotification) that should fire when a notification has been tapped on. Specifying this callback is entirely optional. In this example, it will trigger navigation to another page and display the payload associated with the notification. 

```
Future onSelectNotification(String payload) async {
    if (payload != null) {
      debugPrint('notification payload: ' + payload);
    }
    await Navigator.push(
      context,
      new MaterialPageRoute(builder: (context) => new SecondScreen(payload)),
    );
}
```

In the real world, this payload could represent the id of the item you want to display the details of. Once the initialisation has been done, then you can manage the displaying of notifications.

### Displaying a notification

```
var androidPlatformChannelSpecifics = new AndroidNotificationDetails(
    'your channel id', 'your channel name', 'your channel description',
    importance: Importance.Max, priority: Priority.High);
var iOSPlatformChannelSpecifics = new IOSNotificationDetails();
var platformChannelSpecifics = new NotificationDetails(
    androidPlatformChannelSpecifics, iOSPlatformChannelSpecifics);
await flutterLocalNotificationsPlugin.show(
    0, 'plain title', 'plain body', platformChannelSpecifics,
    payload: 'item id 2');
```

In this block of code, the details for each platform have been specified. This includes the channel details that is required for Android 8.0+. The payload has been specified ('item id 2'), that will passed back through your application when the user has tapped on a notification. Note that for Android devices that notifications will only in appear in the tray and won't appear as a toast aka heads-up notification unless things like the priority/importance has been set appropriately. Refer to the Android docs (https://developer.android.com/guide/topics/ui/notifiers/notifications.html#Heads-up) for additional information.

### Scheduling a notification

```
var scheduledNotificationDateTime =
        new DateTime.now().add(new Duration(seconds: 5));
var androidPlatformChannelSpecifics =
    new AndroidNotificationDetails('your other channel id',
        'your other channel name', 'your other channel description');
var iOSPlatformChannelSpecifics =
    new IOSNotificationDetails();
NotificationDetails platformChannelSpecifics = new NotificationDetails(
    androidPlatformChannelSpecifics, iOSPlatformChannelSpecifics);
await flutterLocalNotificationsPlugin.schedule(
    0,
    'scheduled title',
    'scheduled body',
    scheduledNotificationDateTime,
    platformChannelSpecifics);
```

### Periodically show a notification with a specified interval

```
// Show a notification every minute with the first appearance happening a minute after invoking the method
var androidPlatformChannelSpecifics =
    new AndroidNotificationDetails('repeating channel id',
        'repeating channel name', 'repeating description');
var iOSPlatformChannelSpecifics =
    new IOSNotificationDetails();
var platformChannelSpecifics = new NotificationDetails(
    androidPlatformChannelSpecifics, iOSPlatformChannelSpecifics);
await flutterLocalNotificationsPlugin.periodicallyShow(0, 'repeating title',
    'repeating body', RepeatInterval.EveryMinute, platformChannelSpecifics);
```

### Show a daily notification at a specific time

```
var time = new Time(10, 0, 0);
var androidPlatformChannelSpecifics =
    new AndroidNotificationDetails('repeatDailyAtTime channel id',
        'repeatDailyAtTime channel name', 'repeatDailyAtTime description');
var iOSPlatformChannelSpecifics =
    new IOSNotificationDetails();
var platformChannelSpecifics = new NotificationDetails(
    androidPlatformChannelSpecifics, iOSPlatformChannelSpecifics);
await flutterLocalNotificationsPlugin.showDailyAtTime(
    0,
    'show daily title',
    'Daily notification shown at approximately ${_toTwoDigitString(time.hour)}:${_toTwoDigitString(time.minute)}:${_toTwoDigitString(time.second)}',
    time,
    platformChannelSpecifics);
```

### Show a weekly notification on specific day and time

```
var time = new Time(10, 0, 0);
var androidPlatformChannelSpecifics =
    new AndroidNotificationDetails('show weekly channel id',
        'show weekly channel name', 'show weekly description');
var iOSPlatformChannelSpecifics =
    new IOSNotificationDetails();
var platformChannelSpecifics = new NotificationDetails(
    androidPlatformChannelSpecifics, iOSPlatformChannelSpecifics);
await flutterLocalNotificationsPlugin.showWeeklyAtDayAndTime(
    0,
    'show weekly title',
    'Weekly notification shown on Monday at approximately ${_toTwoDigitString(time.hour)}:${_toTwoDigitString(time.minute)}:${_toTwoDigitString(time.second)}',
    Day.Monday,
    time,
    platformChannelSpecifics);
```

### [Android only] Grouping notifications

This is a "translation" of the sample available at https://developer.android.com/training/notify-user/group.html
For iOS, you could just display the summary notification (not shown in the example) as otherwise the following code would show three notifications 

```
String groupKey = 'com.android.example.WORK_EMAIL';
String groupChannelId = 'grouped channel id';
String groupChannelName = 'grouped channel name';
String groupChannelDescription = 'grouped channel description';
// example based on https://developer.android.com/training/notify-user/group.html
AndroidNotificationDetails firstNotificationAndroidSpecifics =
    new AndroidNotificationDetails(
        groupChannelId, groupChannelName, groupChannelDescription,
        importance: Importance.Max,
        priority: Priority.High,
        groupKey: groupKey);
NotificationDetails firstNotificationPlatformSpecifics =
    new NotificationDetails(firstNotificationAndroidSpecifics, null);
await flutterLocalNotificationsPlugin.show(1, 'Alex Faarborg',
    'You will not believe...', firstNotificationPlatformSpecifics);
AndroidNotificationDetails secondNotificationAndroidSpecifics =
    new AndroidNotificationDetails(
        groupChannelId, groupChannelName, groupChannelDescription,
        importance: Importance.Max,
        priority: Priority.High,
        groupKey: groupKey);
NotificationDetails secondNotificationPlatformSpecifics =
    new NotificationDetails(secondNotificationAndroidSpecifics, null);
await flutterLocalNotificationsPlugin.show(
    2,
    'Jeff Chang',
    'Please join us to celebrate the...',
    secondNotificationPlatformSpecifics);

// create the summary notification required for older devices that pre-date Android 7.0 (API level 24)
List<String> lines = new List<String>();
lines.add('Alex Faarborg  Check this out');
lines.add('Jeff Chang    Launch Party');
InboxStyleInformation inboxStyleInformation = new InboxStyleInformation(
    lines,
    contentTitle: '2 new messages',
    summaryText: 'janedoe@example.com');
AndroidNotificationDetails androidPlatformChannelSpecifics =
    new AndroidNotificationDetails(
        groupChannelId, groupChannelName, groupChannelDescription,
        style: NotificationStyleAndroid.Inbox,
        styleInformation: inboxStyleInformation,
        groupKey: groupKey,
        setAsGroupSummary: true);
NotificationDetails platformChannelSpecifics =
    new NotificationDetails(androidPlatformChannelSpecifics, null);
await flutterLocalNotificationsPlugin.show(
    3, 'Attention', 'Two new messages', platformChannelSpecifics);
```

### Cancelling/deleting a notification

```
// cancel the notification with id value of zero
await flutterLocalNotificationsPlugin.cancel(0);
```

### Cancelling/deleting all notifications

```
await flutterLocalNotificationsPlugin.cancelAll();
```


### Get details on if the app was launched via a notification
```
 var notificationAppLaunchDetails =
     await flutterLocalNotificationsPlugin.getNotificationAppLaunchDetails();
```

This should cover the basic functionality. Please check out the `example` directory for a sample app that illustrates the rest of the functionality available and refer to the API docs for more information. Also read the below on what you need to configure on each platform

## Android Integration

If your application needs the ability to schedule notifications then you need to request permissions to be notified when the phone has been booted as scheduled notifications uses AlarmManager to determine when notifications should be displayed. However, they are cleared when a phone has been turned off. Requesting permission requires adding the following to the manifest

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

Developers will also need to add the following so that plugin can handle displaying scheduled notifications and reschedule notifications upon a reboot

```xml
<receiver android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationReceiver" />
<receiver android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationBootReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"></action>
    </intent-filter>
</receiver>
```
If the vibration pattern of an Android notification will be customised then add the following

```xml
<uses-permission android:name="android.permission.VIBRATE" />
```

Notification icons should be added as a drawable resource. The example project/code shows how to set default icon for all notifications and how to specify one for each notification. It is possible to use launcher icon/mipmap and this by default is `@mipmap/ic_launcher` in the Android manifest and can be passed `AndroidInitializationSettings` constructor. However, the offical Android guidance is that you should use drawable resources. Custom notification sounds should be added as a raw resource and the sample illustrates how to play a notification with a custom sound. Refer to the following links around Android resources and notification icons.

 * https://developer.android.com/guide/topics/resources/providing-resources
 * https://developer.android.com/studio/write/image-asset-studio#notification
 * https://developer.android.com/guide/practices/ui_guidelines/icon_design_status_bar

When specifying the large icon bitmap or big picture bitmap (associated with the big picture style), bitmaps can be either a drawable resource or file on the device. This is specified via a single property (e.g. the `largeIcon` property associated with the `AndroidNotificationDetails` class) and there will be a corresponding property of the `BitmapSource` enum type (e.g. `largeIconBitmapSource`) that indicates if the string value represents the name of the drawable resource or the path to the bitmap file.

Note that with Android 8.0+, sounds and vibrations are associated with notification channels and can only be configured when they are first created. Showing/scheduling a notification will create a channel with the specified id if it doesn't exist already. If another notification specifies the same channel id but tries to specify another sound or vibration pattern then nothing occurs.

If you run into error messages around the `com.android.support:support-compat` library due to version conflicts, you can try adding the following to the `build.gradle` file of your Android head project as reported by another dev [here](https://github.com/MaikuB/flutter_local_notifications/issues/5)

```
allprojects {
    repositories {
       //...
    }
    subprojects {
        project.configurations.all {
            resolutionStrategy.eachDependency { details ->
                if (details.requested.group == 'com.android.support'
                        && !details.requested.name.contains('multidex') ) {
                    details.useVersion "27.1.0"
                }
            }
        }
    }
}
```

Note though this will force other plugins to use the same version of the library that this plugin depends on so may not be desirable, particularly if they use a more recent version than 27.1. If you have another suggestion on how to solve this please do let me know :)

When doing a release build of your app, you'll likely need to customise your ProGuard configuration file as per this [link](https://developer.android.com/studio/build/shrink-code#keep-code) and add the following line

```
-keep class com.dexterous.** { *; }
```

## iOS Integration

By design, iOS applications do not display notifications when they're in the foreground. For iOS 10+, use the presentation options to control the behaviour for when a notification is triggered while the app is in the foreground. For older versions of iOS, you will need update the AppDelegate class to handle when a local notification is received to display an alert. This is shown in the sample app within the `didReceiveLocalNotification` method of the `AppDelegate` class. The notification title can be found by looking up the `title` within the `userInfo` dictionary of the `UILocalNotification` object

```
#import <flutter_local_notifications/FlutterLocalNotificationsPlugin.h>

...

- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
    if(@available(iOS 10.0, *)) {
        return;
    }
    
    NSString *payload = notification.userInfo[@"payload"];
    if(FlutterLocalNotificationsPlugin.resumingFromBackground) {
        // resuming from the background so don't want to show an alert as we would've seen
        // the notification while the app was in the background
        [FlutterLocalNotificationsPlugin handleSelectNotification:payload];
        return;
    }
    
    // display the alert as the app was in the foreground so notification wouldn't be displayed.
    // when the user taps on OK, fire the code in our Flutter app that is responsible for handling
    // the action for when the user taps on a notification
    NSString *title = notification.userInfo[@"title"];
    UIAlertController* alert = [UIAlertController alertControllerWithTitle:title
                                                                   message:notification.alertBody
                                                            preferredStyle:UIAlertControllerStyleAlert];
    
    UIAlertAction* defaultAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault
                                                          handler:^(UIAlertAction * action) {
                                                              [FlutterLocalNotificationsPlugin handleSelectNotification:payload];
                                                          }];
    
    [alert addAction:defaultAction];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alert animated:YES completion:nil];
}
```

In theory, it should be possible for the plugin to handle this but this the method doesn't seem to fire. The Flutter team has acknowledged that the method hasn't been wired up to enable this https://github.com/flutter/flutter/issues/16662

Also if you have set notifications to be periodically shown, then on older iOS versions (< 10), if the application was uninstalled without cancelling all alarms then the next time it's installed you may see the "old" notifications being fired. If this is not the desired behaviour, then you can add the following to the `didFinishLaunchingWithOptions` method of your `AppDelegate` class.

```
if(![[NSUserDefaults standardUserDefaults]objectForKey:@"Notification"]){
    [[UIApplication sharedApplication] cancelAllLocalNotifications];
    [[NSUserDefaults standardUserDefaults]setBool:YES forKey:@"Notification"];
}
```

When using custom notification sound, developers should be aware that iOS enforces restrictions on this (e.g. supported file formats). As of this writing, this is documented by Apple at

https://developer.apple.com/documentation/usernotifications/unnotificationsound?language=objc

**NOTE**: this plugin registers itself as the delegate to handle incoming notifications and actions. This may cause problems if you're using other plugins for push notifications (e.g. `firebase_messaging`) as they will most likely do the same and it's only possible to register a single delegate. iOS handles showing push notifications out of the box so if you're only using this plugin to display the notification payload on Android then it's suggested that you fork the plugin code and remove the following part in the iOS code

```
UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
center.delegate = instance;
```

Unfortunately, this platform limitation does mean that it's not possible to use this plugin together other plugins for push notifications on iOS. If you are in this situation, then my only advice is that you'll need to need to look at writing customised platform-specific code for your application that may involve taking bits and pieces of code from the plugins you need. 

## Testing

As the plugin class is not static, it is possible to mock and verify it's behaviour when writing tests as part of your application. Check the source code for a sample test suite can be found at _test/flutter_local_notifications_test.dart_ that demonstrates how this can be done.
