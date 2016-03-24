---
layout: post
title: "iOS应用程序ipa安装包重签名"
date: 2014-07-20 17:24:52 +0800
comments: true
tags: iOS
---
在日常的工作中经常遇到需要将一些应用程序IPA安装包【已经上架到apple app store,或者是别人写的IPA】发给别人来安装，获取是其他渠道获取来的IPA版本，对于普通人来说安装这些ipa文件有点麻烦，需要使用iTools/iFunBox这些软件来进行安装，不能像企业级的IN-HOUSE方式进行部署，直接在网页上下载。

最近看到iTools/同步推这样的应用可以直接下载应用程序，而且使用的是企业级证书发布的，排除部分企业是特殊渠道分发的，但是其中一份仍然是使用其他公司的证书进行发布打包，这样就提出来一个新的问题关于IPA文件的重签名。

####IPA重签名步骤

下面以wechat.ipa为例简单介绍下重签名的步骤

(1)解压ipa文件，获取到Payload文件

```
unzip wechat.ipa
```
(2)将Payload目录下面的_CodeSignature文件夹删除

```
rm -rf Payload/*.app/_CodeSignature/
```

(3)替换embedded.mobileprovision文件,到一个能正常部署到设备上的程序中拷贝embedded.mobileprovision文件

```
cp embedded.mobileprovision Payload/*.app/embedded.mobileprovision
```

(4)重新签名，“iPhone Distribution: XXXXXX”这个指的是embedded.mobileprovision用到的签名的证书名称

```
/usr/bin/codesign -f -s "iPhone Distribution: XXXXXX" --resource-rules Payload/*.app/ResourceRules.plist Payload/*.app/
```
(5)重新打包

```
zip -r wechat_inhouse.ipa Payload/

rm -rf Payload/
```

P.S. 可以将上面的几个步骤写成一个Shell脚本方便执行，另外目前只支持在MAC电脑上进行重签名。

####IN HOUSE部署

关于如何使用IN HOUSE方式部署应用程序可以参考[《iOS企业级应用部署》](http://blog.devzeng.com/blog/ios-enteprise-deployment.html)


####参考资料

(1)[《iOS开发--in house发布和安装（ipa重新签名）》](http://blog.csdn.net/gtncwy/article/details/10973285)

(2)[《IPA 重签名》](http://blog.rpplusplus.me/blog/2014/03/03/ipa-re-codesign/)