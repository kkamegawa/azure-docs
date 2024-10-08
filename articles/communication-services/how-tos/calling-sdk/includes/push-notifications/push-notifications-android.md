---
author: sanathr
ms.service: azure-communication-services
ms.topic: include
ms.date: 08/06/2024
ms.author: sanathr
---
> [!IMPORTANT]
> On June 20, 2023, Google announced that it [deprecated sending messages using the FCM legacy APIs](https://firebase.google.com/docs/cloud-messaging). Google is removing the legacy FCM from service in June 2024. Google recommends [migrating from legacy FCM APIs to FCM HTTP v1](https://firebase.google.com/docs/cloud-messaging/migrate-v1).
> Please follow this [migration guide](/azure/communication-services/tutorials/call-chat-migrate-android-push-fcm-v1) if your Communication reosurce is still using the old FCM legacy APIs.

[!INCLUDE [Install SDK](../install-sdk/install-sdk-android.md)]

### Additional Prerequisites for Push Notifications

A Firebase account set up with Cloud Messaging (FCM) enabled and with your Firebase Cloud Messaging service connected to an Azure Notification Hub instance. See [Communication Services notifications](../../../../concepts/notifications.md) for more information.
Additionally, the tutorial assumes you're using Android Studio version 3.6 or higher to build your application.

A set of permissions is required for the Android application in order to be able to receive notifications messages from Firebase Cloud Messaging. In your `AndroidManifest.xml` file, add the following set of permissions right after the `<manifest ...>` or below the `</application>` tag.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
```

## Overview
Mobile push notifications are the pop-up notifications you see on mobile devices. For calling, we'll be focusing on VoIP (Voice over Internet Protocol) push notifications. We register for push notifications, handle push notifications, and then unregister push notifications.

> [!NOTE]
> Registering for Push Notifications and handling of the incoming Push Notifications for a Teams user, the APIs are the same. The APIs described here can also be invoked on the `CommonCallAgent` or `TeamsCallAgent` classes.

## Register for push notifications

To register for push notifications, the application needs to call `registerPushNotification()` on a `CallAgent` instance with a device registration token.

To obtain the device registration token, add the Firebase SDK to your application module's `build.gradle` file by adding the following lines in the `dependencies` section if it's not already there:

```groovy
// Add the SDK for Firebase Cloud Messaging
implementation 'com.google.firebase:firebase-core:16.0.8'
implementation 'com.google.firebase:firebase-messaging:20.2.4'
```

In your project level's *build.gradle* file, add the following line in the `dependencies` section if it's not already there:

```groovy
classpath 'com.google.gms:google-services:4.3.3'
```

Add the following plugin to the beginning of the file if it's not already there:

```groovy
apply plugin: 'com.google.gms.google-services'
```

Select *Sync Now* in the toolbar. Add the following code snippet to get the device registration token generated by the Firebase Cloud Messaging SDK for the client application instance. Be sure to add the below imports to the header of the main Activity for the instance to retrieve the token.

```java
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.InstanceIdResult;
```

Add this snippet to retrieve the token:

```java
FirebaseInstanceId.getInstance().getInstanceId()
    .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
        @Override
        public void onComplete(@NonNull Task<InstanceIdResult> task) {
            if (!task.isSuccessful()) {
                Log.w("PushNotification", "getInstanceId failed", task.getException());
                return;
            }

            // Get new Instance ID token
            String deviceToken = task.getResult().getToken();
            // Log
            Log.d("PushNotification", "Device Registration token retrieved successfully");
        }
    });
```
Register the device registration token with the Calling Services SDK for incoming call push notifications:

```java
String deviceRegistrationToken = "<Device Token from previous section>";
try {
    callAgent.registerPushNotification(deviceRegistrationToken).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while registering for Incoming Calls Push Notifications.")
}
```

## Push notification handling

To receive incoming call push notifications, call *handlePushNotification()* on a *CallAgent* instance with a payload.

To obtain the payload from Firebase Cloud Messaging, begin by creating a new Service (File > New > Service > Service) that extends the *FirebaseMessagingService* Firebase SDK class and override the `onMessageReceived` method. This method is the event handler called when Firebase Cloud Messaging delivers the push notification to the application.

```java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    private java.util.Map<String, String> pushNotificationMessageDataFromFCM;

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        // Check if message contains a notification payload.
        if (remoteMessage.getNotification() != null) {
            Log.d("PushNotification", "Message Notification Body: " + remoteMessage.getNotification().getBody());
        }
        else {
            pushNotificationMessageDataFromFCM = remoteMessage.getData();
        }
    }
}
```
Add the following service definition to the `AndroidManifest.xml` file, inside the `<application>` tag:

```xml
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

Once the payload is retrieved, it can be passed to the *Communication Services* SDK to be parsed out into an internal *IncomingCallInformation* object that will handle calling the *handlePushNotification* method on a *CallAgent* instance. A `CallAgent` instance is created by calling the `createCallAgent(...)` method on the `CallClient` class.

```java
try {
    IncomingCallInformation notification = IncomingCallInformation.fromMap(pushNotificationMessageDataFromFCM);
    Future handlePushNotificationFuture = callAgent.handlePushNotification(notification).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while handling the Incoming Calls Push Notifications.");
}
```

When the handling of the Push notification message is successful, and the all events handlers are registered properly, the application rings.

## Unregister push notifications

Applications can unregister push notification at any time. Call the `unregisterPushNotification()` method on callAgent to unregister.

```java
try {
    callAgent.unregisterPushNotification().get();
}
catch(Exception e) {
    System.out.println("Something went wrong while un-registering for all Incoming Calls Push Notifications.")
}
```

## Disable internal push for incoming call

There are two ways that a push payload of an incoming call can be delivered to the callee.
   - Using FCM and registering the device token with the API mentioned above, `registerPushNotification` on `CallAgent` or `TeamsCallAgent`.
   - When a `CallAgent` or `TeamsCallAgent` is created, SDK also registers with an internal service to get the push payload delivered.

Using the property `setDisableInternalPushForIncomingCall` in `CallAgentOptions` or `TeamsCallAgentOptions` it's possible to instruct the SDK to disable the delivery of the push payload using the internal push service.

```java
CallAgentOptions callAgentOptions = new CallAgentOptions();
callAgentOptions.setDisableInternalPushForIncomingCall(true);
```
