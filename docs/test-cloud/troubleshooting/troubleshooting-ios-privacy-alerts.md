---
title: iOS privacy alerts
description: Handling iOS privacy dialogs in App Center Test
keywords: test cloud, iOS, alerts, privacy, permission
author: oddj0b
ms.author: vigimm
ms.date: 02/11/2019
ms.topic: article
ms.assetid: bd84fb9f-8500-4fdf-beca-06d904ebb4db
ms.service: vs-appcenter
ms.custom: test
---

# iOS privacy alerts

> [!NOTE]
>This article does not address alerts created with `UIAlertViewController` in applications. They can be handled directly by appropriate test queries. Rather this article is about alerts generated by iOS that standard UI queries may not be able to handle.

While an iOS application is executing, the operating system may display alerts to the user at various times as the application attempts to activate or access Apple Push Notifications, Location Services, Contacts, the device Microphone or Camera, etc. to request permission. These are sometimes called alerts, System Alerts, System Popups, Springboard Alerts or Privacy Dialogs. When these requests are accepted, that acceptance state is persisted so the user will generally not see that alert again for that application on that device.

Tests running locally may never encounter these alerts if the requests have been previously accepted on the test device for that application. But, when run locally on a device that the application has not previously run on, these alerts will need to be addressed.

The same is true in App Center Test. When your tests execute in App Center Test they are running on pristine devices and alerts will be triggered when the application attempts to access or activate protected services or features.

Handling these alerts in App Center Test varies by the test framework.

## Xamarin.UITest and Calabash iOS

Xamarin.UITest and Calabash automatically accept alerts that they know about. Known alerts are based on text matching. If you have a case where a SpringBoard alert is not dismissed, search for the alert title in the [DeviceAgent.json files](https://github.com/calabash/DeviceAgent.iOS/tree/develop/Server/Resources.xcassets/springboard-alerts). 

```xml
$ cd DeviceAgent.iOS
$ git pull
$ find Server/Resources.xcassets -name "alerts.json" -exec grep -q "to access your location" {} \; -print
Server/Resources.xcassets/springboard-alerts/springboard-alerts-en_GB.dataset/alerts.json
Server/Resources.xcassets/springboard-alerts/springboard-alerts-en_AU.dataset/alerts.json
Server/Resources.xcassets/springboard-alerts/springboard-alerts-en.dataset/alerts.json
```

If your application has alerts that are not in that file, contact App Center Test Support (the blue chat icon in the lower right corner) to get them added. If they are in that file, there may be some problem with the device configuration which should be considered as a bug and reported to App Center Test Support for a quick fix.  

> [!NOTE]
>You may notice `DismissSpringboardAlerts()` in the Xamarin.UITest API. `DismissSpringboardAlerts()` is a method that Xamarin.UITest uses internally. There is generally no need to call `DismissSpringboardAlerts()` in user test code.

### XCUITest versus UIAutomation

Xamarin.UITest and Calabash use one of two Apple test frameworks for interacting with the devices. 

* If you are running tests locally, then the Apple Test Framework is *XCUITest*.
* If you are running tests in App Center Test, with iOS 10 or newer, then the Apple Test Framework is *XCUITest*.
* If you are running tests in App Center Test, with iOS versions older than iOS 10, then you are using Apple's *UIAutomation*.

Testing with UIAutomation requires a delay in the application before the first Alert occurs in order for Apple's UIAutomation framework to take control of the application under test. If this is an issue for your application and tests, see [Managing Privacy Alerts: Location Services, APNS, Contacts](https://github.com/calabash/calabash-ios/wiki/Managing-Privacy-Alerts:--Location-Services,-APNS,-Contacts).

## Calabash iOS

In addition to handling alerts automatically the same way that Xamarin.UITest does, Calabash also allows for managing alerts manually.

* To check if an alert is showing and query its attributes see `/springboard-alert` in [QueryRoutes.m](https://github.com/calabash/DeviceAgent.iOS/blob/develop/Server/Routes/QueryRoutes.m).
* To dismiss an alert by touching the button with a given title see `/set-dismiss-springboard-alert` in [GestureRoutes.m](https://github.com/calabash/DeviceAgent.iOS/blob/develop/Server/Routes/GestureRoutes.m).
* To dismiss known alerts and to toggle automatic dismissal see `/dismiss-springboard-alerts` and `/set-dismiss-springboard-alerts-automatically` in [MetaRoutes.m](https://github.com/calabash/DeviceAgent.iOS/blob/develop/Server/Routes/MetaRoutes.m). If `/dismiss-spring-board-alerts` is called and encounters an unknown alert, then an exception is raised.

This Calabash iOS code snippet shows how to use these methods:

```ruby
...
# Turn off automatic alert dismissal
device_agent.dismiss_springboard_alerts_manually!

# Do whatever triggers the alert, then wait for the alert
# which may or may not appear.
begin
  device_agent.wait_for_springboard_alert(10) # timeout is optional
  device_agent.dismiss_springboard_alert("OK")
rescue RuntimeError
   # Alert did not appear, so there is nothing to do
end
...

# Turn automatic dismiss back on
dismiss_springboard_alerts_automatically!
```

## Appium

Please see the Appium documentation for *Alert*.

* [Handle Alerts explicitly](https://github.com/appium/appium/issues/6864#issuecomment-258193484)
* [Appium XCUITest driver](https://github.com/appium/appium-xcuitest-driver)

## XCUITest

Please see the Apple documentation for *Monitoring UI Interruptions*.

* [XCTestCase](https://developer.apple.com/documentation/xctest/xctestcase)
* [addUIInterruptionMonitor](https://developer.apple.com/documentation/xctest/xctestcase/1496273-adduiinterruptionmonitor)
* [removeUIInterruptionMonitor](https://developer.apple.com/documentation/xctest/xctestcase/1496263-removeuiinterruptionmonitor)

## Getting help

You can always contact us through [the blue chat icon in the lower-right hand corner](https://intercom.help/appcenter/getting-started/getting-help-with-app-center). We do not provide 24/7 support, but will reply as soon as we can.

If you want help with a test run, navigate to the test run in question and copy the URL from your browser and paste it into the support conversation. A test run URL looks like something like https://appcenter.ms/orgs/OrgName/apps/App-Name/test/runs/77a1c67e-2cfb-4bbd-a75a-eb2b4fd0a747.
