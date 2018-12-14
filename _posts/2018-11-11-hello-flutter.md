---
layout: post
title: "Flutter开发初探"
date: 2018-11-11 18:46:37 +0800
comments: true
tags: iOS
---

[Flutter](https://flutter.io/)是由谷歌创建的一个框架，用于构建“现代移动应用”。目前它还处于beta阶段，不过它的文档和相关工具十分齐全，有些移动应用已经在使用Flutter。

### 开发环境搭建

由于在国内访问Flutter有时可能会受到限制，可以通过如下的配置使用国内的镜像(可以在一定程度上加快下载的速度)：

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

1、下载flutter

```
# 下载
git clone https://github.com/flutter/flutter.git

# 切换到最新的tag(当前最新的是v0.11.13)
git checkout v0.11.13
```

2、设置环境变量

```
# 设置环境变量
export PATH=`pwd`/flutter/bin:$PATH
```

3、安装依赖

```
# 安装依赖
flutter doctor
```

如果是第一次使用可能有如下提示：

```
[!] Android toolchain - develop for Android devices (Android SDK 28.0.1)
    ✗ Android licenses not accepted.  To resolve this, run: flutter doctor --android-licenses
[!] iOS toolchain - develop for iOS devices (Xcode 9.4.1)
    ✗ libimobiledevice and ideviceinstaller are not installed. To install, run:
        brew install --HEAD libimobiledevice
        brew install ideviceinstaller
    ✗ ios-deploy not installed. To install:
        brew install ios-deploy
[✓] Android Studio (version 3.1)
    ✗ Flutter plugin not installed; this adds Flutter specific functionality.
    ✗ Dart plugin not installed; this adds Dart specific functionality.
[!] IntelliJ IDEA Ultimate Edition (version 2017.3.3)
    ✗ Flutter plugin not installed; this adds Flutter specific functionality.
    ✗ Dart plugin not installed; this adds Dart specific functionality.
[!] Connected devices
    ! No devices available

! Doctor found issues in 4 categories.
```

按照上面的说明进行处理即可：

(1) 接受Android开发者协议

```
flutter doctor --android-licenses
```

(2) 安装一些必要的iOS的工具

```
brew install --HEAD libimobiledevice
brew install ideviceinstaller
brew install ios-deploy
```

2、配置IDE

(1) 配置IDEA/Android Studio

`Preferences -> Plugins`搜索`Flutter`并安装插件

也可以到下面的地址去下载

```
http://plugins.jetbrains.com/
```

(2) 配置VS Code

`View -> Command Palette`输入`install`, 然后选择`Extensions: Install Extension action`, 在搜索框中输入`flutter`在搜索结果列表里面选择`Flutter`点击安装(install）即可，安装完成后选择OK重新启动VS Code。

3、初始化创建项目

命令行输入：`flutter create hello_flutter`，即可快速创建一个Flutter的项目, 创建完成后就可以通过如下方式快速运行：

```
# 进入到工程目录
cd hello_flutter
# 打开iPhone模拟器
open -a Simulator
# 运行
flutter run
```

### 如何编写Flutter插件

1、创建模板工程

命令行执行如下命令：

```
flutter create --template=plugin flutter_hybrid_router
```

2、实现Dart调用的代码

在`lib/flutter_hybrid_router.dart`中实现Flutter端调用的方法逻辑：

```
Future<void> openURL({String url, List parameters}) async {
    _channel.invokeMethod('openURL', {"url": url ?? "", "parameters": (parameters ?? [])});
}
```

3、实现iOS端的实现逻辑

实现`handleMethodCall:result:`方法, Flutter端dart代码调用Native的逻辑：

```
- (void)handleMethodCall:(FlutterMethodCall *)call result:(FlutterResult)block {
    if ([@"openURL" isEqualToString:call.method]) {
        id arguments = call.arguments;
        if(arguments && [arguments isKindOfClass:[NSDictionary class]]) {
            NSString *pattern = [arguments objectForKey:@"url"];
            NSArray *parameters = [arguments objectForKey:@"parameters"];
            if(!pattern || pattern.length <= 0) {
                !block ? : block(nil);
                return;
            }
            NSString *url = pattern;
            if(parameters && [parameters isKindOfClass:[NSArray class]] && parameters.count > 0) {
                url = [MGJRouter generateURLWithPattern:pattern parameters:parameters];
            }
            if(!url || url.length <= 0) {
                !block ? : block(nil);
                return;
            }
            [MGJRouter openURL:url completion:^(id result) {
                !block ? : block(result);
            }];
        }
        else {
            !block ? : block(nil);
        }
    }
    else {
        !block ? : block(FlutterMethodNotImplemented);
    }
}
```

### 集成到现有的项目中

1、修改Podfile

修改ios目录下面的Podfile文件，在顶部新增两行配置

```
platform :ios, '8.0'
use_frameworks!
```

2、执行编译脚本

```
echo "===清理flutter历史编译==="
flutter clean

echo "===重新生成plugin索引==="
flutter packages get

echo "===生成App.framework和flutter_assets==="
flutter build ios --release

echo "===生成各个plugin的二进制库文件==="
cd ios/Pods
for plugin_name in "flutter_hybrid_router" "flutter_webview_plugin"
do
    echo "生成${plugin_name}.framework..."
    /usr/bin/env xcrun xcodebuild build -configuration Release ARCHS='arm64 armv7' -target ${plugin_name} BUILD_DIR=../../build/ios -sdk iphoneos -quiet
    /usr/bin/env xcrun xcodebuild build -configuration Debug ARCHS='x86_64' -target ${plugin_name} BUILD_DIR=../../build/ios -sdk iphonesimulator -quiet
    echo "合并${plugin_name}.framework..."
    lipo -create "../../build/ios/Debug-iphonesimulator/${plugin_name}/${plugin_name}.framework/${plugin_name}" "../../build/ios/Release-iphoneos/${plugin_name}/${plugin_name}.framework/${plugin_name}" -o "../../build/ios/Release-iphoneos/${plugin_name}/${plugin_name}.framework/${plugin_name}"
done
```

脚本执行完成之后，创建一个Flutter的文件夹，将生成的产物拷贝过来，文件结构如下所示：

```
.
├── FlutterModule.podspec
└── Pod
    ├── Assets
    │   └── flutter_assets
    └── Framework
        ├── App.framework
        ├── Flutter.framework
        ├── flutter_hybrid_router.framework
        └── flutter_webview_plugin.framework
```

其中`FlutterModule.podspec`文件的内容如下：

```
Pod::Spec.new do |s|
  s.name         = "Flutter"
  s.version      = "0.1.0"
  s.summary      = "Flutter Demo"
  s.description  = <<-DESC
    Flutter Demo                
  DESC
  s.homepage     = "https://blog.devzeng.com"
  s.license      = "MIT"
  s.platform = :ios, '8.0'
  s.author       = { "zengjing" => "hhtczengjing@gmail.com" }
  s.source       = { :git => "", :tag => "#{s.version}" }
  s.resources = "Pod/Assets/*"
  s.vendored_frameworks = "Pod/Framework/**/*.framework"
  s.dependency 'MGJRouter', '~> 0.10.0'
end
```

然后就可以直接在代码中引用了。

3、主工程中注册插件

(1) `AppDelegate`中实现`FlutterPluginRegistry`协议

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 注册插件
    [FlutterHybridRouterPlugin registerWithRegistrar:[self registrarForPlugin:@"FlutterHybridRouterPlugin"]];
    
    // TODO: 给每个插件发送消息
 
    return YES;
}
```

(2) 打开Flutter页面

```
FlutterViewController *controller = [[FlutterViewController alloc] init];
[self.navigationController pushViewController:controller animated:YES];
```

(3) 实现OpenURL的协议

```
[MGJRouter registerURLPattern:@"com.devzeng.demo://goback" toHandler:^(NSDictionary *routerParameters) {
   UIViewController *controller = nil; // 获取当前Flutter页面
   [controller.navigationController popViewControllerAnimated:YES];
}];
```

最后，本文的demo可以在下面找到：

(1) Flutter插件示例：[`flutter_hybrid_router`](https://github.com/hhtczengjing/flutter_hybrid_router_plugin.git)

(2) Flutter示例项目：[`flutter_demo`](https://github.com/hhtczengjing/flutter_demo.git)

(3) Flutter编译后的模块：[`flutter_module_example`](https://github.com/hhtczengjing/flutter_module_example.git)

(4) Flutter集成到现有项目中的示例：[`flutter_ios_project_integrate`](https://github.com/hhtczengjing/flutter_ios_project_integrate.git)

### 参考资料

1、[Flutter官网](https://flutter.io/)

2、[为什么说Flutter是革命性的？](http://www.infoq.com/cn/articles/why-is-flutter-revolutionary)

3、[认识Dart语言](https://www.stephenw.cc/2018/04/20/dart-getting-start/)

4、[Dart的变量和类型](https://www.stephenw.cc/2018/04/21/dart-var-types/)

4、[Dart的函数](https://www.stephenw.cc/2018/04/23/dart-func/)

5、[Dart的类](https://www.stephenw.cc/2018/04/25/dart-class/)

6、[Dart中的泛型](https://www.stephenw.cc/2018/04/27/dart-generics/)


