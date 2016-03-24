---
layout: post
title: "iOS开发之同一应用设置不同图标和名称"
date: 2014-09-06 22:01:47 +0800
comments: true
tags: iOS
---

经常在开发中遇到同一个App会有很多渠道版本，比如OTA内部测试版本，AppStore发布版本等。针对这些不同的版本我们通常会选择不同的图标、应用名称等，效果如下图所示：

![demo_preview.png](/images/ios-two-version-app/demo_preview.png)

P.S上面使用的两个测试图标分别来源于`土巴兔`和`乐视TV`的iPhone版本APP的图标，是两个非常不错的APP，感谢你们。

##Bundle ID

iOS系统区分不同的App是否相同是根据App的Bundle ID是否相同来判断的。如果想要在一个系统上安装一个App的多个版本其实是需要多个Bundle ID，就是说正式版一个Bundle ID，测试版一个Bundle ID。比如我们正式版(发布到AppStore上)的Bundle ID是`com.devzeng.myappappstore`，内部OTA测试版本的Bundle ID是`com.devzeng.myappota`.

##Build Configuration
默认Xcode会提供2个Build配置项(Build Configuration):Debug和Release。一般来说这样两种情况就足够了，但是在有些时候我们需要添加一个新的配置项，添加一个新的配置项的步骤如下：

1、方式一：选中`PROJECT`的名称，然后选中`Info`，点击`Configurations`下面的`+`选择`Duplicate "Debug" Configuration`,如下图：

![build_setting_01.png](http://blog.devzeng.com/images/ios-two-version-app/build_setting_01.png)

2、方式二：选中`PROJECT`的名称，然后选中`Editor`->`Add Configuration`->`Duplicate "Debug" Configuration`，如下图所示：

![build_setting_02.png](/images/ios-two-version-app/build_setting_02.png)


##User-Defined Setting

在Xcode中使用`User-Defined Setting`可以定义一些Xcode编译使用的宏配置，为了实现不同环境下App显示的名称和图标不同，可以在`User-Defined Setting`中定义一些有关应用程序名称和应用图标的配置。

1、开启`User-Defined Setting`，如下图：

![user-defined-01.png](/images/ios-two-version-app/user-defined-01.png)

2、添加`APP_DISPLAY_NAME`(APP的名称)、`APP_ICON_NAME`(APP图标名称)和`BUNDLE_IDENTIFIER`（APP Bundle ID）三个配置选项，效果如下图：

![user-defined-02.png](/images/ios-two-version-app/user-defined-02.png)

##Info.plist配置

关于常见的Info.plist的一些配置可以参考[《iOS中Info.plist文件的常见配置》](http://blog.devzeng.com/blog/ios-info-dot-plist-config.html)。

1、配置应用的图标

使用`${APP_ICON_NAME}.png`、`${APP_ICON_NAME}@2x.png`和`${APP_ICON_NAME}-120@2x.png`替代图标的名称。

![info_plist_icon_name.png](/images/ios-two-version-app/info_plist_icon_name.png)

2、配置应用的名称

设置`Bundle display name`为`${APP_DISPLAY_NAME}`，其中`APP_DISPLAY_NAME`是前面`User-Defined Setting`中设置的应用程序名称的配置项。

![info_plist_display_name.png](/images/ios-two-version-app/info_plist_display_name.png)

3、配置Bundle ID,用于区分不同的版本

设置`Bundle identifier`为`${BUNDLE_IDENTIFIER}`，其中`BUNDLE_IDENTIFIER`是前面`User-Defined Setting`中设置的应用程序Bundle ID的配置项。

![info_plist_bundle_id.png](/images/ios-two-version-app/info_plist_bundle_id.png)


##参考资料

1、[《How to Have Two Versions of the Same App on Your Device》](http://nilsou.com/blog/2013/07/29/how-to-have-two-versions-of-the-same-app-on-your-device/)

2、[《如何在一个设备上安装一个App的两个不同版本》](http://joeyio.com/ios/2013/08/16/how-to-have-two-versions-of-the-same-app-on-your-device/)

3、[《Adding a build configuration in Xcode》](http://stackoverflow.com/questions/19842746/adding-a-build-configuration-in-xcode)
