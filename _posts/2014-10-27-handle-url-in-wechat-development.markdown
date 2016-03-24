---
layout: post
title: "微信公众平台开发之URL处理"
date: 2014-10-27 22:28:12 +0800
comments: true
tags: WeChat
---

  在微信公众平台开发中会遇到很多地方需要向客户端返回一个URL地址，比如用户点击了界面上的菜单，触发了菜单的事件，后台检测到这个事件之后需要想用户返回一些信息，如果信息是以一段文本来展示的而且里面包含一个阅读更多的超链接，这个该怎么处理呢？

熟悉HTML的朋友对超链接一定不会陌生，只要用```<a href="URL地址">这是超链接</a>```这样的形式写一段代码就可以实现一个简单的超链接，用户点击之后会跳转到`href`属性里配置的URL地址。今天主要介绍的是在微信中如何使用超链接和URL的相关处理。

###在文本消息中使用超链接

对于响应给用户的文本消息的消息内容格式如下：

```
<xml>
	<ToUserName><![CDATA[toUser]]></ToUserName>
	<FromUserName><![CDATA[fromUser]]></FromUserName>
	<CreateTime>12345678</CreateTime>
	<MsgType><![CDATA[text]]></MsgType>
	<Content><![CDATA[Hello]]></Content>
</xml>
```
如果我们在返回给用户的文本内容中使用超链接，我们需要在Content的节点的CDATA中拼接相应的HTML代码如我们想要返回给用户如下内容：

```
很感谢您的关注，您需要先进行注册才能使用更多的服务.【点此开始注册】
```

用户在点击点此开始注册之后就可以进行注册了，那么可以把返回的内容设置为如下：

```
很感谢您的关注，您需要先进行注册才能使用更多的服务.<a href="http://wechat.devzeng.com/register">【点此开始注册】</a>
```

说明：

（1）超链接写成下面的两种形式都是不规范的，可能在HTML中这样写是正确的，但是在微信中不推荐这样使用：

```
<a href=http://blog.devzeng.com>曾静的技术博客</a>
```

```
<a href=http://blog.devzeng.com>曾静的技术博客</a>
```

正确的写法应该是使用双引号将URL地址括起来：
     
```
<a href="http://blog.devzeng.com">曾静的技术博客</a>
```

（2）有的时候这个href里面要写的URL会很长，通常我们需要传递微信用户的OpenID用来标识这个用户，有的时候可能还需要一些其他的附加信息，比如服务的安全校验码等等很多信息，那么这样一来这个URL就会变得很长，会导致整个文本内容的长度也会变得很大（微信的文本消息的接口文档中明确表示：回复的消息内容长度不超过2048字节）。下面的部分会介绍使用短网址来解决URL过长的问题。

###短网址

   短网址服务，可能很多朋友都已经不再陌生，特别是在微博应用中十分普遍，比如，当我们在腾讯、新浪微博发微博时有时发很长的网址连接，但由于微博只限制140个字，所以微博就自动把您发的长网址给转换成短网址了。
   
   其实，个人认为短网址不一定真的好记，几位数字+字符的组合，甚至很难记忆。但无可否认在微博和手机短信提醒等限制字数的地方来使用短网址，的确是一个不错的方案。短网址通常使用“比较少字符的网址”+“/”+“代码”，打开短网址网页通常会直接跳转到你要缩短的网址（常见），或者几秒广告后在跳转。比如向百度短网址可以自定义后缀，有些短网址还可以进行泛域名解析，十分方便大家使用。
   
   下面重点介绍如何将一个我们的长URL地址转换成一个短网址的方法，我们是采用第三方的生成工具来处理的，具体如下：
   
1、使用Baidu的短网址服务

```
public static String baidu(String url) {
	String apiUrl = "http://www.dwz.cn/create.php";
	try {
		String json = WeChatHttpHelper.post(apiUrl, "url=" + url, false);
		Map<String, Object> map = JSON.parseObject(json, Map.class);
		int status = Integer.parseInt(map.get("status").toString());
		if(status == 0) {
			return map.get("tinyurl").toString();
		} else {
			LOGGER.info("【生成短网址出错,使用原网址:】" + map.get("err_msg").toString());
			return url;
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	return url;
}
```

上面生成的短网址如`http://dwz.cn/sample`这样的格式，一般来说使用IP地址的网址都被百度认为是非法地址不能生成。

2、使用Tinyurl的短网址服务

```
public static String tinyurl(String url) {
		Map<String, String> param = new HashMap<String, String>();
		param.put("url", url);
		String apiUrl = "http://tinyurl.com/api-create.php";
		try {
			String json = WeChatHttpHelper.get(apiUrl, param, false);
			if(StringUtils.isEmpty(json)) {
				return url;
			} else {
				return json;
			}
		} catch (Exception e) {
			LOGGER.info("【生成短网址出错,使用原网址:】" + e.getMessage());
			e.printStackTrace();
		}
		return url;
	}
```

这个是本人推荐的一个短网址生成的工具，生成的URL如`http://tinyurl.com/weme2013`这样的形式.

3、使用Tencent的短网址服务

```
public static String tencent(String url) {
	Map<String, String> param = new HashMap<String, String>();
	param.put("url", url);
	String apiUrl = "http://vwz.me/API.php";
	try {
		String json = WeChatHttpHelper.get(apiUrl, param, false);
		Map<String, Object> map = JSON.parseObject(json, Map.class);
		String status = map.get("status").toString();
		if("exist".equals(status) || "ok".equals(status)) {
			return map.get("msg").toString();
		} else {
			LOGGER.info("【生成短网址出错,使用原网址:】" + map.get("msg").toString());
			return url;
		}
	} catch (Exception e) {
		LOGGER.info("【生成短网址出错,使用原网址:】" + e.getMessage());
		e.printStackTrace();
	}
	return url;
}
```

上面生成的网址如`http://vwz.me/sample`这样的形式，跳转会稍微慢一些不过整体上来说还是不错的。

4、使用新浪的短网址服务

```
public static String sina(String url) {
	//TODO:我们做了一个艰难的决定:暂不支持Sina(t.cn)
	return url;
}
```

5、使用Google的短网址服务

```
public static String google(String url) {
	//TODO:无法翻越长城，我们做了一个艰难的决定:暂不支持Google(goo.ly)
	return url;
}
```

说明：

（1）由于新浪和接口服务调用需要进行申请就没有使用感兴趣的朋友可以自行研究一下；

（2）由于GWF的原因Google经常出现问题所以也没有使用，这里只是列出有这样服务提供

（3）上面使用的JSON解析工具是fastJSON，发送HTTP请求的工具（WeChatHttpHelper）是自己封装的稍后会进行介绍。

###参考资料

1、[《发送被动响应消息》](http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息#.E5.9B.9E.E5.A4.8D.E6.96.87.E6.9C.AC.E6.B6.88.E6.81.AF)

2、[《 [032] 微信公众帐号开发教程第8篇-文本消息中使用网页超链接》](http://blog.csdn.net/lyq8479/article/details/9157455)

