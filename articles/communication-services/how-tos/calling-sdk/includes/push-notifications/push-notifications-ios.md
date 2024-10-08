---
author: probableprime
ms.service: azure-communication-services
ms.topic: include
ms.date: 09/08/2021
ms.author: rifox
---
[!INCLUDE [Install SDK](../install-sdk/install-sdk-ios.md)]

## Overview
Mobile push notifications are the pop-up notifications you see on mobile devices. For calling, we'll be focusing on VoIP (Voice over Internet Protocol) push notifications. We'll register for push notifications, handle push notifications, and then un-register push notifications. For a guide on CallKit integration in your iOS application, refer here, [CallKit integration with iOS ACS SDK](../../callkit-integration.md).

> [!NOTE]
> Registering for Push Notifications and handling of the incoming Push Notifications for a Teams user, the APIs are the same. The APIs described here can also be invoked on the `CommonCallAgent` or `TeamsCallAgent` classes.

## Set up push notifications

A mobile push notification is the pop-up notification that you get in the mobile device. For calling, we'll focus on VoIP (voice over Internet Protocol) push notifications. 

The following sections describe how to register for, handle, and unregister push notifications. Before you start those tasks, complete these prerequisites:

1. In Xcode, go to **Signing & Capabilities**. Add a capability by selecting **+ Capability**, and then select **Push Notifications**.
2. Add another capability by selecting **+ Capability**, and then select **Background Modes**.
3. Under **Background Modes**, select the **Voice over IP** and **Remote notifications** checkboxes.

:::image type="content" source="../../../../quickstarts/voice-video-calling/media/ios/xcode-push-notification.png" alt-text="Screenshot that shows how to add capabilities in Xcode." lightbox="../../../../quickstarts/voice-video-calling/media/ios/xcode-push-notification.png":::

### Register for push notifications

To register for push notifications, call `registerPushNotification()` on a `CallAgent` instance with a device registration token.

Registration for push notifications needs to happen after successful initialization. When the `callAgent` object is destroyed, `logout` will be called, which will automatically unregister push notifications.

```swift
let deviceToken: Data = pushRegistry?.pushToken(for: PKPushType.voIP)
callAgent.registerPushNotifications(deviceToken: deviceToken!) { (error) in
    if(error == nil) {
        print("Successfully registered to push notification.")
    } else {
        print("Failed to register push notification.")
    }
}
```

## Handle push notifications
To receive push notifications for incoming calls, call `handlePushNotification()` on a `CallAgent` instance with a dictionary payload.

```swift
let callNotification = PushNotificationInfo.fromDictionary(pushPayload.dictionaryPayload)

callAgent.handlePush(notification: callNotification) { (error) in
    if (error == nil) {
        print("Handling of push notification was successful")
    } else {
        print("Handling of push notification failed")
    }
}
```
## Unregister push notifications

Applications can unregister push notification at any time. Simply call the `unregisterPushNotification` method on `CallAgent`.

> [!NOTE]
> Applications are not automatically unregistered from push notifications on logout.

```swift
callAgent.unregisterPushNotification { (error) in
    if (error == nil) {
        print("Unregister of push notification was successful")
    } else {
       print("Unregister of push notification failed, please try again")
    }
}
```

## Disable internal push for incoming call

There are 2 ways that a push payload of an incoming call can be delivered to the callee.
  - Using APNS and registering the device token with the API mentioned above, `registerPushNotification` on `CallAgent` or `TeamsCallAgent`.
  - When a `CallAgent` or `TeamsCallAgent` is created, SDK also registers with an internal service to get the push payload delivered.

Using the property `disableInternalPushForIncomingCall` in `CallAgentOptions` or `TeamsCallAgentOptions` it's possible to instruct the SDK to disable the delivery of the push payload using the internal push service.

```swift
let options = CallAgentOptions()
options.disableInternalPushForIncomingCall = true
```
