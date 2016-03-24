---
layout: post
title: "iOS8中Today Extension的使用"
date: 2014-11-22 23:49:18 +0800
comments: true
tags: iOS
---

扩展（Extension）是iOS 8中引入的一个非常重要的新特性。扩展让app之间的数据交互成为可能。用户可以在app中使用其他应用提供的功能，而无需离开当前的应用。

iOS 8系统有6个支持扩展的系统区域，分别是Today、Share、Action、Photo Editing、Storage Provider、Custom keyboard。支持扩展的系统区域也被称为扩展点。对于赛事比分，股票、天气、快递这类需要实时获取的信息，可以在通知中心的Today视图中创建一个Today扩展实现。Today扩展又称为Widget。本文主要是介绍Today Extension的用法。

![ios8_extension.png](/images/ios8_today_extension/ios8_today_extension.png)

###创建步骤

1、选择`File`->`New`->`Target`创建一个新的Target，如下图所示：

![iOS8_today_extension](/images/ios8_today_extension/ios8_today_extension_001.png)

2、选择`Application Extension`->`Today Extension`（下图中最后一个），指定需要创建的Extension的类型，这里选择`Today Extension`，如下图所示：

![ios8_today_extension](/images/ios8_today_extension/ios8_today_extension_002.png)

3、按照要求填写`Product Name`、`Organization Name`和`Language`，如下图所示：

![ios8_today_extension](/images/ios8_today_extension/ios8_today_extension_003.png)

4、点击`Finish`后Xcode会提示是否需要立即激活当前创建的Scheme，默认选择cancel就行了，Xcode会自动创建一个`Today Extension`的Target，默认会创建如下几个文件：

- `TodayViewController.h/TodayViewController.m`

- `MainInterface.storyboard`

- `Supporting Files/Info.plist`

5、启动应用后在通知中心添加就能看到运行效果，如下图所示：

![ios8_today_extension_004.png](/images/ios8_today_extension/ios8_today_extension_004.png)

###开发实例

1、配置Info.plist文件，使用代码布局

Xcode6中新建的Today Extension默认是使用StoryBoard布局的，如果需要使用代码布局就需要配置Info.plist文件，步骤如下：

（1）删除`NSExtensionMainStoryboard`的配置，如下所示：

```
<key>NSExtensionMainStoryboard</key>
<string>MainInterface</string>
```

（2）添加`NSExtensionPrincipalClass`的配置，如下所示：

```
<key>NSExtensionPrincipalClass</key>
<string>TodayViewController</string>
```
（3）修改`CFBundleDisplayName`，调整Widget显示的名称

```
<key>CFBundleDisplayName</key>
<string>显示的名称</string>
```

2、调整Widget的高度

`self.preferredContentSize = CGSizeMake(0, 200);`

取消widget默认的inset，让应用靠左

```
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets
{
    return UIEdgeInsetsZero;
}
```

3、示例运行效果如下：

![ios8_today_extension_005.png](/images/ios8_today_extension/ios8_today_extension_005.png)

布局代码如下：【以下代码使用了Masonry来进行AutoLayout处理】

```
- (void)makeView
{
    __weak __typeof(&*self)ws = self;
    //内容视图
    self.contentView = [UIView new];
    self.contentView.backgroundColor = [UIColor clearColor];
    [self.view addSubview:self.contentView];
    self.contentView.userInteractionEnabled = YES;
    [self.contentView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(ws.view);
        make.height.mas_equalTo(@44).priorityHigh();
    }];
    //Label1
    self.priceLabel = [UILabel new];
    self.priceLabel.textColor = [UIColor colorWithRed:66.0/255 green:145.0/255 blue:211.0/255 alpha:1];
    self.priceLabel.text = @"$592.12";
    [self.contentView addSubview:self.priceLabel];
    [self.priceLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(ws.contentView.mas_left).with.offset(20);
        make.top.equalTo(ws.contentView.mas_top).with.offset(10);
        make.size.mas_equalTo(CGSizeMake(62, 21));
    }];
    //切换按钮
    self.toggleLineChartButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [self.contentView addSubview:self.toggleLineChartButton];
    [self.toggleLineChartButton addTarget:self action:@selector(toggleLineChartViewVisible:) forControlEvents:UIControlEventTouchUpInside];
    [self.toggleLineChartButton setImage:[UIImage imageNamed:@"caret-notification-center"] forState:UIControlStateNormal];
    [self.toggleLineChartButton mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(ws.contentView.mas_right).with.offset(0);
        make.top.equalTo(ws.contentView.mas_top).with.offset(0);
        make.size.mas_equalTo(CGSizeMake(44, 44));
    }];
    //Label2
    self.priceChangeLabel = [UILabel new];
    self.priceChangeLabel.textColor = [UIColor colorWithRed:133.0/255 green:191.0/255 blue:37.0/255 alpha:1];
    self.priceChangeLabel.text = @"+1.23";
    [self.contentView addSubview:self.priceChangeLabel];
    [self.priceChangeLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(ws.toggleLineChartButton.mas_left).with.offset(-20);
        make.top.equalTo(ws.contentView).with.offset(10);
        make.size.mas_equalTo(CGSizeMake(44, 21));
    }];
    //Label3
    self.detailLabel = [[UILabel alloc] init];
    self.detailLabel.backgroundColor = [UIColor clearColor];
    self.detailLabel.numberOfLines = 0;
    [self.contentView addSubview:self.detailLabel];
    self.detailLabel.text = @"\n\n详细信息";
    self.detailLabel.textAlignment = NSTextAlignmentCenter;
    [self.detailLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(ws.priceLabel.mas_bottom).with.offset(0);
        make.left.equalTo(ws.contentView);
        make.width.equalTo(ws.contentView);
        if(self.lineChartIsVisible)
        {
            self.detailLabel.textColor = [UIColor whiteColor];
            make.height.mas_equalTo(@98);
        }
        else
        {
            self.detailLabel.textColor = [UIColor clearColor];
            make.height.mas_equalTo(@0);
        }
    }];
}
```

按钮点击处理：

```
- (void)toggleLineChartViewVisible:(id)sender
{
    __weak __typeof(&*self)ws = self;
    //重新布局
    if(self.lineChartIsVisible)
    {
        self.preferredContentSize = CGSizeMake(0, 44);
        [self.detailLabel mas_remakeConstraints:^(MASConstraintMaker *make) {
            make.height.mas_equalTo(@0);
            ws.detailLabel.textColor = [UIColor clearColor];
            ws.detailLabel.textAlignment = NSTextAlignmentCenter;
        }];
        //改变按钮的方向
        self.toggleLineChartButton.transform = CGAffineTransformMakeRotation(0);
        self.lineChartIsVisible = NO;
        [self.contentView mas_updateConstraints:^(MASConstraintMaker *make) {
            make.height.mas_equalTo(@44).priorityHigh();
        }];
    }
    else
    {
        self.preferredContentSize = CGSizeMake(0, 142);
        [self.detailLabel mas_remakeConstraints:^(MASConstraintMaker *make) {
            make.height.mas_equalTo(@98);
            ws.detailLabel.textColor = [UIColor whiteColor];
            ws.detailLabel.textAlignment = NSTextAlignmentCenter;
        }];
        //改变按钮的方向
        self.toggleLineChartButton.transform = CGAffineTransformMakeRotation(180.0 * M_PI/180.0);
        self.lineChartIsVisible = YES;
        [self.contentView mas_updateConstraints:^(MASConstraintMaker *make) {
            make.height.mas_equalTo(@142).priorityHigh();
        }];
    }
}
```

###参考资料

1、[《iOS 8 Extensions》](http://www.cnblogs.com/xdream86/p/3855932.html)

2、[《App Extension Programming Guide》](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1)

3、[《如何用纯代码构建一个Widget(today extension)》](http://adad184.com/2014/10/29/how-to-setup-today-extension-programmatically/)

4、[《WWDC2014之App Extensions学习笔记》](http://wangzz.github.io/blog/2014/06/23/wwdc2014zhi-app-extensionsxue-xi-bi-ji/)