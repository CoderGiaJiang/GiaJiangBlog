---
title: Notification in Yours App (二)
date: 2019-11-29 19:55:16
categories: Notification
---

### Managing Your App's Notification Support
应用必须在开启的时候配置本地和远程通知。特殊情况下，你必须提前配置你的应用，如果它执行以下操作：
- 显示 警告，播放音乐 或者 标记它的图标 去响应应到达的通知。
- 显示带有通知的自定义操作按钮。
通常你在应用完成启动之前执行你的所有配置。在 iOS 和 tv OS, 这意味着配置你的通知支持不晚于在你的 UIApplication 代理中 `application:didFinishLaunchingWithOptions:` 的方法。在 watchOS，配置支持不晚于 WKExtension 代理中 `applicationDidFinishLaunching` 的方法。你可以稍后再执行此配置，但是你必须避免在配置完成之前在你的应用中调用本地或者远程通知。
应用支持远程通知需要一些额外的配置，详细描述在 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1)

### Requesting Authorization to Interact with the User

在 iOS , tvOS 和 watchOS, 应用必须授权去显示警告，播放音乐或者标志应用图标即将到来通知的响应。请求授权将这些交互的控制置于用户的手中，他们可以允许或者拒绝你的请求。用户之后还可以在系统设置中修改应用的授权设置。
对于请求授权，调用 `NSUserNotificationCenter` 对象中`requestAuthorizationWithOptions:completionHandler:`方法 。如果你的应用