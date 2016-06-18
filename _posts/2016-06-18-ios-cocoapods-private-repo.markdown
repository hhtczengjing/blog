---
layout: post
title: "在iOS项目中使用CocoaPods私有库"
date: 2016-06-18 22:36
comments: true
tags: iOS
---

`CocoaPods`的出现极大的减轻了我们日常开发的工作量，特别是在做一些繁琐的配置上面，正如CocoaPods官网上面的`Get on with building your app, not duplicating code.`这句话一样CocoaPods让我们把精力放在打磨我们的产品，而不是把时间浪费在做一些重复的事情上面。

在开发项目的过程中引入第三方代码库会涉及到许多内容。有的时候需要配置`build phases`和`linker flags`，这样的细节配置会引起许多人为因素的错误导致整个项目无法正常编译。`CocoaPods`的出现简化了这些，它能够自动配置编译选项。另外，通过`CocoaPods`的搜索功能，可以很方便的查找和使用第三方库文件。

一直想把公司的项目中常用到的一些代码提取出来做成公用的库(同一套代码每个项目中都基本上会用到，而且每次改动之后所有的项目都需要进行同步修改，特别繁琐)，恰好最近在调整新项目的代码，趁着这个机会就整理了一下代码。因为有些代码是公司级别的所以不能直接使用CocoaPods的Spec，需要把代码托管在公司的内部服务器上面，使用CocoaPod私有的Spec。之前写过一篇笔记[《使用CocoaPods管理iOS项目中的依赖库》](http://.devzeng.com/blog/ios-cocoapods-dependency-manager.html)，介绍如何安装CocoaPods如何在项目中使用。本文主要是介绍如何搭建和使用私有库。

###创建私有的Spec仓库

1.在Git上创建Spec仓库

Spec是Pods索引库，所有公开的Pods都在这个里面，他实际是一个Git仓库remote端
在GitHub上，但是当你使用了Cocoapods后他会被clone到本地的`~/.cocoapods/repos`目录下，可以进入到这个目录看到master文件夹就是这个官方的Spec Repo了。这个master目录的结构是这个样子的:

```
├── Specs
    └── [SPEC_NAME]
        └── [VERSION]
            └── [SPEC_NAME].podspec            
```

每次我们在执行`pod install`或者是`pod update`的时候都会自动更新这个spec里面的内容，拉取最新的数据回来。然后我们使用`pod search`的时候就是直接从这个里面进行查询的。

2.向CocoaPods添加该仓库

`pod repo add 仓库名 仓库地址`

示例如下:

`pod repo add zengjing-spec git@gitlab.com:zengjing/Specs.git`

添加成功之后可以通过`pod repo list`的命令查看，结果如下所示：

![repo-list](/images/ios-cocoapods-private-repo/repo-list.png)

###编写podspec文件

####1.搭建项目的结构，创建podspec文件

`podspec`是一个配置文件，该文件描述了一个库是怎样被添加到工程中的。在这个配置文件里面配置了：库文件的名称、源码存放路径、版本号、依赖库和编译条件等。为了避免写错，建议使用命令行的方式生成。

在终端执行`pod spec create Foundation-pd`，CocoaPods会自动帮我们生成`Foundation-pd.podspec`的模板文件。推荐搭建项目的文件组织结构如下所示：

```
Foundation-pd
  ├── Example(示例项目)
  ├── LICENSE(LICENSE文件)
  ├── Foundation-pd.podspec
  ├── Pod
  │   ├── Assets(资源文件)
  │   └── Classes(代码文件)
  └── README.md(一些说明信息)
```

####2.提交代码到git服务器

(1)提交代码到远程仓库

```
git add .
git commit -s -m "first commit"
git push origin master
```

(2)给当前的版本加上tag，提交到远程仓库

```
git tag -a 0.1.0 -m 'v0.1.0' HEAD
git push origin tag 0.1.0
```

####3.编写podspec文件

示例podspec内容如下，建议参考CocoaPods上面的来做，照着那些第三库的podspec文件写就行了：

```
Pod::Spec.new do |s|
  s.name              = "Foundation-pd"
  s.version           = "0.1.0"
  s.summary           = "Powerdata Foundation for iOS Team"
  s.homepage          = "http://blog.devzeng.com"
  s.license			   = { :type => 'MIT', :file => 'LICENSE' }
  s.author            = { "zengjing" => "hhtczengjing@gmail.com" }
  s.source            = { :git => "YOUR_REPO_URL", :tag => "#{s.version}" }
  s.default_subspec   = 'Core'
  s.platform          = :ios, "7.0"
  s.requires_arc      = true
  s.framework         = 'UIKit'

  s.subspec 'Core' do |ss|
    ss.source_files = "Pod/Classes/{Core,Util}/**/*.{h,m,mm,c}"
  end

  s.subspec 'CrashReporter' do |ss|
    ss.source_files = "Pod/Classes/CrashReporter/**/*.{h,m,mm,c}"
    ss.dependency 'PLCrashReporter', '~> 1.2.0'
  end

  s.subspec 'DB' do |ss|
    ss.source_files = "Pod/Classes/DB/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'FMDB', '~> 2.6.2'
    ss.dependency 'FMDBMigrationManager', '~> 1.4.1'
  end

  s.subspec 'Logger' do |ss|
    ss.source_files = "Pod/Classes/PDFoundation/src/Logger/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'CocoaLumberjack', '~> 2.2.0'
  end

  s.subspec 'UI' do |ss|
    ss.source_files = "Pod/Classes/UI/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'Masonry', '~> 1.0.0'
  end

  s.subspec 'Model' do |ss|
    ss.source_files = "Pod/Classes/Model/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'Mantle', '~> 2.0.7'
    ss.dependency 'RPJSONValidator', '~> 0.2.0'
    ss.dependency 'libextobjc', '~> 0.4.1'
  end

  s.subspec 'Store' do |ss|
    ss.source_files = "Pod/Classes/Store/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'Foundation-pd/Network'
  end

  s.subspec 'Service' do |ss|
    service.source_files = "Pod/Classes/Service/**/*.{h,m,mm,c}"
    service.dependency 'Foundation-pd/Core'
  end
  
  s.subspec 'Network' do |ss|
    ss.source_files = "Pod/Classes/Network/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'AFNetworking', '~> 2.5.4'
  end

  s.subspec 'WebService' do |ss|
    ss.source_files = "Pod/Classes/WebService/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
    ss.dependency 'AFNetworking', '~> 2.5.4'
    ss.dependency 'KissXML', '~> 5.0.3'
    ss.xcconfig = { 'HEADER_SEARCH_PATHS' => '$(SDKROOT)/usr/include/libxml2' }
  end

  s.subspec 'Manager' do |ss|
    ss.source_files = "Pod/Classes/Manager/**/*.{h,m,mm,c}"
    ss.dependency 'Foundation-pd/Core'
  end

end
```

语法说明请参考：[《Podspec Syntax Reference》](https://guides.cocoapods.org/syntax/podspec.html)

###提交podspec文件到Spec仓库

####1.检查podspec文件是否正确

`pod spec lint Foundation-pd.podspec --verbose --sources='git@gitlab.com:zengjing/Specs.git,https://github.com/CocoaPods/Specs' --use-libraries`

####2.提交podspec文件到Spec库

`pod repo push zengjing-spec Foundation-pd.podspec --verbose --sources='git@gitlab.com:zengjing/Specs.git,https://github.com/CocoaPods/Specs' --use-libraries --allow-warnings`

提交完成之后可以通过`pod search Foundation-pd`检验是否成功：

![pod-search](/images/ios-cocoapods-private-repo/pod-search.png)

说明：

(1)`--verbose`:表示显示全部的日志信息，建议加上这个，方便判断错误信息。

(2)`--sources`:如果我们在podspec里面依赖到一些私有的库之后，直接进行校验是会报错的提示找不到，这里加上我们的Spec仓库的地址告诉CocoaPods找不到的时候去哪里找。

(3)`--allow-warnings`:表示允许警告.

(4)`--use-libraries`:表示使用静态库或者是framework，这里主要是解决当我们依赖一些framework库后校验提示找不到库的时候用到。

(5)在Podfile的文件开头加上Spec的地址，然后就可以向以前一样使用了

```
source 'https://github.com/CocoaPods/Specs.git'
source 'git@gitlab.com:zengjing/Specs.git'
```

###参考资料

1.[《使用Cocoapods创建私有podspec》](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

2.[《深入理解 CocoaPods》](http://objccn.io/issue-6-4/)

3.[《CocoaPods Guides》](https://guides.cocoapods.org)