---
title: Notification in Yours App (二)
---

### Managing Your App's Notification Support
#### 管理应用通知支持

应用必须在开启的时候去配置本地和远程通知。特殊情况下，如果它执行以下操作, 必须提前配置应用程序：

* 显示 提醒、播放音乐 或者 标记应用icon 去响应到达的通知。
* 显示 带有通知的自定义操作按钮。

通常在应用完成启动之前执行你的所有配置。在 iOS 和 tv OS, 这意味着你的通知配置支持不晚于 UIApplication 代理中 `application:didFinishLaunchingWithOptions:` 的方法。在 watchOS，配置支持不晚于 WKExtension 代理中 `applicationDidFinishLaunching` 的方法。你可以稍后再执行此配置，但是你必须避免在配置完成之前在你的应用中调用本地或者远程通知。
应用支持远程通知需要一些额外的配置，详细描述在 [Configuring Remote Notification Support](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1)

### Requesting Authorization to Interact with the User

#### 向用户请求授权

在 iOS , tvOS 和 watchOS, 应用必须授权去显示提醒，播放音乐或者标志应用图标即将到来通知的响应。请求授权将这些交互的控制置于用户的手中，他们可以允许或者拒绝你的请求。用户还可以在之后的系统设置中修改应用的授权设置。
对于请求授权，调用 `NSUserNotificationCenter` 单例对象中 [requestAuthorizationWithOptions:completionHandler:`](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649527-requestauthorizationwithoptions) 方法。如果你的应用已获得授权所有要求的互动类型的授权，系统回调带有 granted 参数为 YES 的完成处理 block。如果一个或者多个的互动类型没有被允许，参数返回 NO。**列表 2-1** 展示了如何为播放声音和显示提醒请求授权。使用完成处理的 block，根据交互类型是否允许或拒绝去更新你的应用的行为。

**列表 2-1 为用户交互请求授权**

**OBJECTIVE-C**

```
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound)
completionHandler:^(BOOL granted, NSError * _Nullable error) {
// Enable or disable features based on authorization.
}];
```

**SWIFT**

{% highlight Objective-C %} 
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.alert, .sound]) { (granted, error) in
// Enable or disable features based on authorization.
}
{% endhighlight %}

你的应用开启并调用 [
requestAuthorizationWithOptions:completionHandler: ](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649527-requestauthorizationwithoptions) 方法的第一时间，系统提示用户允许或者拒绝交互请求。因为系统保存用户的响应，当用户再次开启并调用该方法时不再提醒。

> NOTE
> 用户能够在任何时候在系统设置中改变交互授权类型。你可以调用 `UNUserNotificationCenter` 中 [getNotificationSettingsWithCompletionHandler: ](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649524-getnotificationsettingswithcompl) 的方法去精确地确定哪种交互类型可以使用。


### Configuring Categories and Actionable Notifications

#### 配置通知中的 Categories 和 Action 

可操作的通知提供给用户一个快速和方便的方式去完成一个通知响应的相关任务。而不是强制用户去开启应用，可操作通知界面上提供用户能点击的自定义操作按钮。当用户点击时，每一个按钮从通知界面上消失并转发选择的操作到你的应用以便立即处理。转发操作到你的应用，避免了进一步引导用户在应用中执行操作的需要，从而节省时间。

应用必须明确地添加支持可操作的通知。在开启时间，应用必须注册一个或者多个 category, category 定义着应用发送的通知类型。每一个 category 关联的 Action 能够在用户收到一种通知类型执行。每一个 category 能够和四个action 关联，尽管 action 的数量显示依赖于通知如何显示和显示的位置。例如，在横幅不能显示超过两个 action。

>NOTE
>可操作通知只支持 iOS 和 watchOS.

### Registering the Notification Categories for Your App

#### 注册通知 Category 到应用中

Categories 定义你的应用支持的通知类型与告知系统你想如何呈现一个通知。使用 Categories 去结合自定义操作和通知，并指定定如何处理改类型通知的选项。例如，使用Categories 选项去指定是否能够在 CarPlay 环境下展示通知。

在开启时，你注册所有的应用 Categories 马上使用单例 [UNUserNotificationCenter](https://developer.apple.com/reference/usernotifications/unusernotificationcenter) 对象的 [setNotificationCategories: method](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649512-setnotificationcategories) 方法。调用方法之前，你创建一个或者多个 [UNNotificationCategory](https://developer.apple.com/reference/usernotifications/unnotificationcategory) 的类的实例对象并指定显示该类型的通知时候要使用的 Categories 的名字和选项。Category 名字存在应用内部，并且永远不会被用户看见。当安排通知是，你在通知内容中包含的 category 名字，该通知内容系统用于恢复选项和显示通知。

**列表 2-2** 显示如何在系统中创建一个简单的 [UNNotificationCategory](https://developer.apple.com/reference/usernotifications/unnotificationcategory) 对象和注册。这个 category 有一个 "GRNERAL" 的名字并被配置带有一个自定义关闭操作的选项，这个选项可以在用户没有做任何操作时的情况下关闭通知界面时让系统通知应用。

**列表 2-1 创建和注册一个通知 category**

**OBJECTIVE-C**
{% highlight Objective-C %} 
UNNotificationCategory* generalCategory = [UNNotificationCategory
categoryWithIdentifier:@"GENERAL"
actions:@[]
intentIdentifiers:@[]
options:UNNotificationCategoryOptionCustomDismissAction];

// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, nil]];
{% endhighlight %}

**SWIFT**

{% highlight Objective-C %} 
let generalCategory = UNNotificationCategory(identifier: "GENERAL",
actions: [],
intentIdentifiers: [],
options: .customDismissAction)

// Register the category.
let center = UNUserNotificationCenter.current()
center.setNotificationCategories([generalCategory])
{% endhighlight %}

你没有必要为所有的你在应用中调用通知安排一个 category。尽管如此，如果你没有包含一个 category, 你的通知在显示的时候没有任何自定义操作或者配置选项。

### Adding Custom Actions to Your Categories

#### 添加自定义 Action 到 Category

每一个注册的 category 可以最多可以包含四个自定义操作。当有一个 category 包含自定义操作时，系统在通知界面上添加一个按钮，每一个按钮都带有自定义操作的标题。如果用户点击其中一个自定义操作，系统发送相同的操作标识到应用，根据需要打开应用。

去定义一个自定义操作，创建一个 [UNNotificationAction](https://developer.apple.com/reference/usernotifications/unnotificationaction) 对象并添加它到你一个 category 的对象中。每一个操作包含相应按钮的标题字符串和如何显示按钮和处理相关任务的选项。当用户选择一个操作时，系统为你应用提供操作标识字符串，你可以使用该字符串去标识要执行的任务。**列表 2-3** 是对**列表 2-2** 的扩充：添加一个带有两个自定义操作的新的 category 

**列表 2-3 定义带有自定义操作的一个 category**

**OBJECTIVE-C**

{% highlight Objective-C %} 

UNNotificationCategory* generalCategory = [UNNotificationCategory
categoryWithIdentifier:@"GENERAL"
actions:@[]
intentIdentifiers:@[]
options:UNNotificationCategoryOptionCustomDismissAction];

// Create the custom actions for expired timer notifications.
UNNotificationAction* snoozeAction = [UNNotificationAction
actionWithIdentifier:@"SNOOZE_ACTION"
title:@"Snooze"
options:UNNotificationActionOptionNone];

UNNotificationAction* stopAction = [UNNotificationAction
actionWithIdentifier:@"STOP_ACTION"
title:@"Stop"
options:UNNotificationActionOptionForeground];

// Create the category with the custom actions.
UNNotificationCategory* expiredCategory = [UNNotificationCategory
categoryWithIdentifier:@"TIMER_EXPIRED"
actions:@[snoozeAction, stopAction]
intentIdentifiers:@[]
options:UNNotificationCategoryOptionNone];

// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, expiredCategory,
nil]];

{% endhighlight %}

**SWIFT**

{% highlight Objective-C  %}

let generalCategory = UNNotificationCategory(identifier: "GENERAL",
actions: [],
intentIdentifiers: [],
options: .customDismissAction)

// Create the custom actions for the TIMER_EXPIRED category.
let snoozeAction = UNNotificationAction(identifier: "SNOOZE_ACTION",
title: "Snooze",
options: UNNotificationActionOptions(rawValue: 0))
let stopAction = UNNotificationAction(identifier: "STOP_ACTION",
title: "Stop",
options: .foreground)

let expiredCategory = UNNotificationCategory(identifier: "TIMER_EXPIRED",
actions: [snoozeAction, stopAction],
intentIdentifiers: [],
options: UNNotificationCategoryOptions(rawValue: 0))

// Register the notification categories.
let center = UNUserNotificationCenter.current()
center.setNotificationCategories([generalCategory, expiredCategory])

{% endhighlight %}

尽管你可以最多指定在每一个 category 带有四个自定义操作，系统在一些情况下也只能显示前两个。例如，系统在横幅中显示通知只能显示两个操作。当初始化 [UNNotificaitonCategory](https://developer.apple.com/reference/usernotifications/unnotificationcategory) 对象，所以在配置操作数组时，总是配置相关的操作在数组前面。

如果你配置一个使用 UNTextInputNotificationAction 类的操作, 系统提供一种用户输入文字作为通知响应一部分的方式。文字输入操作能够用于收集用户的自由形式的文本。例如：一个消息应用能够允许用户为提供一个自定义的回应信息。当一个文本输入操作被发到应用中去处理时，系统将用户的响应打包到 [UNTextInputNotificationResponse](https://developer.apple.com/reference/usernotifications/untextinputnotificationresponse) 对象。

关于处理选中的自定义操作的详细信息，见 [Responding to the Selection of a Custom Action](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW2)。


### Preparing Custom Alert Sounds

#### 准备自定义提示铃声

本地和远程通知能够指定自定义通知铃声，可以打包音频数据在 aiff, wav 或者 caf 文件。因为他们用于播放系统声音，自定义声音能够为以下音频格式中的一种：

* Linear PCM
* MA4(IMA/ADPCM)
* µLaw
* aLaw

放置自定义铃声文件到应用包里面或者在应用容器目录中的 Library/Sounds 文件。自定义铃声必须低于30秒。如果超过这个限制，将会被默认的系统铃声替换掉。

你可以使用 afconvert 工具去转换声音。例如：在 CAF 文件中将16位的 PCM 系统声音 Submarine.aff 转换成 IMA4 音频, 在终端命令行中执行下面代码：

`afconvert /System/Library/Sounds/Submarine.aiff ~/Desktop/sub.caf -d ima4 -f caff -v`

如何关联一个音频文件到通知的详细信息，见 [Adding a Sound to the Notification Content](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW3)。

### Managing Your App‘s Notifications Settings

#### 管理你的应用通知设置

因为用户能够在任何时候改变应用的通知设置，你也可以在任何时候使用 UNUserNotificationCenter 单例对象中 [getNotificationSettingsWithCompletionHandler:](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649524-getnotificationsettingswithcompl) 方法去获取应用授权状态。这个方法返回一个 UNNotificationSettings 对象, 该对象的内容反映了应用当前授权状态和当前的通知环境。

使用 [UNNotificationSettings](https://developer.apple.com/reference/usernotifications/unnotificationsettings) 对象去调整你的应用通知的相关代码。你可以传达你应用的提示，标志，和声音授权设置到自定义的服务端，因此你的服务端能够适应任何远程通知内容的选项。你可以使用其他设置调整如何安排和配置通知。关于更多有用的设置，见 [UNNotificationSettings Class Reference](https://developer.apple.com/reference/usernotifications/unnotificationsettings)。

### Managing Delivered Notifications

#### 管理分发的通知

当本地通知和远程通知没有被应用或者用户直接处理，他们存放在通知中心，因此他们可以稍后被展示。使用 UNUserNotificationCenter 单例对象中的 [getDeliveredNotificationsWithCompletionHandler:](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649520-getdeliverednotifications) 方法去获取一系列仍在通知中心显示的通知。如果你找到过期或者不应该显示给用户的通知，可以使用 [removeDeliveredNotificationsWithIdentifiers: ](https://developer.apple.com/reference/usernotifications/unusernotificationcenter/1649500-removedeliverednotifications) 方法移除他们。

