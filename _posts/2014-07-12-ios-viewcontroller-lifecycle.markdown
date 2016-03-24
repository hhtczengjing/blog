---
layout: post
title: "浅入浅出iOS应用程序的生命周期"
date: 2014-07-12 22:37:29 +0800
comments: true
tags: iOS
---

   由于iOS设备对于系统资源的使用有诸多限制，一个应用程序在前台与后台有不同的行为。为提高电池使用寿命和用户与前台应用程序的体验，操作系统限制应用程序在后台的运行。当应用程序在前台和后台之间进行切换时，操作系统会通知应用来进行相关的处理。
   

###1.应用程序的生命周期
  iOS的应用程序主要由未运行、未激活、激活、后台和挂起这五个状态组成，每个状态具体的描述如下：<br/>
  
  ![iOS应用程序状态切换图](/images/ios-viewcontroller-lifecycle/ios_viewcontroller_lifecycle_001.png)
  
####状态描述
  
```
(1)**未运行(Not Running)**：程序未启动

(2)**未激活(Inactive)**：程序在前台运行，不过没有接收到事件。在没有事件处理情况下程序通常停留在这个状态

(3)**激活(Active)**：程序在前台运行而且接收到了事件,这也是前台的一个正常的模式

(4)**后台(Backgroud)**：程序在后台而且能执行代码，大多数程序进入这个状态后会在这个状态上停留一会。时间到之后会进入挂起状态(Suspended)。有的程序经过特殊的请求后可以长期处于Backgroud状态

(5)**挂起(Suspended)**：程序在后台不能执行代码。系统会自动把程序变成这个状态而且不会发出通知。当挂起时，程序还是停留在内存中的，当系统内存低时，系统就把挂起的程序清除掉，为前台程序提供更多的内存。
```
####AppDelegate回调函数

(1)通知进程已启动但还未进入到状态保存

```
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```

(2)通知启动完成程序准备开始运行

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```

(3)当应用程序将要进入非活动状态，在此期间应用程序不接收消息或事件，如电话来了

```
- (void)applicationWillResignActive:(UIApplication *)application
```

(4)当应用程序进入到活动状态调用

```
- (void)applicationDidBecomeActive:(UIApplication *)application
```

(5)当程序进入到后台时被调用，

```
- (void)applicationDidEnterBackground:(UIApplication *)application
```

(6)当应用程序将要从后台进入到前台的时候调用

```
- (void)applicationWillEnterForeground:(UIApplication *)application
```

(7)当应用程序将要退出，通常用来保存数据和一些退出前的清理工作

```
- (void)applicationWillTerminate:(UIApplication *)application
```

(8)当程序载入后执行

```
- (void)applicationDidFinishLaunching:(UIApplication*)application
```

