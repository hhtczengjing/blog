---
layout: post
title: "使用Wax给你的应用程序打补丁"
date: 2015-05-09 16:36:36 +0800
comments: true
tags: iOS
---

在iOS开发中经常遇到需要对已经上线的APP进行功能微调，或者是一些紧急的Bug修复。对于需要提交到AppStore的程序来说，每次审核的周期都会较长，在审核过程中很有可能因为各种原因被拒。由于Apple的限制，开发者无法在iOS上动态的加载Objective-C源码，使用脚本语言就可以在一定程度上解决这个问题。比如使用HTML+Javascript的方式，支付宝钱包的彩票等功能就是使用这一方式实现的。另外也可以使用Lua脚本来实现，最初我了解到的Lua是使用在游戏上面的，包括Angry Bird在内的很多知名游戏使用了Lua作为其场景设计的语言。

![lua_logo.png](/images/ios_wax_patch/lua_logo.png)

Lua是一个小巧的脚本语言。是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译，运行。

除了开发游戏，那能不能使用Lua开发iOS的应用程序呢？现在，来着国外的大牛`@probablycorey`开源了一个叫做[`wax`](http://github.com/probablycorey/wax)的项目，实现了Lua对于Objective-C的封装，可以使用Lua语言直接写Objective-C的代码了。虽然作者就宣布不再维护了，但是仍然没有降低该框架的关注度，截止到今天都有1620个star和320次Fork。

为什么要用Lua开发iOS的应用程序？作者介绍了如下几点：

- 自动垃圾回收机制。开发者无需关注alloc,retain和release这些；

- 代码少。没有那么多的头文件和数据类型（数组、字典等），以较少的代码实现更强大的功能

- 方便使用Cocoa，UITouch，Foundation等framework

- 更加容易使用的HTTP请求工具，同REST WebService交互更方便

- Lua有闭环（也可以称作blocks）

- Lua内置正则表达式的匹配的工具

###Xcode快速集成Wax

[`WaxPatch`](http://github.com/mmin18/WaxPatch)是来自大众点评的屠毅敏开源的项目，通过修改wax的实现达到动态打Patch的功能。主要是修改了wax中的`wax_instance.m`文件。具体做法是：在wax中加入了`class_replaceMethod`来替换原始实现。项目实现了在程序启动时会从指定地址下载一个包含所有Lua补丁的zip包，通过Wax加载后改变了既有Objective-C实现方法的指向函数，从而改变了程序的行为。

下面介绍如何在项目中快速集成WaxPatch：

1、下载[WaxPatch](http://github.com/mmin18/WaxPatch)和[Wax](http://github.com/probablycorey/wax)的项目到本地。

2、将源码进行整合处理

（1）将wax项目中的stdlib目录拷贝到WaxPatch的wax目录下面

（2）删除/extensions/json/目录下面的yajl-1.0.9.tar.gz和Rakefile文件

（3）不要拷贝extensions下面的SQLite和XML文件夹

3、导入到Xcode项目中，除了wax/extensions/json下面yajl的不需要设置ARC外，其他的wax相关的都需要在Build Phrase中设置`-fno-objc-arc`（前提条件是项目是ARC工程），拷贝ProtocolLoader.h文件到项目中

4、启动应用前需要在Document下创建lua目，并设置Lua的环境变量，示例代码如下:

```
- (id)init {
	if(self = [super init]) {
		NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
		NSString *dir = [doc stringByAppendingPathComponent:@"lua"];
		if(![[NSFileManager defaultManager] fileExistsAtPath:dir isDirectory:NULL] {
			[[NSFileManager defaultManager] createDirectoryAtPath:dir withIntermediateDirectories:YES attributes:nil error:NULL];
		}
		NSString *pp = [[NSString alloc ] initWithFormat:@"%@/?.lua;%@/?/init.lua;", dir, dir];
		setenv(LUA_PATH, [pp UTF8String], 1);
	}
}
```

并引入头文件

```
#import "lauxlib.h"
#import "wax.h"
```

5、下载到本地的zip包里面的(解压zip包推荐使用ZipArchive)lua文件需要放到上面的lua目录下面，并执行如下代码启动patch包：


```
wax_start("patch", luaopen_wax_http, luaopen_wax_json, nil);
```

其中：`luaopen_wax_http` (头文件是：`wax_http.h`)和 `luaopen_wax_json`(头文件是：`wax_json.h`) 表示需要使用wax扩展包，参见 `wax/extensions/`目录下面的扩展包。

6、wax不支持arm64如果需要兼容到64位的系统，请参考[《lua（wax框架） 适配 64位操作系统》](http://www.cnblogs.com/ygm900/p/3732724.html)文章进行处理。

7、运行后如果发现报`Class 'HACK_WAX_DELEGATE_IMPLEMENTOR' defined without specifying a base class`的错误，将如下代码：

```
@interface HACK_WAX_DELEGATE_IMPLEMENTOR <WaxServerDelegate> {}
@end
```

改为：

```
@interface HACK_WAX_DELEGATE_IMPLEMENTOR : NSObject<WaxServerDelegate> {}
@end
```
添加一个NSObject的基类即可

###示例程序

1、推荐阅读wax提供的示例程序。[Examples](http://github.com/probablycorey/wax/tree/master/examples/)

2、WaxPatch项目。[WaxPatch](http://github.com/mmin18/WaxPatch)

3、完整示例程序稍后会同步到GitHub上

###参考资料

1、[《如何创建更加灵活的App》](https://github.com/mmin18/Create-a-More-Flexible-App)

2、[《用Lua给你的iOS程序打patch》](http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5NTIyNTUyMQ==&appmsgid=10000028&itemidx=1&scene=4#wechat_redirect)

3、[《用Lua给你的程序打patch（续）》](http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5NTIyNTUyMQ==&appmsgid=10000031&itemidx=1&scene=4#wechat_redirect)

4、[《基于wax的lua IOS插件开发》](http://blog.csdn.net/linux_zkf/article/details/17123275)

5、[《quick tour of the Wax framework》](https://github.com/probablycorey/wax/wiki/Overview)

6、[《快速将wax配置到项目中进行lua开发》](http://www.cnblogs.com/ygm900/p/3680463.html)
