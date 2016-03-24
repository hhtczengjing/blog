---
layout: post
title: "iOS中Info.plist文件的常见配置"
date: 2014-08-24 16:06:02 +0800
comments: true
tags: iOS
---

在创建一个新的Xcode工程后，会在`Supporting Files`文件夹下自动生成一个`工程名-Info.plist`的文件，这个是对工程做一些运行期配置的文件(很重要，必须有该文件)。如果使用文本编辑器打开这个文件，会发现这是一个XML格式的文本文件，使用Xcode的`Open As`->`Source Code`或者`Property List`可以进行编辑，本文会重点介绍一些在iOS开发中常见的的Info.plist的配置项。

##Info.plist配置项说明

1、设置启动图标(`CFBundleIcons`)

```
<key>CFBundleIcons</key>
<dict>
	<key>CFBundlePrimaryIcon</key>
	<dict>
		<key>CFBundleIconFiles</key>
		<array>
			<string>Icon</string>
			<string>Icon@2x</string>
			<string>Icon_120@2x</string>
		</array>
	</dict>
</dict>
```

2、设置启动闪屏图片(`UILaunchImages`)

```
<key>UILaunchImages</key>
<array>
	<dict>
		<key>UILaunchImageMinimumOSVersion</key>
		<string>7.0</string>
		<key>UILaunchImageName</key>
		<string>Default</string>
		<key>UILaunchImageOrientation</key>
		<string>Portrait</string>
		<key>UILaunchImageSize</key>
		<string>{320, 568}</string>
	</dict>
	<dict>
		<key>UILaunchImageMinimumOSVersion</key>
		<string>7.0</string>
		<key>UILaunchImageName</key>
		<string>Default</string>
		<key>UILaunchImageOrientation</key>
		<string>Portrait</string>
		<key>UILaunchImageSize</key>
		<string>{320, 480}</string>
	</dict>
</array>
```

3、设置版本号相关

(1)设置Bundle的版本号(`Bundle versions string, short`)。

一般包含该束的主、次版本号，这个字符串的格式通常是“n.n.n”（n表示某个数字，如1.1.1）。第一个数字是束的主要版本号，另两个是次要版本号。该关键字的值会被显示在Cocoa应用程序的关于对话框中。该关键字不同于CFBundleVersion，它指定了一个特殊的创建号。而CFBundleShortVersionString的值描述了一种更加正式的并且不随每一次创建而改变的版本号。

```
<key>CFBundleShortVersionString</key>
<string>1.0</string>
```

(2)设置应用程序版本号(`Bundle version`)。

每次部署应用程序的一个新版本时，将会增加这个编号，用于标识不同的版本。

```
<key>CFBundleVersion</key>
<string>1.0</string>
```

4、设置字体相关(`Fonts provided by application`)

在iOS应用中需要使用系统提供的字体之外的字体，可以将字体文件(`.ttf/.odf`)复制到项目文件中，另外需要在Info.plist中添加`Fonts provided by application`的项，对应的源码文件如下：

```
<key>UIAppFonts</key>
<array>
	<string>华文行楷.ttf</string>
	<string>华文新魏.ttf</string>
	<string>黑体_GB2312.ttf</string>
</array>
```
P.S关于如何使用系统支持的字体信息:

(1)在调用字体的时候，要使用字体名。字体名不是文件名，而是字体的`Family Name`。Family Name可以在Font Book中查看。

`label.font = [UIFont fontWithName:@"字体名称" size:16.0];`

(2)遍历出系统支持的全部字体

```
NSArray *familyNames = [[NSArray alloc] initWithArray:[UIFont familyNames]];
for(int indFamily = 0; indFamily < familyNames.count; ++indFamily)
{
	NSLog(@"Family Name: %@", [familyNames objectAtIndex:indFamily]);
	NSString *fontFamilyName = [familyNames objectAtIndex:indFamily];
	NSArray *fontNames = [[NSArray alloc] initWithArray:[UIFont fontNamesForFamilyName:fontFamilyName]];
	for(int indFont = 0; indFont < fontNames.count; ++indFont)
	{
		NSLog(@"   Font Name: %@", [fontNames objectAtIndex:indFont]);
	}
}
```

5、设置应用名称(`Bundle display name`)

```
<key>CFBundleDisplayName</key>
<string>应用程序名称</string>
```
可以通过在InfoPlist.strings中使用配置让应用在不同的语言环境下显示不同的应用名称，如在`English`中使用`CFBundleDisplayName="Hello World";`配置应用程序的名称为`Hello World`，在`Chinese`的环境下使用`CFBundleDisplayName="你好世界";`配置应用程序的名称为`你好世界`。

6、设置应用标识号(`Bundle identifier`)

```
<key>CFBundleIdentifier</key>
<string>com.devzeng.demo</string>
```

7、设置应用支持的屏幕方向(`Supported interface orientations`)

iOS应用程序支持以下四个方向的设置：`UIInterfaceOrientationPortrait`(默认竖直方向，HOME键向下)、`UIInterfaceOrientationLandscapeLeft`(横屏靠左)、`UIInterfaceOrientationLandscapeRight`(横屏向右)和`UIInterfaceOrientationPortraitUpsideDown`(竖直方向倒置，HOME键向上)

对应的配置源码如下:

```
<key>UISupportedInterfaceOrientations</key>
<array>
	<string>UIInterfaceOrientationPortrait</string>
	<string>UIInterfaceOrientationLandscapeLeft</string>
	<string>UIInterfaceOrientationLandscapeRight</string>
	<string>UIInterfaceOrientationPortraitUpsideDown</string>
</array>
```

8、设置应用程序是否支持后台运行(`Application does not run in background`)

通过`UIApplicationExitsOnSuspend`可以设置iOS的应用程序进入到挂起状态下是否立即退出，设置为`YES`表示不支持后台运行退出到后台立即退出，设置为`NO`表示支持后台运行。

(1)设置支持后台运行

```
<key>UIApplicationExitsOnSuspend</key>
<false/>
```

(2)设置不支持后台运行

```
<key>UIApplicationExitsOnSuspend</key>
<true/>
```

##参考资料

1、[《Information Property List Key Reference》](https://developer.apple.com/library/ios/documentation/general/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW1)

2、[《iOS工程中的info.plist文件的完整研究》](http://blog.csdn.net/nicktang/article/details/6875234)

3、[《在iOS程序中使用自定义字体》](http://www.cnblogs.com/gaoxiao228/archive/2012/07/04/2576645.html)