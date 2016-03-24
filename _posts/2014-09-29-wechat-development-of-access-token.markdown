---
layout: post
title: "微信公众平台开发之access_token"
date: 2014-09-29 22:15:03 +0800
comments: true
tags: WeChat
---

  ![wechat_development_logo.png](/images/wechat_development_02/wechat_development_logo.png)

  为了使第三方开发者能够为用户提供更多更有价值的个性化服务，微信公众平台开放了许多接口，包括自定义菜单接口、客服接口、获取用户信息接口、用户分组接口、群发接口等，开发者在调用这些接口时，都需要传入一个相同的参数access_token，它是公众账号的全局唯一票据，它是接口访问凭证。本文重点是介绍在实际的开发中如何获取access_token和如何保持获取到的access_token长期有效。
  
##获取access_token的接口说明

以HTTP GET请求的方式向微信服务器发送请求，请求的URL格式如下：

`https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET`

####1、参数说明：

<table>
<tbody>
<tr>
	<th style="width:120px">参数</th>
	<th style="width:120px">是否必须</th>
	<th>说明</th>
</tr>
<tr>
	<td>grant_type</td>
	<td>是</td>
	<td>获取access_token填写client_credential</td>
</tr>
<tr>
	<td>appid</td>
	<td>是</td>
	<td>第三方用户唯一凭证</td>
</tr>
<tr>
	<td>secret</td>
	<td>是</td>
	<td>第三方用户唯一凭证密钥，即appsecret</td>
</tr>
</tbody>
</table>

说明：

(1)AppID和AppSecret可在开发模式中获得;

(2)注意调用所有微信接口时均需使用https协议。

####2、返回数据说明：

正常情况下，微信会返回下述JSON数据包给公众号：

`{"access_token":"ACCESS_TOKEN","expires_in":7200}`

说明如下：

<table>
<tbody>
<tr>
	<th style="width:120px">参数</th>
	<th>说明</th>
</tr>
<tr>
	<td>access_token</td>
	<td>获取到的凭证(至少有512个字符)</td>
</tr>
<tr>
	<td>expires_in</td>
	<td>凭证有效时间，单位：秒</td>
</tr>
</tbody>
</table>

错误时微信会返回错误码等信息，JSON数据包示例如下（该示例为AppID无效错误）:

`{"errcode":40013,"errmsg":"invalid appid"}`

有关`errcode`的说明请参考[全局返回码说明](http://mp.weixin.qq.com/wiki/index.php?title=全局返回码说明)中的对应描述

##最佳实践

`access_token`是公众号的全局唯一票据，公众号调用各接口时都需使用access_token。正常情况下access_token有效期为`7200`秒，重复获取将导致上次获取的access_token失效。由于获取access_token的api调用次数非常有限，微信官方建议开发者全局存储与更新access_token，频繁刷新access_token会导致api调用受限，影响自身业务。

为了减少获取access_token的API的调用次数，在实际的开发中设计采用单例全局存储access_token的值和过期时间。具体的步骤如下：

1、获取access_token

思路是首先会判断本地有没有存储过access_token，如果有存储就判断是否过期。如果本地没有存储或存储的access_token已经过期便向微信服务器申请获取新的access_token。

```
public static String getAccessToken() throws Exception {
	//首先判断本地有无记录，记录是否过期 7200s
    boolean isExpired = WeChatSystemContext.getInstance().isExpired();
    if(isExpired) {        	//拼接url,APPID和APPSECRET从开发者中心获取
    	String APPID = "";//APPID
    	String APPSECRET = "";//APPSECRET
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid="+ APPID + "&secret=" + APPSECRET";
        //发起HTTPS的GET请求
        String jsonStr = WeChatHTTPClient.get(url);
        Map<String, Object> map = JSONObject.parseObject(jsonStr);
        String accessToken = map.get("access_token").toString();
        //记录到配置 access_token 当前时间
        WeChatSystemContext.getInstance().saveLocalAccessonToke(accessToken);
        return accessToken;
    } else {
    	//从配置中直接获取access_token 
    	return WeChatConfigSingleton.getInstance().getAccessToken();
    }
}
```

说明：

(1)`WeChatHTTPClient`这个类是封装的一个HTTP请求的工具类，用户发起http请求返回相应的数据，后续会进行介绍；

(2)`JSONObject`这个是一个解析JSON的第三方库`fastjson`

2、存储access_token的单例类

```
public class WeChatSystemContext {
	private String accessToken;//接口访问凭据
	private long createTime;//接口访问凭据创建时间，理论上是2小时后过期
		static class WeChatSystemContextHolder {
		static WeChatSystemContext instance = new WeChatSystemContext();
	}	
	public static WeChatSystemContext getInstance() {
		return WeChatSystemContextHolder.instance;
	}	
	//是否过期
	public boolean isExpired() {
		long time = new Date().getTime();
		//如果当前记录时间为0
		if(this.createTime <= 0) {
			return true;
		}
		//判断记录时间是否超过7200s
		if(this.createTime/1000 + 7200 < time/1000) {
			return true;
		}
		return false;
	}
		//记录接口访问凭证
	public void saveLocalAccessonToke(String accessToken) {
		this.accessToken = accessToken;
		this.createTime = new Date().getTime();
	}
		public void setAccessToken(String accessToken) {
		this.accessToken = accessToken;
	}
	public String getAccessToken() {
		return accessToken;
	}	public void setCreateTime(long createTime) {
		this.createTime = createTime;
	}
	public long getCreateTime() {
		return createTime;
	}
}
```

##参考资料

1、[《[051] 微信公众平台开发教程第22篇-如何保证access_token长期有效》](http://blog.csdn.net/lyq8479/article/details/25076223)

2、[《获取access token》](http://mp.weixin.qq.com/wiki/index.php?title=获取access_token)
