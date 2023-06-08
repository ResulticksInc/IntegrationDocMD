# Flutter Integration document

## Install the flutter plugin

- Add the plugin via the terminal using the following syntax

```ruby
flutter pub add refluttersdk
```
## Flutter native android setup

The section details information on initializing the SDK.

- Create an application class as mentioned below:

```ruby
import android.app.Application
import com.resul.refluttersdk.RefluttersdkPlugin
class ProjectApplication: Application() { 
      override fun onCreate() {
      super.onCreate()
      RefluttersdkPlugin().initReSdk(this)
    } 
}
```

>Note Add the **‘ProjectApplication’** class to the AndroidMainfest.xml file.

### Notification message delivery
- Create anFCM project to proceed with the link given herewith - https://firebase.google.com/docs/cloud-messaging/android/client.
- Add Firebase dependencies in the **‘app/build.gradle.’**

```ruby
implementation 'com.google.firebase:firebase-messaging:20.1.0' 
apply plugin: 'com.google.gms.google-services'
```
- Add Firebase dependencies in the **‘root/build.gradle.’**

```ruby
classpath 'com.google.gms:google-services:4.3.3'
```
- Create a new FCM Receiver class extension with **‘FirebaseMessagingService’** and override the **‘onMessageReceived’** method in the class.

```ruby
  import com.resul.refluttersdk.RefluttersdkPlugin 
  class MessagingService:FirebaseMessagingService () {
      override fun onMessageReceived(remoteMessage: RemoteMessage) { 
      super.onMessageReceived(remoteMessage)
      if(RefluttersdkPlugin().onReceivedCampaign(remoteMessage,this)) {
          return; 
       }
     }
  }
```
- Create the **‘FCM Receiver’** class and connect to the **Androidmainfeast.xml**file.
```ruby
<!--The receiver receives FCM notification --> 
<service
android:name="<Firebase receiver class>"
android:exported="false" android:directBootAware="true">
Copyright © RESULTICKS Solution Inc 4
<intent-filter>
<action android:name="com.google.firebase.MESSAGING_EVENT"/>
   </intent-filter>
</service>
```
The **‘onMessageReceived’** method handles the presentation of Resulticks notifications, the In-app notifications, and the activity level screen navigation. Pass through the custom parameters and the fragment name through the activity bundle intent.

- **In-App notifications - FlutterActivity change to FlutterFragmentActivity**

```ruby
import io.flutter.embedding.android.FlutterFragmentActivity
class MainActivity : FlutterFragmentActivity()
```

### Deeplinking/Deferreddeeplinking

For the mobile app, deep linking consists of using a uniform resource identifier (URI) that links to a specific location within a mobile app rather than launching the app.

- Paste the deep linking **AndroidManifest.xml** inside the intentfilter. Refer to an example given below

```ruby
<intent-filter android:autoVerify="true">
<action android:name="android.intent.action.VIEW" /> 
<category android:name="android.intent.category.DEFAULT" /> 
<category android:name="android.intent.category.BROWSABLE" />
<data
android:host="<host>" android:scheme="<scheme>"/>
</intent-filter>
```

### Appconfiguration

The app configuration relates to a set of configuration settings applicable for an application/app to work well as per the functions.

Perform the listed steps for configuring the app

- Paste the metadata **AndroidManifest.xml** inside the application tag.

```ruby
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.res.refluttersdk">
<!-- Application Class-->
<application 
    android:name="<application class name>" 
    android:icon="@mipmap/ic_launcher" 
    android:label="@string/app_name" 
    android:theme="@style/AppTheme">
 
 <!-- Set customized notification icon -->
<meta-data android:name="resulticks.default_notification_icon"
android:resource="@drawable/<Notification-Icon>"/>

 <!-- Set customized notification transparent icon -->
<meta-data android:name="resulticks.default_notification_icon_transparent"
android:resource="@drawable/<Notification-Icon>"/>

 <!-- Set customized notification color -->
<meta-data android:name="resulticks.default_notification_color"
android:resource="@color/<Notification_color "/>

<!-- Setting global variable for Resulticks API key --> 
<meta-data
android:name="resulticks.key"
android:value="api_key_<App Id>" />
</application>
```

## iOS – Native code setup

### InitializingtheSDK

Perform the steps listed to set up the native iOS code
- Initialize the SDK.

```swift
RefluttersdkPlugin().initWithSDK(appId:"<app_id>")
```
**Sample - ForiOS ver.13 and above**

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
      if let window = window,
      let controller = _window.rootViewController as? FlutterViewController {
      ReffluttersdkPlugin().initWithSDK(appId:"<app_id>")
   } 
 }
```
### Notification setup

**Method - setNotificationAction**

Use the ‘setNotificationAction’ method to get the “Notification open” (User engagement) and call to action (CTA) of the respective buttons from the notification message.

```swift
public funcuserNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandlercompletionHandler: @escaping () -> Void) {
   RefluttersdkPlugin.didReceive(response: response)
}
```

**Method – setCustomNotification**

Use the method to send silent notification data for In-app notification

```swift
func application(_ application: UIApplication, didReceiveRemoteNotificationuserInfo: [AnyHashable : Any], fetchCompletionHandlercompletionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    if let _userInfo = userInfo as? [String: Any] {
      RefluttersdkPlugin.didReceiveRemoteNotification(userInfo: _userInfo)
      completionHandler(.newData) }
    }
}
```

**Method - setForegroundNotification**

Use the ‘setForegroundNotification’ method to get a notification message in the foreground and get the In-app notification messages and pass the notification data.

```swift
public func userNotificationCenter(_center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandlercompletionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    RefluttersdkPlugin.setForegroundNotification(notification) { options in
    completionHandler(options) 
    }
}
```

### Dynamiclink

Firebase dynamic link adopts a kind of deferred deep-linking approach for iOS. The firebase dynamic link match_type is unique to identify the correct installation.

For more details, refer to the link: https://firebase.google.com/docs/dynamic-links.

Send the dynamic- link details through the SDK registration.

**Dynamic-link data format**: [String: Any]

Consider the screenname as a ‘string’, and ‘customParams’ as a ‘JSON’ string.

```ruby
["screenName": "SIDOrder", "customParams": "{‘screenId’: 11, ‘productId’: 1029}", “isNewInstall”: true,“isDeepLink”: true]
```

**Case 1: When app installation is available**

The Firebase dynamic link opens the installed app and redirects to the respective screen set when a user clicks on the Firebase dynamic link.

**Case 2: When the app installation is not available.**

When a user clicks on the dynamic link, it redirects to the App store. After installing the app and launching the application, it redirects to the respective screen set.

Example: Using ‘AppDelegate’ method below iOS ver.13:

```swift
func application(_app: UIApplication, open url:URL,options:[UIApplicationOpenURLOptionsKey: Any]) -> Bool {
    RefluttersdkPlugin.open(url: url)
    return true 
   }
```
**Dynamic link receiver**

```swift
func application(_ application: UIApplication, willContinueUserActivityWithType userActivityType: String) ->Bool {
    if userActivityType == NSUserActivityTypeBrowsingWeb { 
        return true
    }
}
func application(_ application: UIApplication, continue userActivity:NSUserActivity, restorationHandler: @escaping ([Any]?) ->Void) ->Bool {
         //Dynamic Link
      RefluttersdkPlugin.handleDynamicLink(userActivity: userActivity) { returnData in
            
        } failureHandler: { error in
            
        }
      return false
     
}
```

Example: Using the **‘SceneDelegate’** method for iOS ver.13 and above

```swift
func scene(_scene: UIScene, openURLContextsURLContexts:Set<UIOpenURLContext>) { 
    if let url = URLContexts.first?.url {
       RefluttersdkPlugin.open(url: url)
    } 
}
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) { 
    //Dynamic Link
   RefluttersdkPlugin.handleDynamicLink(userActivity: userActivity) { returnData in
            
        } failureHandler: { error in
            
        }
}
```

## Flutter – Dart code setup

### Method–Deeplinking[Notifications&Smartlink]

```dart
import ‘package:refluttersdk/refluttersdk.dart’; 
final _reflutterSdk=Refluttersdk();
// Listener register
_reflutterSdk.listener((data) {
    //Do your screen navigation 
});
data sample {"activityName":"","fragmentName":"","MobileFriendlyUrl":"","customParams":"{}"}
```
### Method-sdkRegistration

In order to use the functions, import a plugin class within the Dart file considered for the function.

```dart
var userData = {
"userUniqueId": "", // Mandatory
"deviceToken":"<fcm token>",//Mandatory "name": "<user_name>", // String "email": "<email>", // String
"phone": "<phone_number>", // String "age": "<age>", // String
"gender": "<gender>", // String
"profileUrl": "<url>", // String
"dob": "<date of birth>", // String
"education": "<education>", // String
"employed": "<true||false>", // Bool
"married": "<true||false>", // Bool
"dynamicLink": {// Only for iOS
"applinks": "<applinks from firebase dynamiclink>", // String "storeId": "<store id for the application>" // String
};
_reflutterSdk.sdkRegisteration(userData);
```
Enable the following parameters as mandatory at the code level:

- The parameters **‘Unique id’** and the **‘Token’** are considered mandatory.
- The SDK registration must be called at every login when the login action is successful or when
implementing a similar function in the application.
> Note Implement the app as a dynamic link, and then pass two parameters listed below

- **‘applinks’** – The **‘applinks’** refers to the dynamic link host. Get the link while creating the dynamic link
in the Firebase console.

- **‘storeId’**: The **‘storeId’** refers to the appstore ‘app_id’. While uploading the app to the Appstore is done, it generates the identifier (Id).

### Update push token

**Method – “updatePushToken’** –Use the ‘UpdatePushToken’ method to update the device token.

```dart
_reflutterSdk.updatePushToken(“fcmToken”);
```

### CustomEvent

Use the ‘customEvent’ method to track the events using the below method.

```dart
var eventData =”payment”
_reflutterSdk.customEvent(eventData);
```

**Method – ‘customEventWithData’**

```dart
var eventData = { 
"name": "payment",
"data": {"id": "2d43", "price": "477"}
}
_reflutterSdk.customEvent(eventData);
```

### Location Update

Use the **‘locationUpdate’** method for updating the user location details.

```dart
double lat = 13.0827,
double lang = 80.2707, 
_reflutterSdk.locationUpdate(lat,lang);
```

### AppConversion(For conversion tracking)

Track the actions/activities of the users using the below method.

```dart
_reflutterSdk.appConversion();
```

**Method - For conversion trackingWithData**

```dart
Map appConversionData={ 
“name”:”xyz”,
 “data”:{
     “age”:”23”,
     “city”:”YYY”
  } 
};
_reflutterSdk.appConversionWithData(appConversionData);
```

### Screen Tracking

Use the ‘screenTracking’ method for screen tracking.

### Notification message delivery

**Notifications list**

**Method** – Use the **‘getNotificationList’** method to get Notification list data:

```dart
var notificationList = await _reflutterSdk.getNotificationList();
```

Example for notificationslist

```ruby
[ 
{
  "campaignId": "1582712270839", "notificationId": "1","title": "Notification title", "subtitle": "Notification Subtitle", "body": "Notification body", "notificationImageUrl": "Media_URL", "url": "Media_URL","screenUrl": "HomeViewController:app.HomeViewController",
"customParams": "{\"customParamKey1\":\"Value1\",\"customParamKey2\":100}", "timeStamp": "1593107578.784845","mobileFriendlyUrl":"1593107578.784845",
"customActions":"[{\"actionBgColor\":\"#FFFFFF\",\"actionType\":\"call\",\"actionTextColor\":\"#000000\",\"actionId\":6,\"url\":\"9123123123\",\"actionName\":\"Communicate\"},{\"actionBgColor\":\"#F FFFFF\",\"actionType\":\"maybelater\",\"duration\": \"1 Minute(s)\",\"actionTextColor\":\"#000000\",\"actionId\":7,\"url\":\"https:\\\/\\\/www.resu lticks.com\\\/\",\"actionName\":\"May be later\"}]",
"pushType": "2", "sourceType": "3", "bannerStyle": "2", "isRead": true
}
]
```

**sourceType** - The source type can be either ‘Web url’, video/audio, gif, or an image.

**pushType**-The different push types available for notification are alert/rich push, ‘Inapp’ push and ‘Inbox’ push notifications.

**bannerStyle** - The different banner styles available are full screen, middle of the screen, top of the screen and end (bottom) of it.

**Method ‘deleteNotificationByCampaignId’**

```dart
_reflutterSdk.deleteNotificationByCampaignId(campaingId);
```

**Notification count and read status**

**Method –‘getUnReadNotificationCount’**

Use the **‘getUnReadNotificationCount’** method for receiving the unread notification count from Resulticks SDK.

**Method –‘getReadNotificationCount’**

Use the **‘getReadNotificationCount’** method for receiving the notifications read count from Resulticks
SDK.

```ruby
var _readCount =await _reflutterSdk.getReadNotificationCount();
```

**Method –‘readNotification’**

Use the **‘readNotification’** method to set the notification read status to ‘Yes’.

```dart
_reflutterSdk.readNotification(“CampaignId”);
```

## Implementing service extensions

Add the new notification service extension.

- Navigate to the File>New followed by the Target.
- Click on Target and select the iOS platform.
- Select the **Notification service extension**.
- Confirm to activate the extension; Click on **Activate**. Activate creates a folder with **‘Swift’** file and **‘Info plist’**.
- Add the below code to the Swift file

```swift
import UserNotifications
import REIOSSDK

class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        if let bestAttemptContent = bestAttemptContent {
            // Modify the notification content here...
            bestAttemptContent.title = "\(bestAttemptContent.title) [modified]"
            
            contentHandler(bestAttemptContent)
        }
        REiosHandler.presentServiceExtension(request: request) { content in
                    contentHandler(content)
                }
    }
    
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }

}
```
- Verify the ‘info plist’ whether it has the below code. If not, then add it to the ‘info plist’ file.

```dart
<dict>
<key>NSExtensionPointIdentifier</key> 
  <string>com.apple.usernotifications.service</string> 
  <key>NSExtensionPrincipalClass</key> 
   <string>$(PRODUCT_MODULE_NAME).NotificationService</string>
</dict>
```

## Implementing content extensions

Add the new notification service extension.

- Navigate to File>New followed by Target.
- Click on Target and select the iOS platform.
- Select the **Notification content extension**.
- Confirm to activate the extension. Click on Activate.
Activate creates a folder with the ‘Swift’ file, ‘MainInterface’ and ‘Info plist’ files.
- Add the below code to the swift file:

```swift
import UIKit
import UserNotifications
import UserNotificationsUI
import REIOSSDK

class NotificationViewController: UIViewController, UNNotificationContentExtension, REiosContentExtensionReceiver {
    
    @IBOutlet var label: UILabel?
    
    override func viewDidLoad() {
        super.viewDidLoad()
       // REiosHandler.env = .run
        REiosHandler.contentExtDelegate = self
    }
    
    func didReceive(_ notification: UNNotification) {
        
        let dynamicLink = [ "applinks": "visionbanknative1.page.link", "storeId": "1289654399", "appBundleId": "com.resulticks.visionbank" ]
        REiosHandler.presnetContentExtension(vc: self, notification: notification, deeplinkData: dynamicLink) { height in
        }
    }
    
    func didCloseContentExtention() {
        if #available(iOSApplicationExtension 12.0, *) {
            self.extensionContext?.dismissNotificationContentExtension()
        } else {
            // Fallback on earlier versions
        }
    }
    
    func didReceive(_ response: UNNotificationResponse, completionHandler completion: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
        
    }
}
```
- Verify the **‘info plist’** and add **‘UNNotificationExtensionCategory’** to **‘Resul’**.

```swift
<dict> <key>NSExtension</key> <dict>
<key>NSExtensionAttributes</key> <dict>
<key>UNNotificationExtensionCategory</key>
<string>Resul</string>
<key>UNNotificationExtensionDefaultContentHidden</key> <true/> <key>UNNotificationExtensionInitialContentSizeRatio</key> <real>1</real> <key>UNNotificationExtensionUserInteractionEnabled</key> <true/>
</dict>
<key>NSExtensionMainStoryboard</key> <string>MainInterface</string> <key>NSExtensionPointIdentifier</key> <string>com.apple.usernotifications.content-extension</string>
    </dict>
</dict>
```

Open the ‘MainInterface’ and delete the ‘UILabel’ from the view controller.


## Deployment of iOS app

Follow the steps following steps to upload the application with bitcode.

**Bitcode setup**

- Enable Bitcode in iOS app projects and cocoapods projects.
- Navigate to the Build settings, search for Other C Flags.
- Click on the Add ‘+’ button to add the ‘fembed-bitcode’.
- In the Build settings, add the user-defined setting.
- Add the new key mentioned below and set the value to ‘bitcode’.

```swift
“BITCODE_GENERATION_MODE” = bitcode
```







