---
layout: single
title: "Notifications Preview Placeholder"
date: 2018-03-30
categories: iOS
---

Since iOS 11, users can hide notification previews from all apps. There is both a global setting and individual setting per app to disable notification previews.

![Hide notification previews]({{ "/assets/2018-03-10-notifications-preview-placeholder/hide-notification-previews.png" }})

When disabled, the notification previews are replaced by a placeholder.

![Notifications previews hidden]({{ "/assets/2018-03-10-notifications-preview-placeholder/notifications-previews-hidden.png" }})

The default placeholder is "Notification" but can be customized using a [new `UserNotifications` API](https://developer.apple.com/documentation/usernotifications/unnotificationcategory/2873736-hiddenpreviewsbodyplaceholder)

The placeholder supports string dictionary and is specific to the notification category, so you can recieve notifications like "Thai vacation : 1 image and 2 comments" 

More details in the 2017 WWDC session [Best Practices and What’s New in User Notifications](https://developer.apple.com/videos/play/wwdc2017/708/) 
