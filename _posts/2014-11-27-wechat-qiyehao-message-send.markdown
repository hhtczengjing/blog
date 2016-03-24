---
layout: post
title: "微信企业号开发之消息发送"
date: 2014-11-27 20:37:11 +0800
comments: true
tags: WeChat
---


和服务号不同的是企业号中放开了发送消息的限制，将企业内部的业务需要和微信的消息体系结合起来可以带来更多的便利。在前面也介绍到了在响应用户的请求的时候如果无法及时回应可以直接返回空的消息体，然后调用主动发送消息的接口进行消息的发送来解决这个问题。

###发送消息的类型及数据格式

1、text消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": "PartyID1|PartyID2 ",
   "totag": "TagID1|TagID2",
   "msgtype": "text",
   "agentid": "1",
   "text": {
       "content": "消息内容"
   },
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_text.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_text.png)

2、image消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": " PartyID1 | PartyID2 ",
   "msgtype": "image",
   "agentid": "1",
   "image": {
       "media_id": "MEDIA_ID"
   },
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_image.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_image.png)

3、voice消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": " PartyID1 | PartyID2 ",
   "totag": " TagID1 | TagID2 ",
   "msgtype": "voice",
   "agentid": "1",
   "voice": {
       "media_id": "MEDIA_ID"
   },
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_voice.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_voice.png)

4、video消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": " PartyID1 | PartyID2 ",
   "totag": " TagID1 | TagID2 ",
   "msgtype": "video",
   "agentid": "1",
   "video": {
       "media_id": "MEDIA_ID",
       "title": "Title",
       "description": "Description"
   },
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_video.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_video.png)

5、file消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": " PartyID1 | PartyID2 ",
   "totag": " TagID1 | TagID2 ",
   "msgtype": "file",
   "agentid": "1",
   "file": {
       "media_id": "MEDIA_ID"
   },
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_file.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_file.png)

6、news消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": " PartyID1 | PartyID2 ",
   "totag": " TagID1 | TagID2 ",
   "msgtype": "news",
   "agentid": "1",
   "news": {
       "articles":[
           {
               "title": "Title",
               "description": "Description",
               "url": "URL",
               "picurl": "PIC_URL"
           },
           {
               "title": "Title",
               "description": "Description",
               "url": "URL",
               "picurl": "PIC_URL"
           }    
       ]
   }
}
```

参数说明：

![wechat_msg_send_news.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_news.png)

7、mpnews消息

消息数据格式：

```
{
   "touser": "UserID1|UserID2|UserID3",
   "toparty": "PartyID1|PartyID2",
   "totag": "TagID1|TagID2",
   "msgtype": "mpnews",
   "agentid": "1",
   "mpnews": {
       "articles":[
           {
               "title": "Title",
               "thumb_media_id": "id",
               "author": "Author",
               "content_source_url": "URL",
               "content": "Content",
               "digest": "Digest description",
               "show_cover_pic": "0"
           },
           {
               "title": "Title",
               "thumb_media_id": "id",
               "author": "Author",
               "content_source_url": "URL",
               "content": "Content",
               "digest": "Digest description",
               "show_cover_pic": "0"
           }
       ]
   }
   "safe":"0"
}
```

参数说明：

![wechat_msg_send_mpnews.png](/images/wechat_qiyehao_msg_send/wechat_msg_send_mpnews.png)



###发送消息处理示例

1、获取AccessToken

AccessToken是企业号的全局唯一票据，调用接口时需携带AccessToken。AccessToken需要用CorpID和Secret来换取，不同的Secret会返回不同的AccessToken。正常情况下AccessToken有效期为7200秒，有效期内重复获取返回相同结果，并自动续期。

（1）获取CorpID和Secret

   CorpID是企业号的标识，每个企业号拥有一个唯一的CorpID，Secret是管理组凭证密钥，可以在设置权限管理中获取如下图所示：

![corpid_secrect.jpg](/images/wechat_qiyehao_msg_send/corpid_secrect.jpg)

申请的超级管理员账号不能获取到Secret，必须指定成员作为管理员进行开发，然后配置相应的权限即可。


（2）向微信请求生成AccessToken

```
String corpid = "CorpID";
String secret = "Secret";
String url = "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=" + corpid + "&corpsecret=" + secret;
String jsonStr = WeChatHTTPClient.get(url);//发起HTTP请求，获取返回的JSON字符串
Map<String, Object> map = JSONObject.parseObject(jsonStr);//解析JSON字符串
String accessToken = map.get("access_token").toString();//获取access_token
```

2、发送消息

（1）消息发送的接口说明

发送消息前需要向`https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=ACCESS_TOKEN`（ACCESS_TOKEN指的是前面获取到的AccessToken）这个URL发送HTTPS，POST请求的消息体是前面介绍的不同的JSON格式的数据。

请求的

（2）封装消息发送的接口

```
public static boolean sendMessage(String token, String json) {
	try {
		String url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=" + token;
		String jsonStr = WeChatHTTPClient.post(url, json, true);//向接口发起POST请求
		Map<String, Object> map = JSONObject.parseObject(jsonStr);//解析JSON数据
		if(Integer.parseInt(map.get("errcode").toString()) == 0) {
			return true;
		} else {
			return false;
		}
	} catch (Exception e) {
		e.printStackTrace();
		return false;
	}
}
```
（3）示例

```
String token = "通过前面的方式获取到的Token";
String json = "{\"touser\":\"@all\",\"msgtype\":\"text\",\"agentid\":\"1\",\"text\":{\"content\":\"消息内容\"},\"safe\":\"0\"}";
sendMessage(token, json);
```

###参考资料

1、[《发送接口说明》](http://qydev.weixin.qq.com/wiki/index.php?title=发送接口说明)

2、[《消息类型及数据格式》](http://qydev.weixin.qq.com/wiki/index.php?title=消息类型及数据格式)

3、[《C#开发微信门户及应用(19)-微信企业号的消息发送（文本、图片、文件、语音、视频、图文消息等）》](http://www.cnblogs.com/wuhuacong/p/3995494.html)