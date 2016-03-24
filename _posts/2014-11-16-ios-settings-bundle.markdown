---
layout: post
title: "iOS开发中Settings.bundle的使用"
date: 2014-11-16 16:35:48 +0800
comments: true
tags: iOS
---

在iOS开发中很多时候开发者需要让用户自行设置一些系统的配置项目，比如让用户设置是否支持在3G模式下加载数据，或者是让用户自己设置支不支持网络数据缓存的功能。另外在企业级应用开发中经常有需要对后台的访问地址进行调整那么需要用户自行的进行配置，下面是爱奇艺和招商银行的设置配置项：

![app_settings.png](/images/ios_settings_bundle/app_settings.png)

###Settings.bundle配置说明

在Settings.bundle中支持如下几种配置项：

![settings_preference_control_types.png](/images/ios_settings_bundle/settings_preference_control_types.png)

1、`Group`

Group类似于UITableView中的Group分组，用来表示一组设置项，配置如下所示：

![settings_bundle_group.png](/images/ios_settings_bundle/settings_bundle_group.png)

配置项说明：

(1)Title：表示分组的显示标题

(2)Type：默认是Group

(3)FooterText：Group的底部显示的文字内容

2、`Multi Value`

`Multi Value`是为了让用户在多个值中选择需要的内容，相当于下拉列表的形式进行选择，配置如下所示：

![settings_bundle_multi_value.png](/images/ios_settings_bundle/settings_bundle_multi_value.png)

配置项说明：

(1)Type：默认是Multi Value

(2)Title：配置项显示的标题

(3)Identifier：设置项的标识符，用于读取配置项的配置内容

(4)Default Value：默认的值，对应的是Values中的项目

(5)Titles：显示的标题的集合

(6)Values：显示的值的集合，与标题一一对应

3、`Slider`

![settings_bundle_slider.png](/images/ios_settings_bundle/settings_bundle_slider.png)

配置项说明：

(1)Type：配置类型，默认是Slider

(2)Identifier：设置项的标识符，用于读取配置项的配置内容

(3)Default Value：默认值，Number类型

(4)Minimum Value：最小值，Number类型

(5)Maximum Value：最大值，Number类型

(6)Max Value Image Filename：最大值那一端的图片。

(7)Min Value Image Filename：最小值那一端的图片。

4、`Text Field`

![settings_bundle_text_field.png](/images/ios_settings_bundle/settings_bundle_text_field.png)

配置项说明:

(1)Text Field is Secure：是否为安全文本。如果设置为YES，则内容以圆点符号出现。

(2)Autocapitalization Style：自动大写。有四个值: `None(无)`、`Sentences(句子首字母大写)`、`Words(单词首字母大写)`和`All Characters(所有字母大写)`。

(3)Autocorrection Style：自动纠正拼写，如果开启，你输入一个不存在的单词，系统会划红线提示。有三个值：`Default(默认)`、`No Autocorrection(不自动纠正)`和`Autocorrection(自动纠正)`。

(4)Keyboard Type：键盘样式。有五个值：`Alphabet(字母表，默认)`、`Numbers and Punctuation(数字和标点符号)`、`Number Pad(数字面板)`、`URL(比Alphabet多出了.com等域名后缀)`和`Email Address(比Alphabet多出了@符合)`。

5、`Title`

![settings_bundle_title.png](/images/ios_settings_bundle/settings_bundle_title.png)

配置项说明：

(1)Type：默认是Title

(2)Title：配置项显示的标题

(3)Identifier：设置项的标识符，用于读取配置项的配置内容

(4)Default Value：默认的值

6、`Toggle Switch`

`Toggle Switch`是一个类似于UISwitch的选项，用于设置简单的开启或者关闭的选项，配置如下所示：

![settings_bundle_toggle.png](/images/ios_settings_bundle/settings_bundle_toggle.png)

配置项说明：

(1)Type：默认是Toggle Switch

(2)Title：配置项显示的标题

(3)Identifier：设置项的标识符，用于读取配置项的配置内容

(4)Default Value：默认的值

###在项目中使用

1、添加Setting.bundle文件到项目中

![add_settings_bundle.png](/images/ios_settings_bundle/add_settings_bundle.png)

2、读取配置信息

```
- (void)readingPreference
{
    //获取Settings.bundle路径
    NSString *settingsBundle = [[NSBundle mainBundle] pathForResource:@"Settings" ofType:@"bundle"];
    if(!settingsBundle)
    {
        NSLog(@"找不到Settings.bundle文件");
        return;
    }
    //读取Settings.bundle里面的配置信息
    NSDictionary *settings = [NSDictionary dictionaryWithContentsOfFile:[settingsBundle stringByAppendingPathComponent:@"Root.plist"]];
    NSArray *preferences = [settings objectForKey:@"PreferenceSpecifiers"];
    NSMutableDictionary *defaultsToRegister = [[NSMutableDictionary alloc] initWithCapacity:[preferences count]];
    for(NSDictionary *prefSpecification in preferences)
    {
        NSString *key = [prefSpecification objectForKey:@"Key"];
        if(key)
        {
            [defaultsToRegister setObject:[prefSpecification objectForKey:@"DefaultValue"] forKey:key];
        }
    }
    [[NSUserDefaults standardUserDefaults] registerDefaults:defaultsToRegister];
    [[NSUserDefaults standardUserDefaults] synchronize];
    //TODO：读取指定数据
}
```

3、在AppDelegate中读取配置信息

(1)应用启动后读取配置信息

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    //读取配置文件
    [[SystemConfigHelper shared] readingPreference];
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    return YES;
}
```

(2)应用程序进入到前台后读取配置信息

```
- (void)applicationWillEnterForeground:(UIApplication *)application
{
    //读取配置信息
    [[SystemConfigHelper shared] readingPreference];
}
```

说明：

`SystemConfigHelper`是用来读取系统配置信息的工具.

###典型实例

1、[爱奇艺iPhone客户端的Settings.bundle配置](/images/ios_settings_bundle/iqiyi.plist)

2、[招商银行iPhone客户端的Settings.bundle配置](/images/ios_settings_bundle/cmb.plist)

###参考资料

1、[《整合Settings.bundle显示版本信息》](http://www.cocoachina.com/ios/20141103/10112.html)

2、[《应用程序首选项(application preference)及数据存储》](http://www.cnblogs.com/wayne23/p/3441898.html)

3、[《设置束(Setting Bundle)的使用》](http://blog.csdn.net/nogodoss/article/details/21938771)

4、[《三十而立，从零开始学ios开发（十九）：Application Settings and User Defaults（上）》](http://www.cnblogs.com/minglz/archive/2013/05/30/3048269.html)
