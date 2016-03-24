---
layout: post
title: "iOS8中的Safari Action Extension"
date: 2015-03-29 20:12:53 +0800
comments: true
tags: iOS
---

扩展（Extension）是iOS8中引入的一个非常重要的新特性。扩展让app之间的数据交互成为可能。用户可以在app中使用其他应用提供的功能，而无需离开当前的应用。前面介绍过关于[Today Widget](http://blog.devzeng.com/blog/ios8-today-extension.html)的使用，本文介绍Action Extension在iOS8中的开发。

![201111194125034.png](/images/action_extension/201111194125034.png)

###创建Action Extension

1、使用Xcode6创建一个iOS工程，菜单栏File->New->Target出现下面的可选项，选择Action Extension:

![action_extension_01.png](/images/action_extension/action_extension_01.png)

2、配置Extension的Product Name等内容。

![action_extension_02.png](/images/action_extension/action_extension_02.png)

###配置Action Extension

1、默认Action Extension中的布局使用StoryBoard，如果不想使用StoryBoard布局将Info.plist中的如下配置：

```
<key>NSExtensionMainStoryboard</key>
<string>MainInterface</string>
```

改为下面的配置：

```
<key>NSExtensionPrincipalClass</key>
<string>ActionViewController</string>
```

说明：

插件在UI上以UIViewController模式存在，被parentViewController（Host App）以模态窗口形式弹出（present as modal viewController）。插件工程在Info.plist的NSExtension中通过NSExtensionMainStoryboard指定UI视图入口。当然，如果不想使用storyboard，也可以使用NSExtensionPrincipalClass指定自定义UIViewController子类名（也可以封装到UINavigationController）。


2、NSExtensionActivationRule定义了当前的扩展支持的数据类型及数据项个数，例如设置只支持图片格式和视频格式的数据，并且最多不超过10张图片和1个视频。

```
<key>NSExtensionActivationSupportsWebPageWithMaxCount</key>
<string>1</string>
```

3、NSExtensionJavaScriptPreprocessingFile用于配置与脚本交互的JS脚本文件的名字。为了告知Safari你的应用扩展中包含一个JavaScript文件，你需要在应用扩展的Info.plist文件中，向NSExtensionAttributes字典添加NSExtensionJavaScriptPreprocessingFile关键字来指明你的JavaScript文件。这个关键字的值就是你希望当你的应用扩展运行前，Safari要加载的JavaScript文件的名称。比如：

```
<key>NSExtensionJavaScriptPreprocessingFile</key>
<string>ScreenShotJavaScript</string>
```

4、NSExtensionPointIdentifier用于表示扩展点，每一个扩展点拥有一个唯一的名字。

```
<key>NSExtensionPointIdentifier</key>
<string>com.apple.ui-services</string>
```

###示例程序

实现一个网页截屏的功能，用户在Safari中的Action选择指定的动作，然后将网页的内容以图片的形式保存下来，类似于[`Screenshot`](https://itunes.apple.com/cn/app/awesome-screenshot-for-safari/id918780145?l=en&mt=8).

![action_extension_03.png](/images/action_extension/action_extension_03.png)

1、将UIWebView进行截屏

```
- (UIImage *)screenshot
{
    CGSize boundsSize = self.bounds.size;
    CGFloat boundsWidth = self.bounds.size.width;
    CGFloat boundsHeight = self.bounds.size.height;
    
    CGPoint offset = self.scrollView.contentOffset;
    [self.scrollView setContentOffset:CGPointMake(0, 0)];
    
    CGFloat contentHeight = self.scrollView.contentSize.height;
    NSMutableArray *images = [NSMutableArray array];
    while (contentHeight > 0) {
        UIGraphicsBeginImageContext(boundsSize);
        [self.layer renderInContext:UIGraphicsGetCurrentContext()];
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        [images addObject:image];
        
        CGFloat offsetY = self.scrollView.contentOffset.y;
        [self.scrollView setContentOffset:CGPointMake(0, offsetY + boundsHeight)];
        contentHeight -= boundsHeight;
    }
    [self.scrollView setContentOffset:offset];
    
    UIGraphicsBeginImageContext(self.scrollView.contentSize);
    [images enumerateObjectsUsingBlock:^(UIImage *image, NSUInteger idx, BOOL *stop) {
        [image drawInRect:CGRectMake(0, boundsHeight * idx, boundsWidth, boundsHeight)];
    }];
    UIImage *fullImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return fullImage;
}
```

2、在ActionViewController中获取网页的URL地址

```
for (NSExtensionItem *item in self.extensionContext.inputItems) {
	for (NSItemProvider *itemProvider in item.attachments) {
		__weak UIWebView *webView = self.myWebView;
		NSString *identifier = (NSString *)kUTTypeURL;
		BOOL status = [itemProvider hasItemConformingToTypeIdentifier:identifier];
		if(status){
			[itemProvider loadItemForTypeIdentifier:identifier options:nil completionHandler:^(NSURL *item, NSError *error) {
                    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                        [webView loadRequest:[NSURLRequest requestWithURL:item]];
                    }];
             }];		
             break;
        }
    }
}
```

###参考资料

1、[《将UIWebView显示的内容转为图片和PDF》](http://www.cnblogs.com/tracy-e/p/3151463.html)

2、[《如何给UITableView 或 UIScrollView 的content 做截图》](http://blog.csdn.net/songhongri/article/details/43270293)

3、[《App Extension Programming Guide》](https://developer.apple.com/library/ios/documentation/General/Conceptual/ExtensibilityPG/Services.html#//apple_ref/doc/uid/TP40014214-CH13-SW1)

4、[《iOS8扩展插件开发配置》](http://blog.csdn.net/phunxm/article/details/42715145)
