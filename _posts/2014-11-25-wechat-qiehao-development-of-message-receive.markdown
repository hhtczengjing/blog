---
layout: post
title: "微信企业号开发之消息与事件的接收"
date: 2014-11-25 20:52:05 +0800
comments: true
tags: WeChat
---

企业号是微信为企业客户提供的移动应用入口。它帮助企业建立员工、上下游供应链与企业IT系统间的连接。利用企业号，企业或第三方合作伙伴可以帮助企业快速、低成本的实现高质量的移动轻应用，实现生产、管理、协作、运营的移动化。

![wechat_qyh.jpg](http://blog.devzeng.com/images/wechat_qyh_url_config/wechat_qyh.jpg)

将应用设置在回调模式时，企业可以通过回调URL接收员工回复的消息，以及员工关注、点击菜单、上报地理位置等事件。在接收到事件后，企业可以发送被动响应消息，实现员工与企业的互动。企业在接收消息，以及发送被动响应消息时，数据包以xml格式组成，以AES方式加密传输。本文主要介绍如何处理接收到的消息和事件。

###消息与事件的类型

1、事件

（1）关注与取消关注事件

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[UserID]]></FromUserName>
   <CreateTime>1348831860</CreateTime>
   <MsgType><![CDATA[event]]></MsgType>
   <Event><![CDATA[subscribe]]></Event>
   <AgentID>1</AgentID>
</xml>
```
消息字段说明：

![inmessage_type_event_subscribe.png](/images/wechat_qyh_inmessage/inmessage_type_event_subscribe.png)

Event对应的事件类型有：subscribe(订阅)和unsubscribe(取消订阅)两种。

（2）上报地理位置事件

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[FromUser]]></FromUserName>
   <CreateTime>123456789</CreateTime>
   <MsgType><![CDATA[event]]></MsgType>
   <Event><![CDATA[LOCATION]]></Event>
   <Latitude>23.104105</Latitude>
   <Longitude>113.320107</Longitude>
   <Precision>65.000000</Precision>
   <AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_type_event_subscribe.png](/images/wechat_qyh_inmessage/inmessage_type_event_location.png)


（3）上报菜单事件

消息体格式如下：

```
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[FromUser]]></FromUserName>
<CreateTime>123456789</CreateTime>
<MsgType><![CDATA[event]]></MsgType>
<Event><![CDATA[CLICK]]></Event>
<EventKey><![CDATA[EVENTKEY]]></EventKey>
<AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_type_event_subscribe.png](/images/wechat_qyh_inmessage/inmessage_type_event_menu.png)

Event对应的事件类型有：

- CLICK：点击菜单拉取消息的事件推送
- VIEW：点击菜单跳转链接的事件推送
- scancode_push：扫码推事件的事件推送
- scancode_waitmsg：扫码推事件且弹出“消息接收中”提示框的事件推送
- pic_sysphoto：弹出系统拍照发图的事件推送
- pic_photo_or_album：弹出拍照或者相册发图的事件推送
- pic_weixin：弹出微信相册发图器的事件推送
- location_select：弹出地理位置选择器的事件推送

不同的事件类型对应的消息体的格式略有不同，请参考[官方的文档](http://qydev.weixin.qq.com/wiki/index.php?title=接收事件#.E4.B8.8A.E6.8A.A5.E5.9C.B0.E7.90.86.E4.BD.8D.E7.BD.AE.E4.BA.8B.E4.BB.B6)描述。


2、消息

（1）text消息，即文本消息

消息体格式如下：

```
<xml>
	<ToUserName><![CDATA[toUser]]></ToUserName>
	<FromUserName><![CDATA[fromUser]]></FromUserName>
	<CreateTime>1348831860</CreateTime>
	<MsgType><![CDATA[text]]></MsgType>
	<Content><![CDATA[this is a test]]></Content>
	<MsgId>1234567890123456</MsgId>
	<AgentID>1</AgentID>
</xml>
```
消息字段说明：

![inmessage_text_type.png](/images/wechat_qyh_inmessage/inmessage_type_msg_text.png)

（2）image消息，即图片消息

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1348831860</CreateTime>
   <MsgType><![CDATA[image]]></MsgType>
   <PicUrl><![CDATA[this is a url]]></PicUrl>
   <MediaId><![CDATA[media_id]]></MediaId>
   <MsgId>1234567890123456</MsgId>
   <AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_text_type.png](/images/wechat_qyh_inmessage/inmessage_type_msg_image.png)


（3）voice消息，即语言消息

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1357290913</CreateTime>
   <MsgType><![CDATA[voice]]></MsgType>
   <MediaId><![CDATA[media_id]]></MediaId>
   <Format><![CDATA[Format]]></Format>
   <MsgId>1234567890123456</MsgId>
   <AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_text_type.png](/images/wechat_qyh_inmessage/inmessage_type_msg_voice.png)

（4）video消息，即视频消息

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1357290913</CreateTime>
   <MsgType><![CDATA[video]]></MsgType>
   <MediaId><![CDATA[media_id]]></MediaId>
   <ThumbMediaId><![CDATA[thumb_media_id]]></ThumbMediaId>
   <MsgId>1234567890123456</MsgId>
   <AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_text_type.png](/images/wechat_qyh_inmessage/inmessage_type_msg_video.png)

（5）location消息，即位置消息

消息体格式如下：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1351776360</CreateTime>
   <MsgType><![CDATA[location]]></MsgType>
   <Location_X>23.134521</Location_X>
   <Location_Y>113.358803</Location_Y>
   <Scale>20</Scale>
   <Label><![CDATA[位置信息]]></Label>
   <MsgId>1234567890123456</MsgId>
   <AgentID>1</AgentID>
</xml>
```

消息字段说明：

![inmessage_text_type.png](/images/wechat_qyh_inmessage/inmessage_type_msg_location.png)

###消息与事件的接收和解析

1、通过网站上配置的信息初始化加解密的工具

```
String sToken = "Token";//网站配置的Token
String sCorpID = "CorpID";
String sEncodingAESKey = "EncodingAESKey";
WXBizMsgCrypt wxcpt = new WXBizMsgCrypt(sToken, sEncodingAESKey, sCorpID);
```

2、解析出url上的参数，包括消息体签名(msg_signature)，时间戳(timestamp)以及随机数字串(nonce)

```
String sReqMsgSig = URLDecoder.decode(request.getParameter("msg_signature"), "UTF-8");
String sReqTimeStamp = URLDecoder.decode(request.getParameter("timestamp"), "UTF-8");
String sReqNonce = URLDecoder.decode(request.getParameter("nonce"), "UTF-8");
```

3、获取POST过来的消息体的内容

```
String sReqData = inputStream2String(request.getInputStream());
```

将HTTP Request中的InputStream转换为字符串的方法：

```
private String inputStream2String(InputStream in) throws Exception {
	if(in == null) {
		return "";
	}
	StringBuffer out = new StringBuffer();
	byte[] b = new byte[4096];
	for (int n; (n = in.read(b)) != -1;) {
		out.append(new String(b, 0, n, "UTF-8"));
	}
	return out.toString();
}
```

4、验证消息体签名的正确性

```
String sMsg = wxcpt.DecryptMsg(sReqMsgSig, sReqTimeStamp, sReqNonce, sReqData);
```
验证完成后返回解密后的消息体，XML格式

5、将post请求的数据进行xml解析，并将<Encrypt>标签的内容进行解密，解密出来的明文即是用户回复消息的明文，明文格式请参考官方文档

```
WeChatInMessage inMessage = WeChatMessageUtil.parsingInMessage(sMsg);
System.out.println("解析消息类型为：" + inMessage.getMsgType());
```

解析XML的代码如下：

```
public static WeChatInMessage parsingInMessage(String responseInputString) {
	XStream xs = WeChatXStreamFactory.init(false);
	xs.ignoreUnknownElements();
	xs.alias("xml", WeChatInMessage.class);
	WeChatInMessage msg = (WeChatInMessage) xs.fromXML(responseInputString);
	return msg;}
```

###参考资料

1、[《接收消息与事件》](http://qydev.weixin.qq.com/wiki/index.php?title=接收消息与事件)
