---
layout: post
title: "iOS开发中的Search Paths设置"
date: 2014-09-14 18:34:42 +0800
comments: true
tags: iOS
---

在iOS开发中经常遇到一些关于路径的设置，比如引入了百度地图的SDK，项目拷贝到其他的电脑上或者多人同时开发的时候容易报`Library Not Found`的错误，或者是引入第三方库比如`ASIHttpRequest`/`RETableView`经常报`#include <>`的错误这就需要配置一些搜索路径。

![ios-search-paths.png](/images/ios-search-paths/ios_search_paths.png)

##Framework/Library Search Paths

1、Framework Search Paths

附加到项目中的framework(`.framework` bundles)的搜索路径,在iOS开发中使用的不是特别多，通常对于iOS的开发来说一般使用系统内置的framework。

2、Library Search Paths

附加到项目中的第三方Library(`.a files`)的搜索路径，Xcode会自动设置拖拽到Xcode中的.a文件的路径，为了便于移植或者是多人协作开发一般会手动设置。

比如对于设置百度的地图的SDK，我们会设置如下：

`$(SRCROOT)/../libs/Release$(EFFECTIVE_PLATFORM_NAME)`，其中
`$(SRCROOT)`宏代表您的工程文件目录，`$(EFFECTIVE_PLATFORM_NAME)`宏代表当前配置是OS还是simulator

##Header Search Path

1、C/C++头文件引用

在C/C++中，include是变异指令，在编译时，编译器会将相对路径替换成绝对路径，因此，头文件的绝对路径等同于`搜索路径+相对路径`。

(1)`#include <iostream.h>`：引用编译器的类库路径下的头文件

(2)`#include "hello.h"`：引用工程目录的相对路径的头文件

2、(User) Header Search Path

（1）`Header Search Path`指的是头文件的搜索路径。

（2）`User Header Search Paths`指的是用户自定义的头文件的搜索路径

3、Always Search User Paths

如果设置了`Always Search User Paths`为`YES`,编译器会优先搜索`User Header Search Paths`配置的路径，在这种情况下`#include <string.h>`,`User Header Search Paths`搜索目录下面的文件会覆盖系统的头文件。

##常见配置

1、`libxml/tree.h not found`的错误配置

在build setting中的`header search path`中加入`${SDK_DIR}/usr/include/libxml2`

##参考资料

1、[《iOS: Clarify different Search Paths》](http://stackoverflow.com/questions/8342982/ios-clarify-different-search-paths)

2、[《xcode4的环境变量，Build Settings参数，workspace及联编设置》](http://www.cnblogs.com/xiaodao/archive/2012/03/28/2422091.html)