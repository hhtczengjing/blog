---
layout: post
title: "iOS开发之多系统版本兼容"
date: 2014-08-30 21:21:51 +0800
comments: true
tags: iOS
---

Apple自2007年发布第一版iOS操作系统以来，差不多以后的每年都会发布新版的操作系统，从最开始的iPhone OS到现在即将正式发布的iOS8。目前市面上能见到的iOS的版本主要是iOS4.3、iOS5、iOS6、iOS7，在实际的开发中遇到很多用户由于各种各样的原因没有升级到最新版，这就给我们开发者带了不少的麻烦。

不过值得高兴的是截止到2014年8月24日，目前市面上大部分的iOS设备已经更新到iOS7了(91% of devices are using iOS 7)，对于一些占小份额比例的版本其实可以忽略了。各版本iOS的最新市场占有率分布如下图（数据来源于Apple官方公布）所示。

![chart-8-10-14.png](/images/ios_version/chart-8-10-14.png)

比如当前有很多iPhone应用都是只支持iOS6.0以上的设备

![app_store_app_info.png](/images/ios_version/app_store_app_info.png)

##Xcode配置

作为iOS开发者，我们都希望软件的受众越多越好。怎么样让软件尽量适应最多的系统版本？这里我们就应该了解iPhone项目的`Base SDK`和`Deployment Target`。

1、Base SDK指的是当前编译用的SDK版本。简单来说就是表示当前使用的Xcode所支持的最高的SDK的版本，比如“iOS 6.1”，就表示当前使用iOS6.1的SDK来编译程序，项目可以使用的最高的SDK是6.1。可以在Target->Build Setting中设置，参照如下。

![xcode_base_sdk.png](/images/ios_version/xcode_base_sdk.png)

2、Deployment Target指的是编译出的程序将在哪个系统版本上运行。简单来说是当前的设置的Target所支持的最低的iOS的版本比如经常我们会设置为5.0就表示应用最低能运行在iOS5.0上（前提是没有使用高于5.0的API）。参考设置如下所示。

![xcode_deployment_target.png](/images/ios_version/xcode_deployment_target.png)

##条件编译

每次发布新版的SDK，系统都会添加一些新的有用的API，比如在iOS5.0的时候就引入了`NSJSONSerialization`的类用于解析JSON数据，在iOS5之前开发者要解析JSON就必须使用第三方库比如SBJSON、JSONKit。从效率上来看`SBJSON、JSONKit`在解析或者生成JSON数据都不如`NSJSONSerialization`高。但是为了兼容低版本像这样的地方需要做一些处理，在低版本时使用低版本的解决方案，在高版本的时候使用高版本的方案。

1、系统预设的宏定义

(1)当前支持运行的最低版本，这个相当于前面在Xcode中配置的`Deployment Target`

`__IPHONE_OS_VERSION_MIN_REQUIRED`

(2)当前支持的最高版本，相当于前面Xcode中选择的`Base SDK`

`__IPHONE_OS_VERSION_MAX_ALLOWED`

(3)各个系统版本的宏定义

```
#define __IPHONE_2_0     20000
#define __IPHONE_2_1     20100
#define __IPHONE_2_2     20200
#define __IPHONE_3_0     30000
#define __IPHONE_3_1     30100
#define __IPHONE_3_2     30200
#define __IPHONE_4_0     40000
#define __IPHONE_4_1     40100
#define __IPHONE_4_2     40200
#define __IPHONE_4_3     40300
#define __IPHONE_5_0     50000
#define __IPHONE_5_1     50100
#define __IPHONE_6_0     60000
#define __IPHONE_6_1     60100
#define __IPHONE_7_0     70000
#define __IPHONE_7_1     70100
```

2、针对不同版本使用不同的API

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= __IPHONE_7_0
    // iOS SDK 7.0 以后版本的处理
#else
    // iOS SDK 7.0 之前版本的处理
#endif
```

```
#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_4_3    
	#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_7_0
		// iOS SDK 7.0 以后版本的处理    
	#else        
		// iOS SDK 5.0 ~ 6.1版本的处理    
	#endif
#else
    // iOS SDK 4.3 之前版本的处理
#endif
```

##参考资料

1、[《iPhoneSDK版本宏》](http://blog.163.com/ray_jun/blog/static/1670536422012429104151970/)

2、[《关于iOS和OS X废弃的API》](http://blog.csdn.net/u010969412/article/details/30975301)

3、[《解决多版本SDk的兼容问题》](http://blog.csdn.net/xianghuibeijing/article/details/6259824)
