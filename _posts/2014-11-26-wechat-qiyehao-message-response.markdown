---
layout: post
title: "微信企业号开发之消息与事件的被动响应"
date: 2014-11-26 20:43:26 +0800
comments: true
tags: WeChat
---

企业号是微信为企业客户提供的移动应用入口。它帮助企业建立员工、上下游供应链与企业IT系统间的连接。利用企业号，企业或第三方合作伙伴可以帮助企业快速、低成本的实现高质量的移动轻应用，实现生产、管理、协作、运营的移动化。

![](http://blog.devzeng.com/images/wechat_qyh_url_config/wechat_qyh.jpg)

本文重点介绍接收到用户的消息请求后如何给用户响应消息。

###消息与事件的类型

1、text消息

消息的明文XML结构：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName> 
   <CreateTime>1348831860</CreateTime>
   <MsgType><![CDATA[text]]></MsgType>
   <Content><![CDATA[this is a test]]></Content>
</xml>
```

消息字段描述：

![wechat_respmessage_type_text](/images/wechat_qiyehao_message_response/wechat_respmessage_type_text.png)

2、voice消息

消息的明文XML结构：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1357290913</CreateTime>
   <MsgType><![CDATA[voice]]></MsgType>
   <Voice>
       <MediaId><![CDATA[media_id]]></MediaId>
   </Voice>
</xml>
```

消息字段描述：

![wechat_respmessage_type](/images/wechat_qiyehao_message_response/wechat_respmessage_type_voice.png)

3、video消息

消息的明文XML结构：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>1357290913</CreateTime>
   <MsgType><![CDATA[video]]></MsgType>
   <Video>
       <MediaId><![CDATA[media_id]]></MediaId>
       <Title><![CDATA[title]]></Title>
       <Description><![CDATA[description]]></Description>
   </Video>
</xml>
```

消息字段描述：

![wechat_respmessage_type](/images/wechat_qiyehao_message_response/wechat_respmessage_type_video.png)

4、news消息

消息的明文XML结构：

```
<xml>
   <ToUserName><![CDATA[toUser]]></ToUserName>
   <FromUserName><![CDATA[fromUser]]></FromUserName>
   <CreateTime>12345678</CreateTime>
   <MsgType><![CDATA[news]]></MsgType>
   <ArticleCount>2</ArticleCount>
   <Articles>
       <item>
           <Title><![CDATA[title1]]></Title> 
           <Description><![CDATA[description1]]></Description>
           <PicUrl><![CDATA[picurl]]></PicUrl>
           <Url><![CDATA[url]]></Url>
       </item>
       <item>
           <Title><![CDATA[title]]></Title>
           <Description><![CDATA[description]]></Description>
           <PicUrl><![CDATA[picurl]]></PicUrl>
           <Url><![CDATA[url]]></Url>
       </item>
   </Articles>
</xml>
```

消息字段描述：

![wechat_respmessage_type](/images/wechat_qiyehao_message_response/wechat_respmessage_type_news.png)

**响应的消息同样应该经过加密，并带上msg_signature、timestamp、nonce及密文，其中timestamp、nonce由企业指定，msg_signature、密文经特定算法生成**。加密后的消息体格式如下：

```
<xml>
   <Encrypt><![CDATA[msg_encrypt]]></Encrypt>
   <MsgSignature><![CDATA[msg_signature]]></MsgSignature>
   <TimeStamp>timestamp</TimeStamp>
   <Nonce><![CDATA[nonce]]></Nonce>
</xml>
```

###消息与事件的响应处理

1、生成时间戳(timestamp)和随机数字串(nonce)以便生成消息体签名，也可以直接用从公众平台post的url上解析出的对应值。

```
String sRespTimeStamp = new Date().getTime() + ""; //生成时间戳
String sRespNonce = "1372623149";//随机数字串
```

2、生成响应消息的明文XML数据

```
String sRespData = "<xml><ToUserName><![CDATA[接收的用户名]]></ToUserName><FromUserName><![CDATA[CorpID]]></FromUserName><CreateTime>1348831860</CreateTime><MsgType><![CDATA[text]]></MsgType><Content><![CDATA[这是消息体内容!]]></Content></xml>";
```

3、对XML明文数据加密，生成加密后的密文消息

```
String sEncryptMsg = wxcpt.EncryptMsg(sRespData, sRespTimeStamp, sRespNonce);
```

3、响应请求

```
response.setCharacterEncoding("UTF-8");
response.setContentType("text/xml");
response.getWriter().write(sEncryptMsg);
```

附注：

假如无法保证在五秒内处理并回复，可以直接回复空串，企业号不会对此作任何处理，并且不会发起重试。这种情况下，可以使用发消息接口进行异步回复。主动发送消息的接口处理会在稍后进行介绍。

###参考资料

1、[《被动响应消息》](http://qydev.weixin.qq.com/wiki/index.php?title=被动响应消息)