---
layout: post
title: "使用 InjectionIII 提高开发效率"
date: 2023-11-11 14:09:57 +0800
comments: true
tags: iOS
---

iOS 原生代码的修改编译调试，都是一遍遍的 `Command + R` 重新编译重启 App 来进行的。一般来说，随着项目的复杂性增加，代码量越大，编译的耗时就越久。大多数项目都采用二进制集成的方式（将部分代码/组件库先编译成二进制的库集成到工程里面），来避免每次都全量编译来提升编译的速度，但即使这样也没有解决每次修改代码（比如对UI的进行还原度调整）还是需要重新编译的情况。

一直很羡慕前端开发或者是Flutter开发的同学，修改完代码就能直接看到效果。官方的效果动画如下：

![flutter_hot_reload](/images/ios_injectioniii/flutter_hot_reload.gif)

如果对于 iOS 原生开发来说要如何实现 Flutter 的这种极速调试开发体验呢？由国外的一位大神[John Holdsworth](https://johnholdsworth.com/) 开发的 [InjectionIII](https://github.com/johnno1962/InjectionIII.git) 这个工具，可以动态地将 Swift 或 Objective-C 的代码在已运行的程序中执行，以加快调试速度，同时保证程序不用重启。官方的效果动画如下：

![ios_hot_reload](/images/ios_injectioniii/ios_hot_reload.gif)

之前只是简单试用了一下对于组件化开发的项目来说确实没有太好的效果，因为我们的项目源码基本上都分散在多个仓库里面。近期看了一下源码，结合工具的一些特性实现了在我们的项目里面集成，并推广应用起来，从大家的反馈来看确实能提高研发的效率，基本上能做到修改完代码马上能看到效果。

### 如何使用

#### 1、安装 InjectionIII.app

可以到[AppStore](https://apps.apple.com/cn/app/injectioniii/id1380446739) 和 [Github Release](https://github.com/johnno1962/InjectionIII/releases/download/4.7.5/InjectionIII.app.zip)下载，推荐到Github Release页面下载一般更新比较及时 ，当前的最新版本是 4.7.5。

![download_app](/images/ios_injectioniii/download_app.png)

#### 2、设置项目目录

安装完成后，打开 InjectionIII.app ，在状态栏点图标在弹框页面中选择 `Open Project`，设置当前正在开发的项目目录（xcodeproj文件所在的目录）。

![open_project_settings](/images/ios_injectioniii/open_project_settings.png)

设置的项目会在 `Open Recent` 中展示，同时需要保持 `File Watcher` 的选项选中。

![file_watcher_settings](/images/ios_injectioniii/file_watcher_settings.png)

> 说明：如果需要切换其他项目，可以 `Clean Menu` 清除当前选中的项目，然后重新设置。

#### 3、项目配置

#####（1）AppDelegate 的 didFinishLaunchingWithOptions 方法中添加注入代码

以 OC版本 为例，Swift的可以参考官方的文档，基本上也是一样的做法：

``` 
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
#if DEBUG
   [[NSBundle bundleWithPath:@"/Applications/InjectionIII.app/Contents/Resources/iOSInjection.bundle"] load];
#endif

   ...
}
```

运行项目，控制台出现 `Injection connected, watching xxxx` 表示配置成功。

> 说明：同时还支持 tvOS（tvOSInjection.bundle） 和 MacOS（macOSInjection.bundle）版本，只要在对应的路径进行替换即可

##### (2) 监听变化并刷新

1) 添加 `INJECTION_BUNDLE_NOTIFICATION` 通知

```
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

- (instancetype)init {
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(injectedAction:) name:@"INJECTION_BUNDLE_NOTIFICATION" object:nil];
    }
    
    return self;
}

- (void)injectedAction:(id)sender {
    // TODO: 刷新UI
}
```

2）添加 injected 方法

```
- (void)injected {
    // TODO: 刷新UI
}
```

### 项目整合

如果利用 CocoaPods 组件化的开发模式，可能存在通过 CocoaPods 引入一些外部的代码到项目里面来进行开发。找到了一个[issue](https://github.com/johnno1962/InjectionIII/issues/121)，作者做了回复就是推荐将源码放到工程目录下面，这样就可以实现自动监听。

![cocoapods_sources_settings](/images/ios_injectioniii/cocoapods_sources_settings.png)

这就是我前面提到的之前试了确实对我们的项目不太友好的原因，我们是通过可视化工具进行项目的一些管理的，代码的结构调整起来会比较繁琐。

知道需要通过设置 `Open Project` 来绑定一个项目的工作空间目录，InjectionIII 会自动监听文件的变化，刚开始想的就是能不能改下代码，自动去解析项目额外依赖的源码目录，然后加入到监听的列表中呢? 下载源码到本地看下：

```
git clone https://github.com/johnno1962/InjectionIII --recurse-submodules
```

首先看下 `Open Project` 具体干了什么？通过翻阅代码发现其实主要是拉起来一个文件目录选择对话框，获取选中的目录赋值给一个全局的变量 `selectedProject`。

```
let open = NSOpenPanel()
open.prompt = "Select Project File"
open.directoryURL = url
open.canChooseDirectories = false
open.canChooseFiles = true
// open.showsHiddenFiles = TRUE;
if open.runModal() == .OK,
	let url = open.url {
	selectedProject = url.path
}
```

然后下面就出现一段将 `selectedProject` 加入到 `watchedDirectories` 的代码：

```
watchedDirectories.removeAll()
watchedDirectories.insert(url.path)
if let alsoWatch = defaults.string(forKey: "addDirectory"),
	let resolved = resolve(path: alsoWatch) {
	watchedDirectories.insert(resolved.path)
}
```

`watchedDirectories` 的定义 `var watchedDirectories = Set<String>()` 从变量名就可以看出来是监听的文件夹列表，把项目目录加进去很好理解，但是 `addDirectory` 这个又是啥？经过追踪代码，发现新版本添加了一个 `Add Directory` 的设置项用于设置一个额外的目录用于监听代码的变动。

```
@IBAction func addDirectory(_ sender: Any) {
	let open = NSOpenPanel()
	open.prompt = openProject
	open.allowsMultipleSelection = true
	open.canChooseDirectories = true
	open.canChooseFiles = false
	if open.runModal() == .OK {
		for url in open.urls {
			appDelegate.watchedDirectories.insert(url.path)
			self.lastConnection?.watchDirectory(url.path)
			persist(url: url)
		}
	}
}
```

对照的功能入口就是：

![add_directory_settings](/images/ios_injectioniii/add_directory_settings.png)

这样我们就可以把那些外部引用的代码都放在一个固定的目录下面（其实我们现在也是这样做的）然后在 `Add Directory` 里面设置这个源码根目录就好了。当然前面说的可以通过自动解析我们的配置文件然后实现自动加入监听也是可以实现的。

前面提到了如果要实现动态刷新UI需要添加监听代码，这个确实存在对项目有一定的侵入性，如何解决呢？

1、自动注入 `iOSInjection.bundle`

可以通过在 `+load` 方法中监听 `UIApplicationDidFinishLaunchingNotification` 通知的方法搞定

2、UIViewController 实现自动刷新

可以通过 hook UIViewController 的 init 方法动态添加通知监听实现。

整合了一些代码，写了一个简单的Demo：

```
pod 'InjectionIIISupport', :git => 'https://github.com/hhtczengjing/InjectionIIISupport.git', :configuration => ['Debug']
```

### 问题排查与解决

> 说明：xib/storyboard 目前测试有点问题，后续再补充

1、控制台出现 `Loaded .dylib - Ignore any duplicate class warning ^` 的日志

> 解决办法：无需关注，不影响使用

2、添加了 `injected` 方法不生效或者是直接Crash

> 解决办法：监听 `INJECTION_BUNDLE_NOTIFICATION` 通知实现

3、控制台出现 `Your project file seems to be in the Desktop or Documents folder and may prevent InjectionIII working as it has special permissions.`

> 解决办法：无需关注，不影响使用

4、如果修改的 UITableViewCell 的内容，保存代码后不生效

> 解决办法：重新打开页面，或者是上下滚动页面

### 参考资料

- 1、[InjectionIII](https://github.com/johnno1962/InjectionIII)

- 2、[App 如何通过注入动态库的方式实现极速编译调试？](https://time.geekbang.org/column/article/87188)