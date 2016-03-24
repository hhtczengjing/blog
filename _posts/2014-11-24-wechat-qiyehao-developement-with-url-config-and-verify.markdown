---
layout: post
title: "微信企业号开发之开启回调模式"
date: 2014-11-24 19:53:17 +0800
comments: true
tags: WeChat
---

[企业号](https://qy.weixin.qq.com)是微信为企业客户提供的移动应用入口。它帮助企业建立员工、上下游供应链与企业IT系统间的连接。利用企业号，企业或第三方合作伙伴可以帮助企业快速、低成本的实现高质量的移动轻应用，实现生产、管理、协作、运营的移动化。

![wechat_qyh.jpg](/images/wechat_qyh_url_config/wechat_qyh.jpg)

本文主要介绍如何使用Java语言进行回调模式的URL验证等内容。

###开启回调模式配置

1、到应用中心选择需要使用回调模式开发的应用，点击进入后会有如下的模式可供选择：

![wechat_qyh_url_config_001](/images/wechat_qyh_url_config/wechat_qyh_url_config_001.png)

2、选择回调模式点击进入回调模式的配置界面，如下图所示：

![wechat_qyh_url_config_002](/images/wechat_qyh_url_config/wechat_qyh_url_config_002.png)

当你开启应用的回调模式时，企业号会要求你填写应用的URL、Token、EncodingAESKey三个参数，各个参数说明如下：

（1）URL是企业应用接收企业号推送请求的访问协议和地址，支持http或https协议(目前只支持80/443端口)。

（2）Token可由企业任意填写，用于生成签名。

（3）EncodingAESKey用于消息体的加密，是AES密钥的Base64编码。

###验证回调URL及密钥

   当提交前面的信息时，企业号将发送GET请求到填写的URL上，GET请求携带四个参数，企业在获取时需要做urldecode处理，否则会验证不成功。
   
   ![wechat_qyh_url_config_003](/images/wechat_qyh_url_config/wechat_qyh_url_config_003.png)
   
   企业通过参数msg_signature对请求进行校验，如果确认此次GET请求来自企业号，那么企业应用对echostr参数解密并原样返回echostr明文(不能加引号)，则接入验证生效，回调模式才能开启。后续回调企业时都会在请求URL中带上以上参数（echostr除外），校验方式与首次验证URL一致。

开发步骤如下：

####下载官方加解密库及示例

1、[到官方指定的地址下载](http://qydev.weixin.qq.com/java.zip)后解压java.zip文件，解压后的文件里面包含Java版本的加解密的类和示例程序，说明如下：

（1）com\qq\weixin\mp\aes目录下是用户需要用到的接入企业微信的接口，其中WXBizMsgCrypt.java文件提供的WXBizMsgCrypt类封装了用户接入企业微信的三个接口，其它的类文件用户用于实现加解密，用户无须关心。sample.java文件提供了接口的使用示例。

（2）WXBizMsgCrypt封装了VerifyURL, DecryptMsg, EncryptMsg三个接口，分别用于开发者验证回调url、接收消息的解密以及开发者回复消息的加密过程。使用方法可以参考Sample.java文件。

（3）请开发者使用jdk1.6或以上的版本。针对org.apache.commons.codec.binary.Base64，需要导入jar包commons-codec-1.9（或comm ons-codec-1.8等其他版本），我们有提供，官方下载地址：http://commons.apache.org/proper/commons-codec/download_codec.cgi

（4）异常`java.security.InvalidKeyException:illegal Key Size`的解决方案：

在官方网站下载JCE无限制权限策略文件，下载后解压，可以看到`local_policy.jar`和`US_export_policy.jar`以及`readme.txt`这样三个文件。如果安装了JRE，将两个jar文件放到`%JRE_HOME%\lib\security`目录下覆盖原来的文件。如果安装了JDK，将两个jar文件放到`%JDK_HOME%\jre\lib\security`目录下覆盖原来文件。

附Java加密框架Jasypt依赖的JCE文件下载地址：

- [JDK6对应版本，点此下载](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)

- [JDK7对应版本，点此下载](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
)

- [JDK8对应版本，点此下载](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

####验证URL有效性的步骤

（1）从GET过来的URL中获取msg_signature、timestamp、nonce和echostr四个参数，需要对URL进行URLDecoder，示例代码如下所示：

```
String req_msg_signature = request.getParameter("msg_signature");
String req_timestamp = request.getParameter("timestamp");
String req_nonce = request.getParameter("nonce");
String req_echostr = request.getParameter("echostr");
String sVerifyMsgSig = URLDecoder.decode(req_msg_signature, "UTF-8");
String sVerifyTimeStamp = URLDecoder.decode(req_timestamp, "UTF-8");
String sVerifyNonce = URLDecoder.decode(req_nonce, "UTF-8");
String sVerifyEchoStr = URLDecoder.decode(req_echostr, "UTF-8");
```

（2）使用网站上配置的Token和EncodingAESKey，以及CorpID，初始化加解密验证工具。

```
String sToken = "Token";//配置URL时配置的Token
String sCorpID = "CorpID";//企业号的CorpID
String sEncodingAESKey = "EncodingAESKey";//配置URL时配置的EncodingAESKey
WXBizMsgCrypt wxcpt = new WXBizMsgCrypt(sToken, sEncodingAESKey, sCorpID);
```

说明：

1）如何获取CorpID，CorpID是表示企业号的唯一标识码，相当于微信服务号/订阅号的AppID，在后台管理界面中的`设置`中可以看到，如下图所示：

![wechat_qyh_url_config_004](/images/wechat_qyh_url_config/wechat_qyh_url_config_004.jpg)

2）WXBizMsgCrypt是前面下载的示例代码中由官方提供的加解密的工具类，可以直接使用。

（3）验证URL的有效性

```
String sEchoStr = wxcpt.VerifyURL(sVerifyMsgSig, sVerifyTimeStamp, sVerifyNonce, sVerifyEchoStr);
```

验证URL有效性的函数各个参数说明：

![wechat_qyh_url_config_005](/images/wechat_qyh_url_config/wechat_qyh_url_config_005.png)


（4）响应微信服务器验证结果（将echostr参数解密并原样返回echostr明文，不能加引号）

```
request.setCharacterEncoding("UTF-8");
response.setContentType("text/xml;charset=UTF-8");
response.getWriter().write(sEchoStr);
```

####验证完成后企业号的配置

验证通过后可以支持用户自定义菜单、用户状态变更通知、用户消息上报和上报用户地理位置几个方面的配置，如下图示所示：

![wechat_qyh_url_config_006](/images/wechat_qyh_url_config/wechat_qyh_url_config_006.png)


###参考资料

1、[《回调模式》](http://qydev.weixin.qq.com/wiki/index.php?title=回调模式)

2、[《加解密库下载与返回码》](http://qydev.weixin.qq.com/wiki/index.php?title=加解密库下载与返回码)