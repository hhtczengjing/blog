---
layout: post
title: "iOS开发之Architectures设置"
date: 2014-08-28 22:37:46 +0800
comments: true
tags: iOS
---

在iOS开发中经常遇到的一个错误是`Undefined symbols for architecture arm64`，这个错误表示工程某些地方不支持`arm64指令集`。本文围绕在iOS开发中经常遇到的关于Architectures方面的设置介绍iOS的指令集方面的知识点。

对于iOS设备来说iOS的指令集有`armv6、armv7、armv7s、arm64`这样四种，不同型号的iOS设备使用不同的指令集，下面是各自的区别：

- `armv6`
	- iPhone、iPhone 3G
	- iPod 1G、iPod 2G
- `armv7`
	- iPhone 3GS、iPhone 4
	- iPod 3G、iPod 4G、iPod 5G
	- iPad、iPad 2、iPad 3、iPad Mini
- `armv7s`
	- iPhone 5、iPhone 5C
	- iPad 4 
- `arm64`
	- iPhone 5S
	- iPad Air, Retina iPad Mini


在Xcode的`target->Build Settings`中有一个Architectures的分组主要是用来设置Architectures方面的内容，下面重点介绍下面几个设置项的内容。

![ios-architectures-config.png](/images/ios-architectures/ios-architectures-config.png)

##Architectures

该编译选项指定了工程将被编译成支持哪些指令集，支持指令集是通过编译生成对应的二进制数据包实现的，如果支持的指令集数目有多个，就会编译出包含多个指令集代码的数据包，造成最终编译的包很大。

官方文档说明：

Space-separated list of identifiers. Specifies the architectures (ABIs, processor models) to which the binary is targeted. When this build setting specifies more than one architecture, the generated binary may contain object code for each of the specified architectures.


##Build Active Architectures Only
该编译项用于设置是否只编译当前使用的设备对应的arm指令集。

当该选项设置成YES时，你连上一个armv7指令集的设备，就算你的Valid Architectures和Architectures都设置成armv7/armv7s/arm64，还是依然只会生成一个armv7指令集的二进制包。

当然该选项起作用的前提是你的Xcode必须成功连接了调试设备。如果你没有任何活跃设备，即Xcode没有成功连接调试设备，就算该设置项设置成YES依然还会编译Valid Architectures和Architectures指定的二进制包。

通常情况下，该编译选项在Debug模式都设成YES，Release模式都设成NO。

官方文档说明：

Boolean value. Specifies whether the product includes only object code for the native architecture.

##Valid Architectures

该编译项指定可能支持的指令集，该列表和Architectures列表的交集，将是Xcode最终生成二进制包所支持的指令集。

比如将Valid Architectures设置支持的arm指令集版本有：armv7、armv7s、arm64，对应的Architectures设置的支持arm指令集版本有：armv7s，这时Xcode只会生成一个armv7s指令集的二进制包。

官方文档说明：

Space-separated list of identifiers. Specifies the architectures for which the binary may be built. During the build, this list is intersected with the value of ARCHS build setting; the resulting list specifies the architectures the binary can run on. If the resulting architecture list is empty, the target generates no binary.

##说明

1、指令集是向下兼容的。比如，armv7s指令集的设备，可以兼容运行使用armv7、armv6编译的程序。
	
2、不同的Xcode支持的指令集不同，比如Xcode5.1.1支持armv7、armv7s、arm64三个指令集。自Xcode4.5起不再支持armv6。
  
3、如果想自己的app在各个机器都能够最高效率的运行，则需要将`Build Active Architecture Only`设置为`NO`，`Valid architectures`选择对应的指令集`armv7 armv7s arm64`。这个会为各个指令集编译对应的代码，因此最后生成的ipa体积基本翻了3倍。

4、如果想让app体积保持最小，则现阶段应该选择`Valid architectures`为`armv7`，`Build Active Architecture Only`设置YES/NO都无关紧要。

5、如果遇到`Undefined symbols for architecture arm64`这样的错误，一般是表示引入的第三方库不支持arm64的指令集，解决这个问题就只需要删除Architectures中的arm64的选项。
  
##参考资料

1、[《Xcode设置项之Architectures和Valid Architectures》](http://wangzz.github.io/blog/2014/05/09/xcodeshe-zhi-xiang-zhi-architectureshe-valid-architectures/)

2、[《xcode5 arm64》](http://justsee.iteye.com/blog/2009954)

3、[《64-Bit Transition Guide for Cocoa Touch》](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaTouch64BitGuide/Introduction/Introduction.html)
