---
layout: post
title: "iOS推送支持显示用户头像"
date: 2023-09-25 12:30:00 +0800
comments: true
tags: iOS
---

从 iOS 15.0 开始新增了 `Communication Notifications` 的支持，可以让通知消息更加人性化。`Communication Notifications` 包含发送消息的联系人的头像，并且可以与 SiriKit 集成，以便 Siri 可以智能地根据常用联系人提供操作的快捷方式和建议。当前在 iOS 上很多Apple自带的应用或一些第三方应用（如钉钉、飞书、微博等）都使用了这个特性。

![demo_preview](/images/ios_communication_notifications/demo_preview.png)

### 代码实现

整体的实现流程：

- (1) 在APNs的Payload里面开启mutable-content（当值为1的时候就可以触发Notification Service Extension相关的操作），然后添加一个自定义的字段(icon)存放用户头像下载地址
- (2) 创建一个Notification Service Extension的插件Target，从Payload里面拿到头像的字段内容并下载下来
- (3) 利用INInteraction的机制模拟一个用户发信息，将下载的头像数据设置为消息发送者的头像
- (4) 修改通知内容(基于上面的的数据创建)并交给系统进行后续处理

#### APNs推送数据改造支持

修改APNs的Payload参数：

```
{
   "aps": {
      "mutable-content": 1,
      "alert": {
         "title": "消息标题",
         "body": "消息内容"
     },
   },
   "icon": "https://xxxxxx"
}
```

#### 主Target改造支持

1、修改 Info.plist 添加如下配置

![info_plist](/images/ios_communication_notifications/info_plist.png)

对应代码如下：

```
<key>NSUserActivityTypes</key>
<array>
  <string>INStartCallIntent</string>
  <string>INSendMessageIntent</string>
</array>
```

2、新增Capabilities支持"Communication Notifications"

![add_capabilities](/images/ios_communication_notifications/add_capabilities.png)

对应 entitlements 文件配置如下：

```
<key>com.apple.developer.usernotifications.communication</key>
<true/>
```

> 注意：需要对应修改appid的Capabilities，并更新Profile

#### 新建Target "Notification Service Extension"

1、Xcode 导航栏 "File -> New -> Target" 选择 "Notification Service Extension"

![create_target_01](/images/ios_communication_notifications/create_target_01.png)

2、按要求填写Target名称

![create_target_02](/images/ios_communication_notifications/create_target_02.png)

3、修改配置

(1) 设置 `Minimum Deployments` 为 15.0

![create_target_03](/images/ios_communication_notifications/create_target_03.png)

(2) 修改Info.plist

参考主Target的配置同样的添加一份：

```
<key>NSUserActivityTypes</key>
<array>
  <string>INStartCallIntent</string>
  <string>INSendMessageIntent</string>
</array>
```

4、代码实现

Extension Target 创建完成后会自动生成模板代码，只需要在示例模板中添加需要的业务代码即可：

(1) 添加头文件

```
#import <Intents/Intents.h>
#import <UserNotifications/UserNotifications.h>
```

(2) 解析头像地址并展示

核心代码如下：

```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    // 获取头像地址
    NSString *senderImageURLString = self.bestAttemptContent.userInfo[@"icon"];
    if (!senderImageURLString || ![senderImageURLString isKindOfClass:[NSString class]] || senderImageURLString.length <= 0) {
        self.contentHandler(self.bestAttemptContent);
        return;
    }
    
    // 标题
    NSString *title = self.bestAttemptContent.title;
    // 副标题
    NSString *subtitle = self.bestAttemptContent.subtitle;
    // 内容
    NSString *body = self.bestAttemptContent.body;
    // 下载并获取图片
    [self downloadINPersonWithURLString:senderImageURLString completionHandle:^(NSData *data) {
        // 处理下载失败的情况
        if (!data || data.length <= 0) {
            self.contentHandler(self.bestAttemptContent);
            return;
        }

        // 将图片数据转换成INImage
        INImage *avatar = [INImage imageWithImageData:data];
        if (!avatar) {
            self.contentHandler(self.bestAttemptContent);
            return;
        }

        // 创建发信对象(发送人)
        INPersonHandle *messageSenderPersonHandle = [[INPersonHandle alloc] initWithValue:@"" type:INPersonHandleTypeUnknown];
        NSPersonNameComponents *components = [[NSPersonNameComponents alloc] init];
        INPerson *messageSender = [[INPerson alloc] initWithPersonHandle:messageSenderPersonHandle
                                                          nameComponents:components
                                                             displayName:title
                                                                   image:avatar
                                                       contactIdentifier:nil
                                                        customIdentifier:nil
                                                                    isMe:NO
                                                           suggestionType:INPersonSuggestionTypeNone];
        // 创建自己对象(接收人)
        INPersonHandle *mePersonHandle = [[INPersonHandle alloc] initWithValue:@"" type:INPersonHandleTypeUnknown];
        INPerson *mePerson = [[INPerson alloc] initWithPersonHandle:mePersonHandle
                                                     nameComponents:nil
                                                        displayName:nil
                                                              image:nil
                                                  contactIdentifier:nil
                                                   customIdentifier:nil
                                                               isMe:YES
                                                     suggestionType:INPersonSuggestionTypeNone];
            
        // 创建intent
        INSpeakableString *speakableString = [[INSpeakableString alloc] initWithSpokenPhrase:subtitle ? subtitle : @""];
        INSendMessageIntent *intent = [[INSendMessageIntent alloc] initWithRecipients:@[mePerson, messageSender]
                                                                      outgoingMessageType:INOutgoingMessageTypeOutgoingMessageText
                                                                                  content:body
                                                                       speakableGroupName:speakableString
                                                                   conversationIdentifier:nil
                                                                              serviceName:nil
                                                                                   sender:messageSender
                                                                              attachments:nil];
        [intent setImage:avatar forParameterNamed:@"speakableGroupName"];
            
        // 创建 interaction
        INInteraction *interaction = [[INInteraction alloc] initWithIntent:intent response:nil];
        interaction.direction = INInteractionDirectionIncoming;
        [interaction donateInteractionWithCompletion:nil];

        // 创建 处理后的 UNNotificationContent
        NSError *error = nil;
        UNNotificationContent *messageContent = [request.content contentByUpdatingWithProvider:intent error:&error];
        if (!error && messageContent) {
            // 处理过的
            self.contentHandler(messageContent);
        } else {
            // 处理失败的情况
            self.contentHandler(self.bestAttemptContent);
        }
    }];
}

- (void)downloadINPersonWithURLString:(NSString *)urlStr completionHandle:(void(^)(NSData *data))completionHandler {
    __block NSData *data = nil;
    NSURL *imageURL = [NSURL URLWithString:urlStr];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    [[session downloadTaskWithURL:imageURL completionHandler:^(NSURL *temporaryFileLocation, NSURLResponse *response, NSError *error) {
        if (error != nil) {
            NSLog(@"%@", error.localizedDescription);
        } else {
            data = [[NSData alloc] initWithContentsOfURL:temporaryFileLocation];
        }
        completionHandler(data);
    }] resume];
}
```

### 存在的问题

#### 1、仅支持iOS15及以上的系统，低版本不生效

在开发调试过程中发现代码不生效，可以通过系统日志来看有相关的报错信息：

```
-[MIBundle pluginKitBundlesPerformingPlatformValidation:withError:]: Ignoring plugin at /var/installd/Library/Caches/com.apple.mobile.installd.staging/temp.ARJvVf/extracted/Payload/xxxx.app/PlugIns/yyyy.appex because it doesn't work on this OS version
```

#### 2、Extension Target里面存在下载请求，可能存在使用了HTTP地址的情况，需要在Info.plist里面添加如下内容

```
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

### 参考资料

- 1、[Implementing communication notifications](https://developer.apple.com/documentation/usernotifications/implementing_communication_notifications?language=objc)

- 2、[Handling Communication Notifications and Focus Status Updates](https://developer.apple.com/documentation/usernotifications/handling_communication_notifications_and_focus_status_updates)