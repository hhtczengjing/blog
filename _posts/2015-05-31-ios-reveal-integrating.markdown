---
layout: post
title: "iOS开发中集成Reveal"
date: 2015-05-31 00:50:23 +0800
comments: true
tags: iOS
---

[Reveal](http://revealapp.com/) 是一个界面调试工具。使用Reveal，我们可以在iOS开发时动态地查看和修改应用程序的界面。它类似Chrome的“审查元素”功能，我们不但可以在运行时看到iOS程序的界面层级关系，还可以实时地修改程序界面，不用重新运行程序就可以看到修改之后的效果。

在使用时，我们将Reveal连接上模拟器或真机上正在运行的iOS程序，然后就可以查看和调试iOS程序的界面。

![reveal-hero.png](/images/reveal_integrating/reveal-hero.png)

###下载安装

Releal官方提供试用版本，免费试用期是30天，功能和正式版没有差别.[点此下载](http://revealapp.com/download/)。

###配置Reveal

####1、配置方式一

(1)启动Reveal，选择`Reveal -> Help -> Show Reveal Library in Finder`。

![show-reveal-library-in-finder.jpg](/images/reveal_integrating/show-reveal-library-in-finder.jpg)

(2)在Xcode中打开iOS项目,将`Reveal.framework`拖到项目中，如果升级了Reveal，对应的`Reveal.framework`文件也要更新到对应的版本。

![add-resource-to-project.jpg](/images/reveal_integrating/add-resource-to-project.jpg)

(3)选择Target -> Build Phases -> Link Binary With Libraries将Reveal.framework移除。经测试本步骤不是必须的

![remove-framework-from-project.jpg](/images/reveal_integrating/remove-framework-from-project.jpg)

(4)在Xcode的`Target -> Build Setting -> Other Linker Flags`添加如下几个配置项

` -ObjC -lz -framework Reveal`

![add-linker-flags.jpg](/images/reveal_integrating/add-linker-flags.jpg)

(5)运行项目，然后打开Reveal的界面，在左上角选择连接的设备

![reveal-app-chooser.jpg](/images/reveal_integrating/reveal-app-chooser.jpg)

然后就可以看到实际的运行效果

![reveal_demo.png](/images/reveal_integrating/reveal_demo.png)

####2、配置方式二

Reveal官方介绍了好几种办法使Reveal连接模拟器，都需要修改工程文件。但如果修改了工程文件，就需要参与项目开发的所有人都装有Reveal，下面介绍一种比较方便的方式来集成Reveal，步骤如下：

首先打开Terminal，输入`vim ~/.lldbinit`创建一个名为.lldbinit的文件，然后将如下内容输入到该文件中：

```
command alias reveal_load_sim expr (void*)dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib", 0x2);
command alias reveal_load_dev expr (void*)dlopen([(NSString*)[(NSBundle*)[NSBundle mainBundle]               pathForResource:@"libReveal" ofType:@"dylib"] cStringUsingEncoding:0    x4], 0x2);
command alias reveal_start expr (void)[(NSNotificationCenter*)[NSNotificationCenter defaultCenter]           postNotificationName:@"IBARevealRequestStart" object:nil];
command alias reveal_stop expr (void)[(NSNotificationCenter*)[NSNotificationCenter defaultCenter]            postNotificationName:@"IBARevealRequestStop" object:nil];
```

该步骤其实是为lldb设置了4个别名，为了后续方便操作，这4个别名意义如下：

`reveal_load_sim` 为模拟器加载reveal调试用的动态链接库

`reveal_load_dev` 为真机加载reveal调试用的动态链接库

`reveal_start` 启动reveal调试功能

`reveal_stop`  结束reveal调试功能

(1)Reveal连接模拟器

![reveal_load_sim.png](/images/reveal_integrating/reveal_load_sim.png)

在AppDelegate类的`application:didFinishLaunchingWithOptions:`方法中，作如下3步操作（如下图所示）：

1)点击该方法左边的行号区域，增加一个断点，之后右击该断点，选择“Edit Breakpoint”。

2)点击”Action”项边右的”Add Action”,然后输入“reveal_load_sim”

3)勾选上Options上的”Automatically continue after evaluating”选项。

(2)Reveal连接真机

要用Reveal连接真机调试，我们需要先把Reveal的动态链接库上传到真机上。由于iOS设备有沙盒存在，所以我们只能将Reveal的动态链接库添加到工程中。

1)点击Reveal菜单栏的”Help”->”Show Reveal Library in Finder”选项，可以在Finder中显示出Reveal的动态链接库：`libReveal.dylib`

![show-reveal-library-in-finder.jpg](/images/reveal_integrating/show-reveal-library-in-finder.jpg)

2)调整`libReveal.dylib`的引用方式，这里我们只需要将`libReveal.dylib`文件拷贝到Sandbox中，但是我们在引入`libReveal.dylib`的时候Xcode默认是以`Link Binary With Libraries`的方式的，实际上应该是`Copy Bundle Resources`,所以应该先将`libReveal.dylib`从`Link Binary With Libraries`中移除掉，然后在`Copy Bundle Resources`中添加。

3)安装之前处理模拟器的方式，将配置文件改成`reveal_load_dev`.

![reveal_load_dev.png](/images/reveal_integrating/reveal_load_dev.png)

启动后在控制台会出现如下内容：

![reveal_load_console.png](/images/reveal_integrating/reveal_load_console.png)


###参考资料

1、[《使用Reveal来查看、修改、调试iOS应用》](http://wufawei.com/2013/12/use-reveal-to-inspect-ios-apps/)

2、[《Integrating Reveal without modifying your Xcode project》](http://blog.ittybittyapps.com/blog/2013/11/07/integrating-reveal-without-modifying-your-xcode-project/)

3、[《Reveal查看任意app的高级技巧》](http://c.blog.sina.com.cn/profile.php?blogid=cb8a22ea89000gtw)