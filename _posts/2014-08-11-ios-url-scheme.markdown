---
layout: post
title: "iOS中的URL Scheme"
date: 2014-08-11 19:07:27 +0800
comments: true
tags: iOS
---

   在iOS的SDK中提供了一个非常有意思的功能，它能将iOS的Application同自定义的URL Schema绑定，同时可以通过`URL Scheme`在浏览器或者是其他应用中启动这个Application。本文主要介绍如何通过URL Scheme的方式启动应用和参数的传递。

##创建URL Scheme

1、首先在*-Info.plist中添加一行,选择`URL types`，效果如下图所示：

![ios_url_scheme_001.png](/images/ios_url_scheme/ios_url_scheme_001.png)

2、在展开的Item 0中填写`URL identifier`,这个用来唯一标识用户自定义的URL Scheme，推荐使用域名的反转形式，如:com.devzeng.demo

![ios_url_scheme_002.png](/images/ios_url_scheme/ios_url_scheme_002.png)

3、在Item 0中添加新的一行，选择`URL Schemes`

![ios_url_scheme_003.png](/images/ios_url_scheme/ios_url_scheme_003.png)

4、展开`URL Schemes`，在Item 0中输入自定义的Scheme的名称。在这里只需要输入自定义的Scheme的名称即可，不需要加上`://`，例如这里输入的是`devzeng`,那么对应的自定义的URL就是`devzeng://`，这里可以输入多个。

![ios_url_scheme_004.png](/images/ios_url_scheme/ios_url_scheme_004.png)

5、最后一个完整的示例效果图：

![ios_url_scheme_005.png](/images/ios_url_scheme/ios_url_scheme_005.png)

对应的源码配置文件为:

```
<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLName</key>
			<string>com.devzeng.demo.urlschema</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>devzeng</string>
			</array>
		</dict>
	</array>
```


##使用URL Scheme

1、在Safari中使用

在Safari中直接在浏览器的地址栏中输入`devzeng://`,即可启动刚才的应用

2、在其他的应用程序中使用

在需要调用的地方使用下面的代码即可实现调用

```
NSString *customURL = @"devzeng://";
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:customURL]];
```

3、参数的传递

```
- (void)openOtherApp
{
    NSString *customURL = @"devzeng://?token=123abct&registered=1";
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:customURL]];
}
```

在AppDelegate中可以实现下面的两个方法

`- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url`

`- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation`

说明:

(1)上面的两个函数作用是一致的只是参数不同而已，函数的返回值是BOOL,如果为YES表示可以打开，NO表示不可以打开应用程序

(2)参数可以通过`[url query]`来获取，比如使用的是`devzeng://?token=123abct&registered=1`那么通过`[url query]`获取到的值是`token=123abct&registered=1`,然后可以通过这些数据再作相应的处理.

(3)调用的应用程序的Bundle ID可以通过`sourceApplication`参数获取

(4)通过`[url scheme]`可以获取到请求的URL Scheme，比如是通过`devzeng://`打开的那么`[url scheme]`的值就是`devzeng`。可以通过不同的参数来判断来源的合法性

(5)示例

```
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    if ([sourceApplication isEqualToString:@"com.devzeng.demo.urlscheme"])
    {
        NSLog(@"调用的应用程序的Bundle ID是: %@", sourceApplication);
        NSLog(@"URL scheme:%@", [url scheme]);
        NSLog(@"URL query: %@", [url query]);
        return YES;
    }
    else
    {
        return NO;
    }
}
```

##参考资料
1、[《通过自定义的URL Scheme启动你的App》](http://blog.csdn.net/ba_jie/article/details/6884818)

2、[《The Complete Tutorial on iOS/iPhone Custom URL Schemes》](http://iosdevelopertips.com/cocoa/launching-your-own-application-via-a-custom-url-scheme.html)
