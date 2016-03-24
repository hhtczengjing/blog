---
layout: post
title: "初识Apple Watch应用开发"
date: 2015-07-08 22:35:09 +0800
comments: true
tags: iOS
---

![hello_watch.jpg](/images/apple_watch_dev_glance/hello_watch.jpg)

自发布iOS8.2 SDK和Xcode6.2来，大家对于WatchKit的关注就不绝于耳。特别随着Apple Watch的发售一大批的Apple Watch的应用就如雨后春笋一般涌入AppStore。本文对[《WatchKit Programming Guide》](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide)中提到的相关概念结合目前的各方资料进行整理，重点介绍在Apple Watch开发中的一些基本概念和数据通信方面的内容。

###配置Xcode添加Watch应用

1、使用最新的Xcode(Xcode 6.2及以上)打开现有的iOS项目

2、选择File -> New -> Target, 选择Apple Watch

![new_target.png](/images/apple_watch_dev_glance/new_target.png)

3、选择WatchKit App

![watchkit_app.png](/images/apple_watch_dev_glance/watchkit_app.png)

4、如果您想要使用glance或者自定义通知界面，请选择相应的选项(推荐激活应用通知选项)。选中之后就会创建一个新的文件来调试该通知界面。如果您没有选择这个选项，那么之后只能手动创建这个文件了。

![watchapp_config.png](/images/apple_watch_dev_glance/watchapp_config.png)

完成上述操作之后，Xcode会自动创建两个Target：WatchKit应用扩展(WatchKit Extension)和Watch应用(Watch App)，并自动配置相应的设置。

![target_structure.jpg](/images/apple_watch_dev_glance/target_structure.jpg)

说明：

(1)Xcode将基于iOS应用的bundle ID来为两个新对象设置它们的bundle ID。比如说，iOS应用的bundle ID为`com.example.MyApp`，那么Watch应用的bundle ID将被设置为 `com.example.MyApp.watchapp`，WatchKit应用扩展的bundle ID被设置为`com.example.MyApp.watchkitextension`。这三个可执行对象的基本ID（即`com.example.MyApp`）必须相匹配，如果更改了iOS应用的bundle ID，那么就必须相应的更改另外两个对象的bundle ID。

(2)上面的结构图描述了iOS App、WatchKit Extension和Watch App三者之间的依赖关系。WatchKit依赖于iOS应用，而其同时又被Watch应用依赖。编译iOS应用将会将这三个对象同时编译并打包。其中Watch App运行在Watch上，只包含Storyboards和资源文件；WatchKit Extension运行在iPhone上，和对应的iPhone App在一起。用户点击Watch App后，与Watch匹配的iPhone会启动WatchKit Extension，然后和Watch建立连接，然后两者可以进行交互。

###iPhone App、WatchKit Extension和Watch App之间的关系

自iOS8起Apple推出一个新的概念-App Extension,可以让开发者开发第三方的键盘、通知中心（Today Widget）等。

![iphone_watch_communication.png](/images/apple_watch_dev_glance/iphone_watch_communication.png)

WatchKit Extension是iPhone App和Watch App通信的桥梁，下图展示了三者间的关系：

![watchkit_communication.jpg](/images/apple_watch_dev_glance/watchkit_communication.jpg)

1、WatchKit Extension(Extension)和Watch App(Host App)间的通信

在Extension中可以获取到Watch App界面元素的数据接口`WKInterfaceObject`（非控件View本身，更接近于Model），通过修改WKInterfaceObject的数据，来修改对应Watch App的界面。另外通过Xcode将Watch App控件的响应事件绑定到Extension中，可以实现Watch操作响应逻辑的处理。

2、WatchKit Extension(Extension)和iPhone App(Containing App)的通信

(1)共享存储空间

Extension和iPhone App之间的一种通信方式是读写同一块共享存储空间，达到数据交换的目的（见下图）。需要注意的是Extension和iPhone App属于不同的进程，要共享存储空间，需要在工程对应的target中同时打开App Groups的权限，并选择共享的组名（打开权限需要在Xcode中配置开发者账号和密码，同时本地需要有对应的开发证书，本文后面有介绍）。

![share_container.png](/images/apple_watch_dev_glance/share_container.png)

(2)直接进行通信

Extension和iPhone App另外一种通信方式是Extension主动向iPhone App发起请求，进行某种操作，或者请求数据（场景：Watch 收到新邮件通知后，点击已读按钮，在iPhone App上置已读）。

WatchKit Extension中向iPhone App发送请求：

```
+ (BOOL)openParentApplication:(NSDictionary * nonnull)userInfo reply:(void (^ nullable)(NSDictionary * nonnull replyInfo, NSError * nullable error))reply;
```

在iPhone App的AppDelegate中实现如下方法响应Watch App的请求：

```
- (void)application:(UIApplication * nonnull)application handleWatchKitExtensionRequest:(NSDictionary * nullable)userInfo reply:(void (^ nonnull)(NSDictionary * nullable replyInfo))reply;
```

在Github上找到一个叫做[MMWormhole](https://github.com/mutualmobile/MMWormhole)的开源库，它是专门用于Container App与Extension间传递消息，整个项目非常简洁实用。

(1)初始化MMWormhole，需要指定App Group配置的Identifer

```
self.wormhole = [[MMWormhole alloc] initWithApplicationGroupIdentifier:kGroupIdentifer optionalDirectory:nil];
```

(2)监听指定标识的消息，并作出相应的处理

```
[self.wormhole listenForMessageWithIdentifier:@"user" listener:^(id message) {
        self.label.text = [NSString stringWithFormat:@"%@", message];
    }];
```

(3)发送消息，需要指定标识符

```
[self.wormhole passMessageObject:@{@"name":@"zengjing"} identifier:@"user"];
```

###Watch App的启动过程和生命周期

当用户在Apple Watch上运行应用时，用户的iPhone会自行启动相应的WatchKit应用扩展。通过一系列的信号交换，Watch App和WatchKit Extension互相连接，因此消息能够在二者之间流通，直到用户停止与应用进行交互为止。此时，iOS将暂停应用扩展的运行。

在启动的过程中，WatchKit自动为当前界面创建相应的界面控制器。如果用户正在查看glance，WatchKit创建出来的界面控制器将与glance相连接。如果用户直接启动应用，WatchKit将从应用的主故事板文件中加载初始界面控制器。无论哪种情况，WatchKit应用扩展都提供一个名为WKInterfaceController的子类来管理相应的界面。

1、启动Watch App的过程

![launch_cycle_2x.png](/images/apple_watch_dev_glance/launch_cycle_2x.png)

当用户在Apple Watch上与应用进行交互时，WatchKit应用扩展将保持运行。如果用户明确退出应用或者停止与Apple Watch进行交互，那么iOS将停用当前界面控制器，并暂停应用扩展的运行。与Apple Watch的互动是非常短暂的，因此这几个步骤都有可能在数秒之间发生。所以，界面控制器应当尽可能简单，并且不要运行长时任务。重点应当放在读取和显示用户想要的信息上来。

2、界面控制器的生命周期

当应用启动时，WatchKit框架自行创建了相应的`WKInterfaceController`对象并调用`initWithContext:`方法。使用该方法来初始化界面控制器，然后加载所需的数据，最后设置所有界面对象的值。

![watch_app_lifecycle_simple_2x.png](/images/apple_watch_dev_glance/watch_app_lifecycle_simple_2x.png)

在应用生命周期的不同阶段，iOS将会调用WKInterfaceController对象的相关方法来让您做出相应的操作。WKInterfaceController的几个主要的方法说明如下：

- initWithContext：这个方法用来准备显示界面。借助它来加载数据，以及更新标签、图像和其他在Storyboard场景上的界面对象。

- willActivate：这个方法可以让您知道该界面是否对用户可视。借助它来更新界面对象，以及完成相应的任务，完成任务只能在界面可视时使用。

- didDeactivate：使用didDeactivate方法来执行所有的清理任务。例如，使用此方法来废止计时器、停止动画或者停止视频流内容的传输。您不能在这个方法中设置界面控制器对象的值，在本方法被调用之后到willActivate方法再次被调用之前，任何更改界面对象的企图都是被忽略的。 

说明：glances不支持动作方法。单击应用glance始终会启动应用。

###App Group配置

1、打开开发者中心后台,选择Certificates, Identifiers & Profiles

`https://developer.apple.com/membercenter/index.action`

![membercenter_home.png](/images/apple_watch_dev_glance/membercenter_home.png)

2、选择Identifiers下面的App Groups分组，然后点击右上角的加号创建一个新的分组

![app_groups_1.png](/images/apple_watch_dev_glance/app_groups_1.png)

3、填写App Groups的描述信息和Identifier字符串，其中ID必须以group开头，推荐的格式是`group.域名反转.应用的名称`, 如`group.com.devzeng.mydemo`

![app_groups_2.png](/images/apple_watch_dev_glance/app_groups_2.png)

4、在Xcode中进行配置,Targets -> Capabilities -> App Groups中开启选项，并设置为上面添加的ID(iPhone App和WatchKit Extension中都需要进行设置)

![app_groups_4.png](/images/apple_watch_dev_glance/app_groups_4.png)

###参考资料

1、[《Apple Watch开发初探》](http://nilsun.github.io/apple-watch/)

2、[《Apple Watch和iPhone通信实践》](http://nilsun.github.io/iPhone-watch-communication/)

3、[《WatchKit Programming Guide》](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide)

4、[《Apple Watch编程指南（中文版）》](http://www.cocoachina.com/ios/20141217/10660.html)

5、[《App与Extensions间通信共享数据》](http://yulingtianxia.com/blog/2015/04/06/Communication-between-your-App-and-Extensions/)