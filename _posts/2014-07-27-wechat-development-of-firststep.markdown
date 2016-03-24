---
layout: post
title: "微信公众平台开发之开启开发者模式"
date: 2014-07-27 15:47:07 +0800
comments: true
tags: WeChat
---

说起来接触微信公众平台账号开发差不多快有半年了，在这期间利用零散的时间也开发了个企业服务号。经过这个公众账号的开发，对目前微信公众平台的开放的API算是比较熟悉了，对于常见的消息类型（text、news、music、voice、location等）等都有了一些认识和在使用过程中的技巧有了一定的认识。在开发过程中遇到的一些问题也有了一些领悟，所以想将一些经验分享出来，让大家共同进步。


##开发者模式配置

1、在浏览器中输入`https://mp.weixin.qq.com`或[点此打开微信后台界面](https://mp.weixin.qq.com)

![微信后台登录界面](/images/wechat_develop/wechat_login.png)

2、申请消息接口:登录成功后选择`开发者中心`->`填写服务器配置`，微信公众平台提供的高级功能包含两种模式：`编辑模式`和`开发模式`，并且这两种模式是互斥关系，即两种模式不能同时开启。

那两种模式有什么区别呢？作为开发人员到底要开启哪一种呢？

- 编辑模式：主要针对非编程人员及信息发布类公众帐号使用。开启该模式后，可以方便地通过界面配置“自定义菜单”和“自动回复的消息”。

- 开发模式：主要针对具备开发能力的人使用。开启该模式后，能够使用微信公众平台开放的接口，通过编程方式实现自定义菜单的创建、用户消息的接收/处理/响应。这种模式更加灵活，建议有开发能力的公司或个人都采用该模式。

![微信后台登录界面](/images/wechat_develop/wechat_config.png)

这里需要填写`URL`和`Token`两个值。URL指的是能够接收处理微信服务器发送的GET/POST请求的地址，并且是已经存在的，现在就能够在浏览器访问到的地址，这就要求我们先把公众帐号后台处理程序开发好（至少应该完成了对GET请求的处理）并部署在公网服务器上。

3、验证URL有效性

开发者提交信息后，微信服务器将发送GET请求到填写的URL上，GET请求携带四个参数：

- signature 微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp、nonce参数。
- timestamp 时间戳
- nonce 随机数 
- echostr 随机字符串

开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。

```
加密/校验流程如下：
1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
```

#####PHP版验证URL有效性

```
private function checkSignature()
{
	$signature = $_GET["signature"];
    $timestamp = $_GET["timestamp"];
    $nonce = $_GET["nonce"];
    	
	$token = TOKEN;
	$tmpArr = array($token, $timestamp, $nonce);
	sort($tmpArr, SORT_STRING);
	$tmpStr = implode($tmpArr);
	$tmpStr = sha1($tmpStr);
	
	if($tmpStr == $signature){
		return true;
	}else{
		return false;
	}
}
```

#####Java版验证URL有效性

(1)获取signature、timestamp、nonce和echostr四个参数

```
public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// 微信加密签名
		String signature = request.getParameter("signature");
		// 时间戳
		String timestamp = request.getParameter("timestamp");
		// 随机数
		String nonce = request.getParameter("nonce");
		// 随机字符串
		String echostr = request.getParameter("echostr");

		PrintWriter out = response.getWriter();
		// 通过检验signature对请求进行校验，若校验成功则原样返回echostr，表示接入成功，否则接入失败
		if (SignUtil.checkSignature(signature, timestamp, nonce)) {
			out.print(echostr);
		}
		out.close();
		out = null;
	}
```

(2)验证签名

```
public static boolean checkSignature(String signature, String timestamp, String nonce) {
	String[] arr = new String[] { token, timestamp, nonce };
	// 将token、timestamp、nonce三个参数进行字典序排序
	Arrays.sort(arr);
	StringBuilder content = new StringBuilder();
	for (int i = 0; i < arr.length; i++) {
		content.append(arr[i]);
	}
	MessageDigest md = null;
	String tmpStr = null;

	try {
		md = MessageDigest.getInstance("SHA-1");
		// 将三个参数字符串拼接成一个字符串进行sha1加密
		byte[] digest = md.digest(content.toString().getBytes());
		tmpStr = byteToStr(digest);
	} catch (NoSuchAlgorithmException e) {
		e.printStackTrace();
	}

	content = null;
	// 将sha1加密后的字符串可与signature对比，标识该请求来源于微信
	return tmpStr != null ? tmpStr.equals(signature.toUpperCase()) : false;
}
	
//将字节数组转换为十六进制字符串
private static String byteToStr(byte[] byteArray) {
	String strDigest = "";
	for (int i = 0; i < byteArray.length; i++) {
		strDigest += byteToHexStr(byteArray[i]);
	}
	return strDigest;
}

//将字节转换为十六进制字符串
private static String byteToHexStr(byte mByte) {
	char[] di={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F' };
	char[] tempArr = new char[2];
	tempArr[0] = di[(mByte >>> 4) & 0X0F];
	tempArr[1] = di[mByte & 0X0F];

	String s = new String(tempArr);
	return s;
}
```

###注意事项

- 验证成功后返回的字符串必须为`echostr`这个，否则验证不会成功

- 接口配置里面写的Token和代码里面校验用的`Token`是指的同一个字符串
 
4、成为开发者

验证URL有效性成功后即接入生效，成为开发者。如果公众号类型为服务号（订阅号只能使用普通消息接口），可以在公众平台网站中申请认证，认证成功的服务号将获得众多接口权限，以满足开发者需求。

此后用户每次向公众号发送消息、或者产生自定义菜单点击事件时，响应URL将得到推送。

公众号调用各接口时，一般会获得正确的结果，具体结果可见对应接口的说明。返回错误时，可根据返回码来查询错误原因。[全局返回码说明](http://mp.weixin.qq.com/wiki/index.php?title=全局返回码说明)

用户向公众号发送消息时，公众号方收到的消息发送者是一个OpenID，是使用用户微信号加密后的结果，每个用户对每个公众号有一个唯一的OpenID。

`此外请注意，微信公众号接口只支持80接口。`


##参考资料
1、[《接入指南》](http://mp.weixin.qq.com/wiki/index.php?title=接入指南)

2、[《全局返回码说明》](http://mp.weixin.qq.com/wiki/index.php?title=全局返回码说明)

3、[《[027] 微信公众帐号开发教程第3篇-开发模式启用及接口配置》](http://blog.csdn.net/lyq8479/article/details/8944988)