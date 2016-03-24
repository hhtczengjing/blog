---
layout: post
title: "iOS企业级应用部署"
date: 2014-07-20 18:07:16 +0800
comments: true
tags: iOS
---
   在iOS开发计划中有一种是`iOS Developer Enterprise Program(iOS开发者企业级计划)`，对于这种开发者证书发布的应用程序是无法上传到Apple App Store上的，目前对于这种企业级开发的应用程序最好的分发方式是部署到内网服务器上通过网络进行下载。

####IN-HOUSE应用程序分发

下面介绍下使用网络方式进行部署的方式,用户直接在iPhone/iPad的Safari浏览器里面输入URL地址即可安装，如在浏览器输入`http://www.itools.cn`能出现如下的网页，点击下载安装即可安装iTools这款软件。

![下载iTools](/images/ios_enterprise_deploy/ios_enterprise_deploy.png)

#####前提条件

(1)已鉴定的用户可以访问的安全Web服务器，对于iOS7.1以上的设备必须有HTTPS的服务器

(2).ipa格式的iOS应用程序，经构建用于发布/生产(使用了企业级预制描述文件)

(3)关于应用程序描述的清单文件(.plist)

(4)使用了`itms-services://?action=download-manifest&url=`形式的HTML超链接

#####部署步骤

(1)打包ipa文件

使用Xcode的`Product->Archive`来进行打包，在选择发布的方式上选择`Save for Enterprise or Ad- Hoc Deployment`,生成IPA文件即可。
  
(2)配置plist文件
在plist文件中必须配置IPA文件的下载路径、应用的名称和应用的`bundle-identifier`(需要和Xcode中的配置一致)

```
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"><plist version="1.0"><dict>   <key>items</key>   <array>       <dict>           <key>assets</key>           <array>
	           <!— 必填项，ipa文件 —>               <dict>                   <key>kind</key>                   <string>software-package</string>                   <key>url</key>                   <string>http://www.example.com/apps/foo.ipa</string>               </dict>               <!-- 可选项display-image: 在下载过程中显示的图标.—>               <dict>                   <key>kind</key>                   <string>display-image</string>                   <!-- optional.indicates if icon needs shine effect applied.-->                   <key>needs-shine</key>                   <true/>                   <key>url</key>                   <string>http://www.example.com/image.57x57.png</string>               </dict>               <!-- 可选项，full-size-image:(512x512)在iTunes使用的图标.-->               <dict>                   <key>kind</key>                   <string>full-size-image</string>                   <key>needs-shine</key>                   <true/>                   <key>url</key>		   			<string>http://www.example.com/image.512x512.jpg</string>               </dict>           </array><key>metadata</key>           <dict>               <!-- 必填项，应用程序的indentifier>               <key>bundle-identifier</key>               <string>com.example.fooapp</string>               <!-- 可选项，应用程序的版本号 -->               <key>bundle-version</key>               <string>1.0</string>               <!-- —必填项 下载类型默认为software -—>               <key>kind</key>               <string>software</string>               <!-- 可选项 在提示下载时显示，一般为公司的名称 -->               <key>subtitle</key>               <string>Apple</string>               <!-- 必填项，在下载的过程中显示.—>               <key>title</key>               <string>Example Corporate App</string>           </dict>       </dict>   </array></dict></plist>
```

(3)编写下载页面，其中URL指向的是plist文件的路径，对于iOS7.1以上的系统这里的plist的路径必须是HTTPS的，iOS7.1以前的则不需强制使用HTTPS

```
<a href="itms-services://?action=download-manifest&url=http://www.example.com/test.plist">下载应用</a>
```

(4)设定服务器MIME类型

对于OS X Server,将以下MIME类型添加到WEB服务的“MIME TYPES”设置中：

`application/octet-stream ipa`

`text/xml plist`

对于IIS，使用IIS Manager在服务器的“属性”页面中添加MIME类型

`.ipa application/octet-stream`

`.plist text/xml`