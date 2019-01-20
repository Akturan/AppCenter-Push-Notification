# App Center Push Notification
Using App Center push notification.

## Xamarin Android and Xamarin.Forms Android
Firstly you will have to set up the platform specific push services. We need to create an new Project in fireabase console then we can get SenderID and Server Key in Counfiguration tab.
Add the App Center Push SDK in your Project. (Search for App Center and App Center Push and add both packages by clicking Add Packages)

### Modifiy AndroidManifest.xml
Insert following elements inside application tag
```xml
<receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
<receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
  <intent-filter>
    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
    <category android:name="${applicationId}" />
  </intent-filter>
</receiver>
```
### Add th following using statements in App.cs und MainActivity.cs
```c#
using Microsoft.AppCenter;
using Microsoft.AppCenter.Push;
```

### Initialize App Center in App.cs 
```c#
protected override void OnStart()
{
    // Handle when your app starts
    AppCenter.Start("app-center-push-app-secret-here", typeof(Push));
}
```
If you are using Xamarin.Forms you need to set the Sender ID in your MainActivity.cs file in the OnCreate method, befoew the loadApplication call.
```c#
protected override void OnCreate(Bundle savedInstanceState)
{
    TabLayoutResource = Resource.Layout.Tabbar;
    ToolbarResource = Resource.Layout.Toolbar;

    base.OnCreate(savedInstanceState);
    global::Xamarin.Forms.Forms.Init(this, savedInstanceState);
    
    Push.SetSenderId("your-sender-id");

    LoadApplication(new App());
}
```
When notification is received, PushNotificationReceived event is triggered and giving us notification details.
```c#
protected override void OnCreate(Bundle savedInstanceState)
{
    TabLayoutResource = Resource.Layout.Tabbar;
    ToolbarResource = Resource.Layout.Toolbar;

    base.OnCreate(savedInstanceState);
    global::Xamarin.Forms.Forms.Init(this, savedInstanceState);
    
    Push.SetSenderId("your-sender-id");
    Push.PushNotificationReceived += Push_PushNotificationReceived;
    LoadApplication(new App());
}

private void Push_PushNotificationReceived(object sender, PushNotificationReceivedEventArgs e)
{
    AlertDialog.Builder dlg = new AlertDialog.Builder(this);
    AlertDialog alert = dlg.Create();
    alert.SetTitle(e.Title);
    alert.SetButton("Ok", delegate
    {
        alert.Dismiss();
    });
    alert.SetMessage(e.Message);
    alert.Show();
}
```

## Using App Center open api for integrating App Center utilities into your Backend
App Center open Api: https://openapi.appcenter.ms

Firstly you need to get API-Token for open api authorization. You can create this in App Center under profile settings.
All methods for push are available under push category and we will use api endpoint 'notifications'

Auth Header Name is: X-API-Token

#### Endpoint
```
https://api.appcenter.ms/v0.1/apps/{owner_name}/{app_name}/push/notifications
```
You can find your owner_name and app_name in your project url.
https://appcenter.ms/orgs/MyOwnerName/apps/MyAppName

#### Send Notification all devices

```json
PoST https://api.appcenter.ms/v0.1/apps/MyOwnerName/MyAppName/push/notification
{
  "notification_content": {
    "name": "notification-name",
    "title": "notification-title",
    "body": "notification-body",
    "custom_data": {
      "additionalProp1": "Prop1"
    }
  }
}
```

#### Send Notification device-specific
```json
{
  "notification_target": {
    "type": "devices_target",
    "devices": ["deviceId"]
  },
  "notification_content": {
    "name": "notification-name",
    "title": "notification-title",
    "body": "notification-body",
    "custom_data": {
      "additionalProp1": "notification-addtional-prop"
    }
  }
}
```
You can find the device id using GetInstallIdAsync method. 
```c#
var deviceId = await AppCenter.GetInstallIdAsync()
```
