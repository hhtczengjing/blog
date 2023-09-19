---
layout: post
title: "搭建Flutter引擎源码调试环境"
date: 2023-09-19 23:50:26 +0800
comments: true
tags: iOS
---

最近在排查问题的时候总会遇到一些和 Flutter 引擎相关的问题，需要直接能在Xcode里面挂在引擎的源码能进行断点Debug，这里记录一下搭建Flutter引擎源码调试环境过程：

### 环境准备

##### 1、开发工具

（1）下载depot_tools工具包

depot_tools 是 chromium 使用的源码库管理工具，可以方便的管理源码以及对应依赖，通过gclinet可以获取所有的编译需要的源码和依赖

```
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=$PATH:/Users/zengjing/flutter_source/depot_tools
```

（2）安装Ninja

编译工具，负责对项目进行编译生成对应的产物

```
brew install ninja
```

##### 2、下载源码

（1）创建engine目录，在目录下创建.gclient配置文件，文件内容如下：

```
solutions = [
  {
    "managed": False,
    "name": "src/flutter",
    "url": "https://github.com/hhtczengjing/engine.git",
    "custom_deps": {},
    "deps_file": "DEPS",
    "safesync_url": "",
  },
]
```

目录结构如下：

```
flutter_source
├── depot_tools
├── engine
│   ├── .gclient
```

（2）下载Flutter Engine源码

进入engine目录，执行如下命令获取Flutter所有依赖（耗时比较长，可能需要一个多小时）：

```
cd engine
gclient sync
```

（3）同步Flutter Engine代码

```
cd engine/src/flutter
git remote add upstream https://github.com/flutter/engine.git
git pull upstream master
```

##### 3、下载指定的Engine版本源码

（1）获取指定的Engine的版本号

```
cat $FLUTTER_SDK_PATH/bin/internal/engine.version
```

> 2dce47073a378673f6ca095e91b8065544c3a881

（2）切换源码到指定的Commit

```
cd engine/src/flutter
git reset --hard ${engine.version}
```

> git reset --hard 2dce47073a378673f6ca095e91b8065544c3a881

（3）下载相关的依赖

```
cd engine/src/flutter
gclient sync --with_branch_heads --with_tags
```

### 生成工程

```
cd engine/src

./flutter/tools/gn --unoptimized # host_debug_unopt
./flutter/tools/gn --runtime-mode=debug --ios --simulator --unoptimized # 模拟器Debug
./flutter/tools/gn --runtime-mode=debug --ios --ios-cpu=arm64 # 设备Debug
./flutter/tools/gn --runtime-mode=release --ios --ios-cpu=arm64 # 设备Release
```

参数说明：

- （1）`--unoptimized`: 是否优化性能，默认优化
- （2）`--runtime-mode`: 可选值debug/profile/release/jit_release
- （1）`--target-os`:  指定编译产物的平台，可选值android/ios/linux/fuchsia/win/winuwp
- （3）`--ios`: iOS设备
- （3）`--ios-cpu`: 可选值arm/arm64
- （4）`--simulator`: iOS模拟器
- （4）`--simulator-cpu`: 可选值x64/arm64
- （4）`--android`: Android
- （4）`--android-cpu`: 可选值arm/x64/x86/arm64

### 编译

```
cd engine/src

ninja -C out/host_debug_unopt
ninja -C out/ios_debug_sim_unopt
ninja -C out/ios_debug
ninja -C out/ios_release
```

### 调试

1、生成测试的工程

```
flutter create -i objc -a java flutter_demo
flutter run --local-engine-src-path=/Users/zengjing/flutter_source/engine/src --local-engine=ios_debug_sim_unopt --verbose
```

2、把 `ios_debug_sim_unopt` 里面的 `flutter_engine.xcodeproj` 拖进需要调试的Demo工程目录

3、在Genrated.xcconfig中加上内容为

```
FLUTTER_ENGINE=/Users/zengjing/flutter_source/engine/src
LOCAL_ENGINE=ios_debug_sim_unopt
```

### 参考资料

1、[Compiling the engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)

2、[怎样的Flutter Engine定制流程，才能实现真正“开箱即用”？](https://www.yuque.com/xytech/flutter/osg73p)