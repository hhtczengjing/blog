---
layout: post
title: "iOS动态切换APP图标"
date: 2023-09-23 19:34:25 +0800
comments: true
tags: iOS
---

有些时候需要App能在一些特殊的日子更换桌面图标，或者是出于一些个性化的需求需要能让用户自己选择自己喜欢的图标（比如VIP用户可以显示会员专属的图标）。如“微博和百度云盘的VIP个性化设置就提供了切换图标相关的功能。

![demo](/images/ios_alternate_appicon/demo.png)

## 技术实现

从 iOS 10.3 开始系统提供了动态切换图标相关的实现，要求提前将支持切换的可选图标提前预埋到APP并添加相关的配置，然后根据业务需要在合适的时机进行切换即可。

### 配置相关

以`百度云盘`的配置为例:

![info_plist](/images/ios_alternate_appicon/info_plist.png)

```
<key>CFBundleIcons</key>
<dict>
    <key>CFBundleAlternateIcons</key>
    <dict>
        <key>svip_app_icon</key>
        <dict>
          <key>CFBundleIconFiles</key>
          <array>
            <string>svip_app_icon</string>
          </array>
          <key>UIPreernderedIcon</key>
          <false/>
        </dict>
    </dict>
</dict>
```

> 说明：

- （1）填写的文件名可以不需要指定后缀名(默认是png)，`@2x/@3x` 可以统一不添加，系统会自动寻找合适的尺寸
- （2）iPad版本如果需要支持，可以在Info.plist的 `CFBundleIcons~iPad`下再次添加一套即可
- （3）替换后桌面、推送、设置几处的图标都会被替换
- （4）图片当前的方案不支持放在 Assets.xcassets 里面（需要直接放在MainBundle里面）
- （5）图片尺寸推荐：iPhone(120x120, 180x180), iPad(167x167, 152x152)
- （6）按照文档，需要在CFBundleIcons里面配置CFBundlePrimaryIcon这个主图标对应的内容，实际只在 Assets.xcassets 中配置图标即可，系统会自动处理
- （7）UIPrerenderedIcon的value是BOOL值。这个键值所代表的作用在iOS7之后（含iOS7）已失效，在iOS6中可渲染app图标为带高亮效果，所以这个值目前可以不用关心

### 代码相关

#### 1、获取当前设置的图标名称

```
+ (nullable NSString *)currentAlternateIconName {
    if (@available(iOS 10.3, *)) {
        if (![[UIApplication sharedApplication] supportsAlternateIcons]) {
            return nil;
        }
        return [[UIApplication sharedApplication] alternateIconName];
    } else {
        return nil;
    }
}
```

> 说明：如果返回的结果为空表示为默认图标

#### 2、切换图标

```
+ (void)changeAlternateIconName:(NSString *_Nullable)iconName completion:(nullable void (^)(NSError *_Nullable error))completion {
    if (@available(iOS 10.3, *)) {
        if (![[UIApplication sharedApplication] supportsAlternateIcons]) {
            !completion ? : completion([NSError errorWithDomain:@"error" code:0 userInfo:@{@"message": @"不支持动态切换图标"}]);
            return;
        }
        if (!iconName || ![iconName isKindOfClass:[NSString class]] || iconName.length <= 0) {
            iconName = nil;
        }
        [[UIApplication sharedApplication] setAlternateIconName:iconName completionHandler:^(NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                !completion ? : completion(error);
            });
        }];
    } else {
        !completion ? : completion([NSError errorWithDomain:@"error" code:0 userInfo:@{@"message": @"不支持动态切换图标"}]);
    }
}
```

> 说明：如果iconName传空表示恢复为默认图标

### 存在的一些问题

##### 1、切换过程中会出现一个弹框

> 解决办法：

可以通过hook UIViewController 的 `presentViewController:animated:completion:`方法。核心代码如下（说明：该方法可能存在误判，请谨慎使用。）：

```
@implementation UIViewController (Additions)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method presentM = class_getInstanceMethod(self.class, @selector(presentViewController:animated:completion:));
        Method presentSwizzlingM = class_getInstanceMethod(self.class, @selector(zjh_presentViewController:animated:completion:));
        method_exchangeImplementations(presentM, presentSwizzlingM);
    });
}

- (void)dz_presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)flag completion:(void (^)(void))completion {
    if ([viewControllerToPresent isKindOfClass:[UIAlertController class]]) {
        UIAlertController *alertController = (UIAlertController *)viewControllerToPresent;
        NSArray *childViewControllers = alertController.childViewControllers;
        if (childViewControllers && childViewControllers.count > 0) {
            id vc = [childViewControllers objectAtIndex:0];
            NSString *className = [@[@"_", @"UI", @"Alternate", @"Application", @"Icons", @"Alert", @"Content", @"ViewController"] componentsJoinedByString:@""];
            if ([NSStringFromClass([vc class]) isEqualToString:className]) {
                return;
            }
        }
    }
    [self dz_presentViewController:viewControllerToPresent animated:flag completion:completion];
}
```

##### 2、在 didFinishLaunchingWithOptions 里面进行切换时候可能会失败

> 解决办法：

使用延时处理适当延后一点执行

##### 3、执行切换的过程中App进入后台可能会失败

> 解决办法：

每次启动都进行一次检测