---
layout: post
title: "iOS开发中Xcode常见使用技巧"
date: 2014-08-27 23:07:17 +0800
comments: true
tags: iOS
---

Xcode是苹果公司开发向开发人员提供的集成开发环境(IDE)，主要用于开发Mac OS X和iOS的应用程序只能运行于Mac操作系统下。本文旨在记录一些在开发中常见的Xcode的配置和使用技巧，后续会持续更新。

![xcode6_welcome.png](/images/xcode_usage/xcode6_welcome.png)

##更新说明

- 2014-08-27 v1.0 初稿
- 2014-09-24 V1.1 增加Xcode断点不进调试的相关内容
- 2014-09-26 V1.2 增加Xcode6缺失新建`Empty Application`相关模板的问题


##旧版本Xcode支持高版本的设备

问题描述：如何解决Xcode5.0无法识别iOS7.1以上的设备的问题？

解决办法：

(1)首先在终端下进入到Xcode的安装目录下面的DeviceSupport目录下，如下所示：

`cd /Applications/Xcode5.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport`

(2)然后执行下面的代码（目前iOS7最新的版本号是7.1.2 11D257）,如何看当前iOS的最新版本号可以到手机的`设置->通用->关于本机->版本`里面看到如`7.1.2(11D257)`

`ln -s ./7.0.3\ \(11B508\) ./7.1.2\ \(11D257\)`

(3)重启Xcode，下次启动Xcode的时候检查设备耗时会稍微长一些

##安装旧版SDK

有时候在开发过程中需要使用旧版的SDK进行编译，比如之前的博文中有提到为了简化iOS7的适配工作有时候会直接使用iOS7以下的SDK编译打包的程序，下面的步骤介绍如何在Xcode中安装旧版的SDK（`前提条件是安装新版前需要按照下面的要求备份`）。

(1)安装新版的Xcode前需要到下面的目录下备份SDK

`/Applications/Xcode4.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs`

如：iOS6.1 SDK(`iPhoneOS6.1.sdk`)、iOS7.0 SDK(`iPhoneOS7.0.sdk`)

(2)复制上面备份的SDK按照上面的路径

`/Applications/Xcode6-Beta.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs`

注：Xcode6-Beta.app这个指的是Xcode的名称


##安装旧版模拟器

   每次安装新版本的Xcode，默认只会安装最新版本的模拟器，虽然有些模拟器可以在设置里面下载，但是那些更老的版本就无法在Xcode的设置中下载了，而且下载一个旧版的Xcode耗时会非常长。

(1)安装新版的Xcode前需要到下面的目录下备份SDK

`/Applications/Xcode5.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/`

如：iOS6.1 SDK(`iPhoneOS6.1.sdk`)、iOS7.0 SDK(`iPhoneOS7.0.sdk`)

(2)复制上面备份的SDK按照上面的路径

`/Applications/Xcode6-Beta.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/`

P.S.`建议每次安装新Xcode的时候，把模拟器和SDK都备份一下。`

##模拟器中的应用程序数据

使用模拟器开发一段时间后在模拟器上会安装很多我们写的应用程序，那么这些应用程序是存储在哪里呢？
其实模拟器中的程序存储在：
`~/Library/Application Support/iPhone Simulator/7.0/Applications` 目录下。

如果要删除所安装的程序，也可以直接将Applications目录下的文件夹删掉，也可以直接把模拟器还原(但是设置的数据也被重置了)。

##Xcode调试不进断点的问题

安装多个Xcode后有时候会出现断点调试的时候无法定位到断点指定的位置而是进入堆栈页面,如下图：

![xcode_debug_error_1.png](/images/xcode_usage/xcode_debug_error_1.png)

解决办法是：

将Xcode导航栏`Debug -> Debug Workflow -> Always Show Disassembly`前面的勾选去掉即可

![xcode_debug_error_1.png](/images/xcode_usage/xcode_debug_error_2.png)

##Xcode6缺失相关模板的问题

更新到新版本的Xcode6后，新建一个项目发现缺少以前常用的`Empty Application`模板，如下所示：

![xcode_new_application_1.png](/images/xcode_usage/xcode_new_application_1.png)

如何在Xcode6中中添加缺失的`Empty Application`模板呢？解决的办法如下：

1、到GitHub上下载缺失的模板数据

`git clone https://github.com/cDigger/AddMissingTemplates.git`

2、复制`AddMissingTemplates`目录下面的`Empty Application.xctemplate`到如下目录

`Xcode6.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/Project Templates/iOS/Application/`

重启Xcode后，新建项目效果如下：

![xcode_new_application_2.png](/images/xcode_usage/xcode_new_application_2.png)

##参考资料

1、[《Xcode4使用技巧》](http://blog.devtang.com/blog/2012/03/10/xcode4-tips/)

2、[《在Xcode中安装低版本的SDK和模拟器》](http://vbtboy.iteye.com/blog/1956696)
