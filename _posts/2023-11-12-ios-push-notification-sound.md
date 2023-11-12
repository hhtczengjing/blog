---
layout: post
title: "iOS推送设置个性化消息提示音"
date: 2023-11-12 13:32:38 +0800
comments: true
tags: iOS
---

更新到 iOS 17 后，不少用户反馈默认的通知声音不一样了，觉得新的提示音太安静了，以至于错过某些关键的通知。很可惜的是我们无法指导用户通过设置更改 App 默认的通知铃声，只能更改系统自带APP相关的提示音（如：电话、短信、提醒、收到新邮件声等）。

部分用户表示微信可以修改推送的铃声，而且是修改完立即就生效了。设置界面效果：

![ios_apns_bells](/images/ios_apns_bells/wechat_bell_settings.png)

首先想到的是 APNs 推送可以指定 `sound` 字段，用于设置用户的铃声，大多数APP使用的跟随系统默认就是 `default`。示例数据：

```
{
    "aps" : {
        "alert" : "You got your emails.",
        "badge" : 9,
        "sound" : "bingbong.aiff"
    }
}
```

> 说明：应用程序主包或Library/Sounds应用程序容器目录的文件夹中的声音文件的名称（建议带后缀名）。`default` 为播放系统默认通知声音。

但是这里存在一个问题，在发送消息的时候怎么知道对方是使用什么铃声设置的，难道需要在服务端存这个设置数据，然后触发推送之前先查然后再对应进行赋值。从理论上是可行的，但是成本较高，不太实际。但是对应像微信那样不使用系统的默认铃声倒是可以采用这种方式。

之前写过一篇文章介绍使用 `Communication Notifications` 方式实现推送通知显示头像，利用的就是创建了一个 `Notification Service Extension` Target 来解决的。iOS 10 新增了 [Notification Service Extension](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension?language=objc)，官方给出的说明图如下：

![ios10_service_extension](/images/ios_apns_bells/ios10_service_extension.png)

这意味着在 APNs 到达我们的设备之前，还会经过一层允许用户自主设置的 Extension 服务进行处理，为 APNs 增加了多样性。个性化推送铃声设置的功能就是基于这个特性实现。

### 代码实现

整体的实现流程：

- (1) 在APNs的Payload里面开启mutable-content（当值为1的时候就可以触发Notification Service Extension相关的操作）
- (2) 创建一个Notification Service Extension的插件Target，可以直接复用之前创建的Target
- (3) 在APP的主Target做一个类似微信的"消息提示音"的设置页面，其中使用到的铃声需要提前导入到主Target里面（可能会增加包体积）
- (4) 收到通知后修改通知内容的sound字段，替换为上面设置的铃声名称

#### APNs推送数据改造支持

修改APNs的Payload参数（主要变动是设置mutable-content字段的内容为1）：

```
{
   "aps": {
      "mutable-content": 1,
      "alert": {
         "title": "消息标题",
         "body": "消息内容"
     }
   }
}
```

#### 核心代码实现

##### 1、多Target之间如何共享配置数据

NSUserDefaults 提供了一个特性用于实现在多个Target之间进行共享数据，方法定义如下：

```
- (nullable instancetype)initWithSuiteName:(nullable NSString *)suitename NS_AVAILABLE(10_9, 7_0) NS_DESIGNATED_INITIALIZER;
```

使用这个 API 需要在每个Target里面(需要使用的都得加)提前启用 App Groups 设置：

![app_groups_settings](/images/ios_apns_bells/app_groups_settings.png)

然后 `suitename` 传的内容就是 `App Groups ID`，即上面设置的内容，其他的就是正常 `NSUserDefaults` 的读写方法。

##### 2、设置铃声并写入配置

```
NSUserDefaults *sharedDefaults = [[NSUserDefaults alloc] initWithSuiteName:@"group.xxxxx"];
// 必须是包含后缀的完整名称
[sharedDefaults setObject:@"ring.mp3" forKey:@"sound"];
[sharedDefaults synchronize];
```

##### 3、读取铃声设置并修改通知内容

```
UNNotificationSound *sound = nil;
NSUserDefaults *sharedDefaults = [[NSUserDefaults alloc] initWithSuiteName:@"group.xxxxx"];
NSString *soundName = [sharedDefaults objectForKey:@"sound"];
if (soundName && [soundName isKindOfClass:[NSString class]]) {
	sound = [UNNotificationSound soundNamed:soundName];
}
if (sound) {
	self.bestAttemptContent.sound = sound;
}
```

> 说明：

- 关于铃声的格式仅支持`aiff/wav/caf/mp3`，且不能超过30秒
- 铃声文件添加到项目的main bundle里面，设置通知铃声为包含后缀名的的完整文件名，如：ring.mp3
- 这个特性仅支持 >= iOS 10.0

### 参考资料

- 1、[UNNotificationServiceExtension](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension?language=objc)

- 2、[Modifying content in newly delivered notifications](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications?language=objc)

- 3、[UNNotificationSound](https://developer.apple.com/documentation/usernotifications/unnotificationsound?language=objc)