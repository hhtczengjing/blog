---
layout: post
title: "iOS开发中善用日志记录工具"
date: 2015-04-14 23:08:11 +0800
comments: true
tags: iOS
---

在iOS开发中经常需要靠记录日志来调试应用程序，最常见的做法是使用`NSLog`来输出相关的信息。大量的使用NSLog存在一定的弊端，将设备连接到电脑，打开XCode中的Device->Console，就可以从console查看到每条日志信息（或者是使用iTools的实时日志,推荐使用）。试想如果将很多核心的算法或者是信息都通过NSLog打印到控制台上，那么很有可能会被其他人获取到相关信息造成很多安全隐患，另外这样的应用也极有可能被Apple拒绝审核通过。

![console_log.png](/images/ios_logging_tools/console_log.png)

###使用宏来处理

常用的做法是在PCH文件添加如下的代码：

```
#ifdef DEBUG
#define DebugLog(format, ...) NSLog((@"%s [Line %d] " format), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__)
#else
#define DebugLog(...) do { } while (0)
#endif
```
然后将NSLog替换为DebugLog即可。

###使用CocoaLumberjack替代NSLog

![LumberjackLogo.png](/images/ios_logging_tools/LumberjackLogo.png)

CocoaLumberjack是Mac和iOS上一个集快捷、简单、强大和灵活于一身的日志框架。CocoaLumberjack类似于流行的日志框架（如log4j），但它是专为Objective-C设计的，利用了多线程、GCD（如果可用）、无锁原子操作Objective-C运行时的动态特性。

####CocoaLumberjack基本组件

CocoaLumberjack是由`DDASLLogger`、`DDTTYLogger`和`DDFileLogger`三个Log组件组成，各自的功能描述如下：

- DDASLLogger：支持将调试语句写入到苹果的日志中。一般正对Mac开发。。
- DDTTYLogger：支持将调试语句写入xCode控制台。在iOS开发中使用。
- DDFileLogger：支持将调试语句写入到文件系统。。

####1、快速集成

（1）下载CocoaLumberjack，引入CocoaLumberjack的头文件

`#import "DDLog.h"`

（2）指定日志的记录级别

`static const int ddLogLevel = LOG_LEVEL_VERBOSE;`

或者是

```
#ifdef DEBUG
static const int ddLogLevel = LOG_LEVEL_VERBOSE;
#else
static const int ddLogLevel = LOG_LEVEL_OFF;
#endif
```

日志的级别有如下几种：

- LOG_LEVEL_ERROR：如果设置为LOG_LEVEL_ERROR，仅仅能看到Error相关的日志输出。
- LOG_LEVEL_WARN：如果设置为LOG_LEVEL_WARN，能看到Error、Warn相关的日志输出。
- LOG_LEVEL_INFO：如果设置为LOG_LEVEL_INFO，能够看到Error、Warn、Info相关的日志输出。
- LOG_LEVEL_DEBUG：如果设置为LOG_LEVEL_DEBUG，能够看到Error/Warn/Info/Debug相关的日志输出。
- LOG_LEVEL_VERBOSE：如果设置为LOG_FLAG_VERBOSE，能够看到所有级别的日志输出。
- LOG_LEVEL_OFF:不输出日志。

（3）使用DDLogError/DDLogWarn/DDLogDebug/DDLogVerbose来替代NSLog

```
DDLogError(@"[Error]:%@", @"输出错误信息");//输出错误信息
DDLogWarn(@"[Warn]:%@", @"输出警告信息");//输出警告信息
DDLogInfo(@"[Info]:%@", @"输出描述信息");//输出描述信息
DDLogDebug(@"[Debug]:%@", @"输出调试信息");//输出调试信息
DDLogVerbose(@"[Verbose]:%@", @"输出详细信息");//输出详细信息
```

####2、结合XcodeColor让日志带上颜色

有了解过Android开发的朋友都会知道，在Android开发中LogCat的日志查看功能是十分强大的，特别是不同级别的日志输出显示的颜色是不同的，例如错误信息的颜色是红色的，其实在Xcode中结合XcodeColor插件也是可以实现该效果的，具体的配置步骤如下：

（1）下载安装插件

到`https://github.com/DeepIT/XcodeColors`下载XcodeColors插件，直接使用Xcode打开`XcodeColors.xcodeproj`文件，然后`Command+B`编译项目可以自动将插件安装至`~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/XcodeColors.xcplugin`路径下。

也可以使用Alcatraz来安装插件，具体请参考[《使用Alcatraz来管理Xcode插件》](http://blog.devtang.com/blog/2014/03/05/use-alcatraz-to-manage-xcode-plugins/)。

（2）重启Xcode，运行测试用例

彻底退出Xcode，重新启动Xcode。再次打开`XcodeColors.xcodeproj`运行`TestXcodeColors`的target，测试插件是否安装成功。

（2）CocoaLumberjack开启颜色分级

`[DDTTYLogger sharedInstance].colorsEnabled = YES;`

各个的级别的颜色如下：

```
DDLogError(@"Error");//红色
DDLogWarn(@"Warn");//黄色
DDLogInfo(@"Info");//默认是黑色
DDLogDebug(@"Debug");//默认是黑色
DDLogVerbose(@"Verbose");//默认是黑色
```
如果要设置不同分级的颜色值，可以使用如下代码：

```
[[DDTTYLogger sharedInstance] setForegroundColor:[UIColor orangeColor] backgroundColor:nil forFlag:LOG_FLAG_INFO];//设置INFO级别的日志的颜色为橙色
```

效果如下图所示：

![log_color_level.png](/images/ios_logging_tools/log_color_level.png)

说明：

可能在Xcode中无法正常显示颜色，需要配置Xcode的环境变量，设置“Edit Scheme”-> "Run" -> "Arguments"(Environment Variabl)环境变量，添加一个叫做`XcodeColors`并且设置值为`YES`，如下图所示。

![xcode_color_config.png](/images/ios_logging_tools/xcode_color_config.png)

###参考资料

1、[《XcodeColors.md》](https://github.com/CocoaLumberjack/CocoaLumberjack/blob/master/Documentation/XcodeColors.md)

2、[《CocoaLumberjack——带颜色的Log》](http://www.cnblogs.com/liufan9/p/3552832.html)

3、[《iOS第三方库-CocoaLumberjack-DDLog 》](http://blog.sina.com.cn/s/blog_7b9d64af0101kkiy.html)