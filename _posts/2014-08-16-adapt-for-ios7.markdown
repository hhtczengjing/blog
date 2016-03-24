---
layout: post
title: "iOS应用程序适配iOS7"
date: 2014-08-16 23:02:33 +0800
comments: true
tags: iOS
---

Apple自去年发布iOS7以来一直以来都有关注目前各个大厂发布的app，到目前为止基本上绝大多数App Store上的app已经做到iOS7适配，不光是支持iOS7的布局调整更多的是在iOS7整体设计方面的改进，朝着扁平化和简单线条话的方式进行设计。

##UI适配

  在iOS7中view默认是全屏模式，状态栏的高度也加在了view的高度上，例如iOS7之前iPhone5/5s/5c中self.view.frame.size.height = 548，在iOS7中就是568了，在iOS7中navigationbar是半透明的，statusbar则是全透明的，这样一来，原来的程序用xcode5+iOS7sdk上编译后运行就会出现问题了。
  
  关于iOS6/iOS7 SDK在不同的设备上显示的差异对比图如下：
  
  ![ios_design.png](/images/ios7_adapt/ios_design.png)

1、没有导航栏的适配

![ios7_no_navi.png](/images/ios7_adapt/ios7_no_navi.png)

在没有导航栏的情况下，代码运行在iOS7上，内容向上偏移了20px。适配的办法是将view整体向下移20。
```
self.view.bounds = CGRectMake(0, -20, self.view.frame.size.width, self.view.frame.size.height);
```

2、有导航栏的适配

![ios7_navi.png](/images/ios7_adapt/ios7_navi.png)

在iOS7中，苹果引入了一个新的属性，叫做[UIViewController setEdgesForExtendedLayout:]，它的默认值为UIRectEdgeAll。当你的容器是navigation controller时，默认的布局将从navigation bar的顶部开始。这就是为什么所有的UI元素都往上漂移了44pt，修复这个问题很简单，在`viewDidLoad`方法中加入如下代码：

```
#ifdef IOS7_SDK_AVAILABLE
    if(IOS7_OR_LATER)
    {
        self.edgesForExtendedLayout = UIRectEdgeNone;
        self.extendedLayoutIncludesOpaqueBars = NO;
        self.modalPresentationCapturesStatusBarAppearance = NO;
    }
#endif
```

上面使用的宏的代码如下，其中`IOS7_SDK_AVAILABLE`是用来判断是否使用的是iOS7的SDK，`IOS7_OR_LATER`指的是当前系统的版本是否是iOS7及以上的系统。

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 70000
#define IOS7_SDK_AVAILABLE 1
#endif
#define IOS7_OR_LATER ([[[UIDevice currentDevice] systemVersion] compare:@"7.0" options:NSNumericSearch] != NSOrderedAscending)
```

补充说明:

(1)edgesForExtendedLayout是一个类型为UIExtendedEdge的属性，指定边缘要延伸的方向。 因为iOS7鼓励全屏布局，它的默认值很自然地是UIRectEdgeAll，四周边缘均延伸，就是说，如果即使视图中上有navigationBar，下有tabBar，那么视图仍会延伸覆盖到四周的区域。

如果把视图做如下设置，那么视图就不会延伸到这些bar的后面了.

3、其他UI方面的适配

(1)可以用旧版的sdk来编译，这样在真机上还是和原来一样的效果

(2)如果勾选了Hide during application lauch的话，在iOS7的设备上是没有问题的，启动完以后status bar会重新出现的，但是在iOS7以下的设备需要在launch didfinish里面把status bar显示出来

(3)对于UILabel，在iOS 7中它的background颜色默认是clearColor，而在iOS 6中默认的是白色。所以，我们最好在代码中对label的background颜色进行明确的设置`label.backgroundColor = [UIColor clearColor];`

##其他适配

1、mac地址无法获取

  在iOS6之后，苹果禁用了禁用了UIDevice的uniqueIdentifier方法，所以获取设备唯一标识的方法采用了获取Mac地址然后MD5加密，但是在iOS7中发现，该方法统一返回02:00:00:00:00:00,所以用做设备的标识符已经没有意义。
  
  推荐使用CFUUID(它是CoreFoundatio包的一部分，因此API属于C语言风格。CFUUIDCreate 方法用来创建CFUUIDRef，并且可以获得一个相应的NSString).
  
  ```
  CFUUIDRef cfuuid = CFUUIDCreate(kCFAllocatorDefault);
NSString *cfuuidString = (NSString*)CFBridgingRelease(CFUUIDCreateString(kCFAllocatorDefault, cfuuid));
  ```
  获得的这个CFUUID值系统并没有存储。每次调用CFUUIDCreate，系统都会返回一个新的唯一标示符。如果你希望存储这个标示符，那么需要自己将其存储到NSUserDefaults, Keychain, Pasteboard或其它地方。

##参考资料

1、[《iOS 7 教程：让程序同时支持iOS 6和iOS 7》](http://beyondvincent.com/blog/2013/11/19/122-working-with-ios-6-and-7/)

2、[《iOS 7 教程：定制iOS 7中的导航栏和状态栏》](http://beyondvincent.com/blog/2013/11/03/120-customize-navigation-status-bar-ios-7/)

3、[《【IOS】IOS7 UI适配》](http://blog.csdn.net/toss156/article/details/11843873)