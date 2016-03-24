---
layout: post
title: "iOS8中的通知中心快速回复"
date: 2014-09-21 17:15:45 +0800
comments: true
tags: iOS
---

iOS8拥有了全新的通知中心，有全新的通知机制。当屏幕顶部收到推送时只需要往下拉，就能看到快速操作界面，并不需要进入该应用才能操作。在锁屏界面，对于推送项目也可以快速处理。基本上就是让用户尽量在不离开当前页面的前提下处理推送信息，再次提高处理效率。

能够进行直接互动的短信、邮件、日历、提醒，第三方应用，可以让你不用进入程序就能进行快捷操作，并专注于手中正在做的事情，用户可以做如下操作：

- 在通知横幅快速回复信息，不用进入短信程序；
- 可直接拒绝或接受邮件邀请；
- 可对提醒进行标记为完成或推迟；
- 当第三方应用更新接口后便可直接对应用进行快速操作.

最常见的短信的快速回复，以下面的一条广告短信为例说明下如何使用快速回复的功能，具体如下所示：

![ios8_notification_reply_1.png](/images/ios8_notification_reply/ios8_notification_reply_1.png)


![ios8_notification_reply_1.png](/images/ios8_notification_reply/ios8_notification_reply_2.png)


![ios8_notification_reply_1.png](/images/ios8_notification_reply/ios8_notification_reply_3.png)

最近在网上找了点资料，对这个功能进行了一下简单的实现，具体的步骤如下：

##创建步骤

在`AppDelegate`中的`didFinishLaunchingWithOptions`，按照下面的步骤加入各个代码片段：

1.创建消息上要添加的动作，以按钮的形式显示

```
//接受按钮
UIMutableUserNotificationAction *acceptAction = [[UIMutableUserNotificationAction alloc] init];
acceptAction.identifier = @"acceptAction";
acceptAction.title = @"接受";
acceptAction.activationMode = UIUserNotificationActivationModeForeground;
//拒绝按钮    
UIMutableUserNotificationAction *rejectAction = [[UIMutableUserNotificationAction alloc] init];
rejectAction.identifier = @"rejectAction";
rejectAction.title = @"拒绝";
rejectAction.activationMode = UIUserNotificationActivationModeBackground;
rejectAction.authenticationRequired = YES;//需要解锁才能处理，如果action.activationMode = UIUserNotificationActivationModeForeground;则这个属性被忽略；
rejectAction.destructive = YES;
```

2.创建动作（按钮）的类别集合

```
UIMutableUserNotificationCategory *categorys = [[UIMutableUserNotificationCategory alloc] init];
categorys.identifier = @"alert";
NSArray *actions = @[acceptAction, rejectAction];
[categorys setActions:actions forContext:UIUserNotificationActionContextMinimal];
```
    
3.创建UIUserNotificationSettings，并设置消息的显示类型

```
UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeAlert|UIUserNotificationTypeBadge|UIUserNotificationTypeSound) categories:[NSSet setWithObjects:categorys, nil]];
```

4.注册推送

```
[[UIApplication sharedApplication] registerForRemoteNotifications];
[[UIApplication sharedApplication] registerUserNotificationSettings:settings];
```
在使用Push的时候需要在数据包中加入特定的Category字段（字段内容需要前后端定义为一致）,终端接收到到后，就能展示上述代码对应Category设置的按钮，和响应按钮事件。

```
 {"aps":{"alert":"测试推送的快捷回复", "sound":"default", "badge": 1, "category":"alert"}}
```
说明：`Push数据包之前能带的数据最多为256字节，现在APPLE将该数值放大到2KB。`

##按钮处理事件

1.以本地通知为例介绍怎么处理，如下代码是发起本地通知事件：

```
UILocalNotification *notification = [[UILocalNotification alloc] init];
notification.fireDate = [NSDate dateWithTimeIntervalSinceNow:10];
notification.timeZone = [NSTimeZone defaultTimeZone];
notification.alertBody = @"测试推送的快捷回复";
notification.category = @"alert";
[[UIApplication sharedApplication] scheduleLocalNotification:notification];
```

2.实现下面的Delegate方法

（1）成功注册registerUserNotificationSettings后回调的方法

```
- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
{
    NSLog(@"%@", notificationSettings);
}
```

（2）事件处理

```
-(void)application:(UIApplication *)application handleActionWithIdentifier:(NSString *)identifier forLocalNotification:(UILocalNotification *)notification completionHandler:(void (^)())completionHandler
{
    //在非本App界面时收到本地消息，下拉消息会有快捷回复的按钮，点击按钮后调用的方法，根据identifier来判断点击的哪个按钮，notification为消息内容
    NSLog(@"%@----%@",identifier,notification);
    //处理完消息，最后一定要调用这个代码块
    completionHandler();
}
```

运行之后要按`shift + command + H`，让程序推到后台，或者按`command+L`让模拟器锁屏，才会看到效果！如果是程序退到后台了，收到消息后下拉消息，则会出现刚才添加的两个按钮；如果是锁屏了，则出现消息后，左划就会出现刚才添加的两个按钮。

##参考资料

1、[《iOS8推送消息的快速回复处理》](http://blog.csdn.net/yujianxiang666/article/details/35260135)