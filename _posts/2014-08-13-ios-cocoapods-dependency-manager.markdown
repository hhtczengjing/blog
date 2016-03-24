---
layout: post
title: "使用CocoaPods管理iOS项目中的依赖库"
date: 2014-08-13 23:13:28 +0800
comments: true
tags: iOS
---

一种开发语言发展到一定的程度就会出现相应的依赖管理工具（Dependency Manager）或者是中央代码仓库，例如：

- Java: Maven，Ivy
- Ruby: gems
- Python: pip, easy_install
- Node.js: npm

随着iOS开发者的增加，业界也出现了为iOS程序提供依赖管理的工具，这个工具叫做：[CocoaPods](http://cocoapods.org)。CocoaPods是一个负责管理iOS项目中第三方开源代码的工具，其源码在Github上托管。该项目开始于2011年8月12日，经过一年多的发展，现在已经超过1000次提交，并且持续保持活跃更新。开发iOS项目不可避免地要使用第三方开源库，CocoaPods的出现使得开发者可以节省设置和更新第三方开源库的时间。

![CocoaPods Logo](/images/ios_cocoapods/cocoapods_logo.png)


##安装CocoaPods

CocoaPods是使用

1、由于ruby的软件源因使用Amazon的云服务所以在国内无法访问，需要更新下ruby源，使用`ruby.taobao.org`（淘宝）替换`rubygems.org`（默认）源，具体的说明请戳:[RubyGems 镜像 - 淘宝网](http://ruby.taobao.org)

(1)移除`https://rubygems.org/`源

`gem sources --remove https://rubygems.org/`

(2)添加`https://ruby.taobao.org/`源

`gem sources -a https://ruby.taobao.org/`

(3)检查安装情况，确认只有`https://ruby.taobao.org`

`gem sources -l`

执行结果如下表示替换源成功:

```
*** CURRENT SOURCES ***

https://ruby.taobao.org
```

2、安装CocoaPods非常简单直接使用下面的两行命令即可

`sudo gem install cocoapods`

`pod setup`

补充说明:

(1)如果提示Ruby环境不够新，使用`sudo gem update --system`升级即可，升级完成重复上面的步骤。

(2)`pod setup`是将`Spec`项目复制到当前用户的`.cocoapods/master`目录下，其实就相当于将CocoaPods所管理的一些第三方的库的描述文件同步一份到本地，便于进行查询和安装，以后更新新版本的Spec项目只需要再次执行`pod setup`即可 。


##使用CocoaPods

1、使用CocoaPods管理第三方库

使用CocoaPods需要新建一个名为Podfile的文件，以如下格式将依赖库的名字依次列在文件中，`platform`指的是平台的名称和版本号，`'~> 2.0'`这个指的是版本号，如果没有默认是最新版本。

```
platform :ios, '7.0'
pod 'AFNetworking', '~> 2.0'
pod 'JSONKit', '~> 1.4'
pod 'RegexKitLite'
```
将编辑好的Podfile文件放到项目的根目录中，cd到Podfile文件所在的目录，然后执行下面的命令：

`pod install`

CocoaPods会自动帮你把Podfile文件中列出来的第三方库全部下载下来，并且自动设置好变异参数和依赖关系。

补充说明:

(1)`pod install`命令执行完成之后终端会出现[!] From now on use `CocoaPodsDemo.xcworkspace`.这样一句话，意思就是从现在开始起以后打开项目使用`*.xcworkspace`这个来打开不再使用原来的`*.xcodeproj`.

(2)每次更改了`Podfile`文件，需要执行`pod update`来更新一下。

2、搜索可用的第三方库

`pod search AFNetworking`

执行上面的命令可用查询出和``相关的一些开源的库，下面的是查询结果的示例，可以看到库的名称，最新版本号和lis版本号等信息：

```
-> AFNetworking (2.3.1)
   A delightful iOS and OS X networking framework.
   pod 'AFNetworking', '~> 2.3.1'
   - Homepage: https://github.com/AFNetworking/AFNetworking
   - Source:   https://github.com/AFNetworking/AFNetworking.git
   - Versions: 2.3.1, 2.3.0, 2.2.4, 2.2.3, 2.2.2, 2.2.1, 2.2.0, 2.1.0, 2.0.3, 2.0.2, 2.0.1, 2.0.0, 2.0.0-RC3, 2.0.0-RC2, 2.0.0-RC1, 1.3.4, 1.3.3, 1.3.2, 1.3.1, 1.3.0,
   1.2.1, 1.2.0, 1.1.0, 1.0.1, 1.0, 1.0RC3, 1.0RC2, 1.0RC1, 0.10.1, 0.10.0, 0.9.2, 0.9.1, 0.9.0, 0.7.0, 0.5.1 [master repo]
   - Sub specs:
     - AFNetworking/Serialization (2.3.1)
     - AFNetworking/Security (2.3.1)
     - AFNetworking/Reachability (2.3.1)
     - AFNetworking/NSURLConnection (2.3.1)
     - AFNetworking/NSURLSession (2.3.1)
     - AFNetworking/UIKit (2.3.1)
```

##其他配置

在日常开发中执行`pod install/update`时总是长时间停留在`Analyzing dependencies`上。这是因为在使用CocoaPods进行`update`或者`install`的时候每次会更新获取的`pod specs`。

1、使用specs仓库镜像，执行下面的三行命令可以使用specs仓库镜像

```
pod repo remove master
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
```

如果想切换到oschina的源，将上面的第二行的网站替换成`http://git.oschina.net/akuandev/Specs.git`

第二条命令执行的时候会比较耗时,这个时候要去把整个specs仓库clone一下大概60M左右。

2、如果不想在pod install/pod update的时候不想升级specs库可以使用参数忽略掉

`pod install --verbose --no-repo-update`

`pod update --verbose --no-repo-update`


##参考资料

1、[《IOS 第三方管理库管理 CocoaPods》](http://www.cnblogs.com/superhappy/archive/2013/04/23/3038493.html)

2、[《CocoaPods 安装使用》](http://zl4393753.iteye.com/blog/1838824)

3、[《CocoaPods-iOS项目第三方库管理利器》](http://tiyanzhimei.com/index.php/cocoapods-ios-xiang-mu-di-san-fang-ku-guan-li-li-qi/)

4、[《开源框架:CocoaPods》](http://blog.csdn.net/ysy441088327/article/details/8611731)

5、[《CocoaPods官网》](http://cocoapods.org)

6、[《cocoapods specs 镜像》](http://akinliu.github.io/2014/05/03/cocoapods-specs-/)

