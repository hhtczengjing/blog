---
layout: post
title: "iOS中JavaScript和OC交互"
date: 2015-01-10 20:16:21 +0800
comments: true
tags: iOS
---

在iOS开发中很多时候我们会和UIWebView打交道，目前国内的很多应用都采用了UIWebView的混合编程技术，最常见的是微信公众号的内容页面。前段时间在做微信公众平台相关的开发，发现很多应用场景都是利用HTML5和UIWebView来实现的。

###机制

Objective-C语言调用JavaScript语言，是通过UIWebView的
`- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;`的方法来实现的。该方法向UIWebView传递一段需要执行的JavaScript代码最后获取执行结果。

JavaScript语言调用Objective-C语言，并没有现成的API，但是有些方法可以达到相应的效果。具体是利用UIWebView的特性：在UIWebView的内发起的所有网络请求，都可以通过delegate函数得到通知。  

###示例

下面提供一个简单的例子介绍如何相互的调用，实现的效果是在界面上点击一个链接，然后弹出一个对话框判断是否登录成功。

![uiwebview_js_demo.png](/images/uiwebview_js/uiwebview_js_demo.png)

（1）示例的HTML的源码如下：

```
<html>
    <head>
        <meta http-equiv="content-type" content="text/html;charset=utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
        <meta content="always" name="referrer" />
        <title>测试网页</title>
    </head>
    <body>
        <br />
        <a href="devzeng://login?name=zengjing&password=123456">点击链接</a>
    </body>
</html>
```

（2）UIWebView Delegate回调方法为：

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    NSURL *url = [request URL];
    if([[url scheme] isEqualToString:@"devzeng"]) {
        //处理JavaScript和Objective-C交互
        if([[url host] isEqualToString:@"login"])
        {
            //获取URL上面的参数
            NSDictionary *params = [self getParams:[url query]];
            BOOL status = [self login:[params objectForKey:@"name"] password:[params objectForKey:@"password"]];
            if(status)
            {
                //调用JS回调
                [webView stringByEvaluatingJavaScriptFromString:@"alert('登录成功!')"];
            }
            else
            {
                [webView stringByEvaluatingJavaScriptFromString:@"alert('登录失败!')"];
            }
        }
        return NO;
    }
    return YES;
}
```

说明：

1、同步和异步的问题
   
（1）Objective-C调用JavaScript代码的时候是同步的
   
   `- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;`
   
（2）JavaScript调用Objective-C代码的时候是异步的
   
   `- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;`
   
2、常见的JS调用

（1）获取页面title

`NSString *title = [webview stringByEvaluatingJavaScriptFromString:@"document.title"];`

（2）获取当前的URL

`NSString *url = [webview stringByEvaluatingJavaScriptFromString:@"document.location.href"];`

3、使用第三方库

`https://github.com/marcuswestin/WebViewJavascriptBridge`

###使用案例

1、动态将网页上的图片全部缩放

JavaScript脚本如下：

```
function ResizeImages() {
	var myImg, oldWidth;
	var maxWidth = 320;
	for(i = 0; i < document.images.length; i++) {
		myImg = document.images[i];
		if(myImg.width > maxWidth) {
			oldWidth = myImg.width;
			myImg.width = maxWidth;
			myImg.heith = myImg.height*(maxWidth/oldWidth);
		}
	}
}
```

在iOS代码中添加如下代码：

```
[webView stringByEvaluatingJavaScriptFromString:  
 @"var script = document.createElement('script');"   
 "script.type = 'text/javascript';"   
 "script.text = \"function ResizeImages() { "   
     "var myimg,oldwidth;"  
     "var maxwidth=380;" //缩放系数   
     "for(i=0;i <document.images.length;i++){"   
         "myimg = document.images[i];"  
         "if(myimg.width > maxwidth){"   
             "oldwidth = myimg.width;"   
             "myimg.width = maxwidth;"   
             "myimg.height = myimg.height * (maxwidth/oldwidth);"   
         "}"   
     "}"
 "}\";"   
 "document.getElementsByTagName('head')[0].appendChild(script);"];
[webView stringByEvaluatingJavaScriptFromString:@"ResizeImages();"];
```

###参考资料

1、[《关于UIWebView和PhoneGap的总结》](http://blog.devtang.com/blog/2012/03/24/talk-about-uiwebview-and-phonegap/)

2、[《iOS开发之Objective-C与JavaScript的交互 》](http://www.uml.org.cn/mobiledev/201108181.asp)