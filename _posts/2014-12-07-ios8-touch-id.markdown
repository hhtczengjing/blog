---
layout: post
title: "iOS8中使用TouchID校验用户身份"
date: 2014-12-07 13:09:47 +0800
comments: true
tags: iOS
---

在iOS8中，开发者们可使用向第三方应用开放了Touch ID权限的API，以便他们在应用中使用指纹认证来完成用户认证部分。相当一部分的APP（如印象笔记、新版QQ）以及在升级后采用了Touch ID来验证用户身份，用以替代过去使用一般密码或者PIN码，如下图所示：

（1）新版QQ：

![touch_id_qq.png](/images/ios8_touch_id/touch_id_qq.png)

（2）印象笔记高级版本：

![touch_id_yxbj.png](/images/ios8_touch_id/touch_id_yxbj.png)

本文主要介绍如何在应用中集成`Touch ID`来校验用户的身份。

###集成步骤

1、环境要求

（1）开发环境：Xcode 6（`iOS8 SDK`）

（2）设备要求：iPhone 5s、iPhone 6 (plus)、iPad Air 2

（3）引入头文件：`LocalAuthentication`

`#import <LocalAuthentication/LocalAuthentication.h>`这个库必须要Xcode6并且连接的是真机,才不会提示找不到的错误,即使不是iPhone5s都行. 如果是模拟器会提示找不到这个库。

2、添加验证的代码

```
- (void)doSomeAuth
{
    LAContext *myContext = [[LAContext alloc] init];
    myContext.localizedFallbackTitle = @"输入密码";
    NSError *authError = nil;
    NSString *myLocalizedReasonString = @"用于解除系统锁定!";
    if ([myContext canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&authError])
    {
        [myContext evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics
                  localizedReason:myLocalizedReasonString
                            reply:^(BOOL success, NSError *error) {
                                if(success)
                                {
                                    //处理验证通过
                                }
                                else
                                {
                                    //处理验证失败
                                }
                            }];
    }
    else
    {
        //不支持Touch ID验证，提示用户
    }
}
```

说明：

（1）localizedFallbackTitle：用于设置左边的按钮的名称，默认是`Enter Password`.

（2）localizedReason：用于设置提示语，表示为什么要使用Touch ID，如上面例子的`解锁印象笔记账户`和`通过验证指纹解锁QQ`等。

3、验证错误码描述

```
- (NSString *)getAuthErrorDescription:(NSInteger)code
{
    NSString *msg = @"";
    switch (code) {
        case LAErrorTouchIDNotEnrolled:
            //认证不能开始,因为touch id没有录入指纹.
            msg = @"此设备未录入指纹信息!";
            break;
        case LAErrorTouchIDNotAvailable:
            //认证不能开始,因为touch id在此台设备尚是无效的.
            msg = @"此设备不支持Touch ID!";
            break;
        case LAErrorPasscodeNotSet:
            //认证不能开始,因为此台设备没有设置密码.
            msg = @"未设置密码,无法开启认证!";
            break;
        case LAErrorSystemCancel:
            //认证被系统取消了,例如其他的应用程序到前台了
            msg = @"系统取消认证";
            break;
        case LAErrorUserFallback:
            //认证被取消,因为用户点击了fallback按钮(输入密码).
            msg = @"选择输入密码!";
            break;
        case LAErrorUserCancel:
            //认证被用户取消,例如点击了cancel按钮.
            msg = @"取消认证!";
            break;
        case LAErrorAuthenticationFailed:
            //认证没有成功,因为用户没有成功的提供一个有效的认证资格
            msg = @"认证失败!";
            break;
        default:
            break;
    }
    return msg;
}
```

###参考资料

1、[《Touch ID Tutorial for Objective-C》](http://www.devfright.com/touch-id-tutorial-objective-c/)

2、[《关于iOS8中Touch id的一些研究》](http://www.tuicool.com/articles/zEbEjaB)

3、[《细数那些集成了Touch ID的iOS 8应用》](http://www.cocoachina.com/apple/20140918/9656.html)
