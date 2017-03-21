---
layout: post
title: "iOS中fastlane的使用"
date: 2017-01-15 13:00:00 +8000
comments: true
tags: iOS
---

对于一个iOS APP的发布上线，一般来说都需要经历:`编译打包` -> `截图` -> `填写一些说明文字` -> `上传ipa到itunes connect` -> `提交供审核`。每次都要进行这么多“繁琐”的步骤，对于某些步骤可能一次还不能执行成功需要等着界面提示上传错误然后手动重新再来一次（想想都觉得可怕）。

在日常开发中，打包也是最后上线不可缺少的环节，如果需要生成ipa文件通常需要在Xcode里点击`Product` -> `Archive`，然后在弹出来的`Organizer`中选择导出什么类型(`ad hoc`/`enterprise`)的包。对于大项目来说动辄编译十分钟以上的来说，一天打几个包就差不多过去了。

为了解决这些问题，[Felix Krause](https://github.com/KrauseFx)大神写了一个工具集[fastlane](https://github.com/fastlane/fastlane)。fastlane是一套使用Ruby写的自动化工具集，用于iOS和Android的自动化打包、发布等工作。

![fastlane-logo.png](/images/ios-fastlane-in-action/fastlane-logo.png)

### fastlane简介

#### 1、安装fastlane

##### (1)Xcode命令行工具

确保Xcode命令行工具安装最新版本，使用如下命令进行安装：

```
xcode-select --install
```

##### (2)安装fastlane

fastlane的安装非常简单，直接使用如下命令：

```
sudo gem install fastlane
```

即可完成安装。

#### 2、fastlane组件

fastlane是一个工具集，包含了我们日常开发中上线时需要的大部分操作。比如gym/deliver等。下面对每个工具进行介绍：

名称 | 说明
--- | ---         
[deliver](https://github.com/fastlane/fastlane/tree/master/deliver#readme) | 自动上传截图，APP的元数据，二进制(ipa)文件到iTunes Connect
[snapshot](https://github.com/fastlane/fastlane/tree/master/snapshot) | 自动截图（基于Xcode7的UI test）
[frameit](https://github.com/fastlane/fastlane/tree/master/frameit) | 可以把截的图片自动套上一层外边框 
[pem](https://github.com/fastlane/fastlane/tree/master/pem) | 自动生成、更新推送配置文件
[sigh](https://github.com/fastlane/fastlane/tree/master/sigh) | 用来创建、更新、下载、修复Provisioning Profile的工具
[produce](https://github.com/fastlane/fastlane/tree/master/produce) | 如果你的产品还没在iTunes Connect(iTC)或者Apple Developer Center(ADC)建立，produce可以自动帮你完成这些工作 
[cert](https://github.com/fastlane/fastlane/tree/master/cert) | 自动创建管理iOS代码签名证书 
[pilot](https://github.com/fastlane/fastlane/tree/master/pilot) | 管理TestFlight的测试用户，上传二进制文件
[boarding](https://github.com/fastlane/fastlane/tree/master/boarding) | 建立一个添加测试用户界面，发给测试者，可自行添加邮件地址，并同步到iTunes Connect(iTC)
[gym](https://github.com/fastlane/fastlane/tree/master/gym) | 自动化编译打包工具
[match](https://github.com/fastlane/fastlane/tree/master/match) | 证书和配置文件管理工具
[scan](https://github.com/fastlane/fastlane/tree/master/scan) | 自动运行测试工具，并且可以生成漂亮的HTML报告

#### 3、fastlane核心概念

在运行fastlane命令行工具的时候，会读取当前目录下面的`fastlane`文件夹里面的`Fastfile`配置文件。里面定义了一个个的`lane`，下面是官方提供的一个示例：

```
lane :beta do
  increment_build_number
  cocoapods
  match
  testflight
  sh "./customScript.sh"
  slack
end
```

像`increment_build_number`、`cocoapods`这样的一条命令都是一个action，由这样的一个个action组成了一个lane（lane中可以调用其他的lane)。

### fastlane实战

#### 1、初始化

在项目的根目录下面，执行`fastlane init`命令开始初始化。在执行的过程中会要求填写一些项目的资料，如Apple ID等，fastlane会自动检测当前目录下项目的App Name和App Identifier，可以选择自行输入这些信息。初始化完成会在当前目录下面生成一个fastlane的文件夹。

![fastlane-init.png](/images/ios-fastlane-in-action/fastlane-init.png)

最重要的两个文件就是Appfile和Fastfile，主要的说明如下：

（1）Appfile里面存放了App的基本信息包括`app_identifier`、`apple_id`、`team_id`等，如果在init的时候输入了正确的apple_id和密码会自动获取team_id。

（2）Fastfile是最重要的一个文件，在这个里面可以编写和定制我们的自动化脚本，所有的流程控制功能都写在这个文件里面。

说明：

（1）如果在init的时候选择了在iTunes Connect创建App，那么fastlane会调用produce进行初始化，如果没有创建后续可以手动执行`produce init`进行创建。如果没有在初始化的时候选择执行produce流程当然deliver也不会执行，后面可以使用`deliver init`运行是一样的。

（2）在iTunes Connect中成功创建App之后，fastlane文件夹里面会生成一个Deliverfile的文件。Deliverfile文件主要是deliver的一些配置信息。

#### 2、Fastfile文件

Fastfile文件的主要结构如下所示：

```
fastlane_version "2.14.2"
default_platform :ios

platform :ios do
	before_all do
   		cocoapods
  	end
  	
  	lane :test do
  	end
  	
  	lane :beta do
  	end
  	
  	lane :release do
  	end
  	
  	after_all do |lane|

  	end

  	error do |lane, exception|

  	end
end
```

说明：

- （1）fastlane_version：指定fastlane使用的最小版本
- （2）default_platform：指定当前默认的平台，可以选择ios/android/mac
- （3）before_all：在执行每一个lane之前都会调用这部分的内容
- （4）after_all：在每个lane执行完成之后都会执行这部分的内容
- （5）error：每个lane执行出错就会执行这部分的内容
- （6）desc：对lane的描述，fastlane会自动将desc的内容生成说明文档
- （7）lane：定义一个lane(任务)，可以理解为一个函数，我们在执行的时候使用`fastlane [ios] lane名称`


#### 3、Fastfile文件的编写

```
lane :release do |option|
	#根据传入参数version设置app的版本号
	increment_version_number(version_number: option[:version]) 
	#自动增加build号	increment_build_number
    #证书签名
    sigh
    #编译打包
    scheme_name = option[:scheme]
    configuration = 'Release'
    version = get_info_plist_value(path: "./#{scheme_name}/Info.plist", key: "CFBundleShortVersionString")
    build = get_info_plist_value(path: "./#{scheme_name}/Info.plist", key: "CFBundleVersion")
    output_directory = File.expand_path("..", Dir.pwd) + File::Separator + 'build'
    output_name = "#{scheme_name}_#{configuration}_#{version}_#{build}_#{Time.now.strftime('%Y%m%d%H%M%S')}.ipa"
    gym(scheme: scheme_name, clean: true, export_method:'appstore', configuration: configuration, output_directory: output_directory, output_name: output_name)
  end
```

### 参考资料

1、[官方网站](https://fastlane.tools/)

2、[小团队的自动化发布－Fastlane带来的全自动化发布](https://whlsxl.github.io)

3、[Fastlane实战（一）：移动开发自动化之道](http://www.jianshu.com/p/1aebb0854c78)

4、[Fastlane实战（二）：Action和Plugin机制](http://www.jianshu.com/p/0520192c9bd7)

5、[Fastlane实战（五）：高级用法](http://www.jianshu.com/p/faae6f95cbd8)
