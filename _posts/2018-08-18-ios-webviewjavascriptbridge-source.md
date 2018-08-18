---
layout: post
title: "iOS中WebViewJavaScriptBridge源码分析"
date: 2018-08-17 22:46:29 +0800
comments: true
categories: iOS
---

[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)是一个WebView中JavaScript和Native进行交互的框架，使用这个框架能够实现JavaScript和Objective-C之间进行消息的互通。另外这个框架设计的也是非常的简介，只有如下几个文件：

(1) `WebViewJavascriptBridge_JS`
   
该文件中只有一个方法`NSString * WebViewJavascriptBridge_js(void);`, 该方法主要是提供拼接创建一个JavaScript的脚步代码，在旧版中该文件生成的JS代码是用一个txt文件进行保存的。生成的JS代码主要负责对Native端发送的消息进行处理与将JavaScript端的消息发送到Native端，另外进行一些全局的组件的注册等。
   
(2) `WebViewJavascriptBridgeBase`

实现了`Bridge`的初始化，提供WebView、Native之间的消息通道

(3) `WKWebViewJavascriptBridge/WebViewJavascriptBridge`

分别负责WKWebView和UIWebView同Native之间的消息处理，实现了`Handler`的注册和调用逻辑，通过对各自平台下面的一些关键回调方法进行处理实现对JavaScript端发送的消息进行拦截并进行解析处理等流程。

### 如何使用

#### 1、JavaScript端

Bridge的创建，调用本函数会触发Native端执行`WebViewJavascriptBridge_JS`文件里面的初始化JS代码

```
function setupWebViewJavascriptBridge(callback) {
	if (window.WebViewJavascriptBridge) { 
	    return callback(WebViewJavascriptBridge); 
	}
	if (window.WVJBCallbacks) { 
	    return window.WVJBCallbacks.push(callback); 
	}
	window.WVJBCallbacks = [callback];
	var WVJBIframe = document.createElement('iframe');
	WVJBIframe.style.display = 'none';
	WVJBIframe.src = 'https://__bridge_loaded__';
	document.documentElement.appendChild(WVJBIframe);
	setTimeout(function() {
	    document.documentElement.removeChild(WVJBIframe)
	}, 0)
}
```

该`setupWebViewJavascriptBridge`函数主要是通过动态在WebView上面创建一个隐藏的`iframe`实现在WebView上面打开`https://__bridge_loaded__`这样的一个链接地址。具体的流程后面的分析部分会进行介绍。

初始化完成后就可以进行Handler的注册与调用了

```
setupWebViewJavascriptBridge(function(bridge) {
	// 注册JS端的Handler供Native进行调用
	bridge.registerHandler('JS Echo', function(data, responseCallback) {
		console.log("JS Echo called with:", data)
		responseCallback(data)
	})
	// 调用Native端注册的Handler
	bridge.callHandler('ObjC Echo', {'key':'value'}, function responseCallback(responseData) {
		console.log("JS received response:", responseData)
	})
})
```

#### 2、Native端初始化与使用

```
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView];
// 注册Native端的Handler供JS端使用
[self.bridge registerHandler:@"ObjC Echo" handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"ObjC Echo called with: %@", data);
	responseCallback(data);
}];
// 调用JS端注册的Handler
[self.bridge callHandler:@"JS Echo" data:nil responseCallback:^(id responseData) {
	NSLog(@"ObjC received response: %@", responseData);
}];
```

### 源码分析

WebViewJavascriptBridge框架主要是Native和JavaScript之间的双向交互，那么接下来从这几个方面来进行分析：

#### 1、Native注册Handler

Native端注册Handler可以供JavaScript进行调用，在Native端注册Handler的时候会调用如下的代码：

```
- (void)registerHandler:(NSString *)handlerName handler:(WVJBHandler)handler {
    _base.messageHandlers[handlerName] = [handler copy];
}
```
从源码来看其实就是在bridge初始化的时候会创建一个`WebViewJavascriptBridgeBase`的实例`_base`，注册Handler其实就是将Handler处理逻辑的block代码记录到`_base`的`messageHandlers`里面，该结构是一个`NSMutableDictionary`。在使用的时候直接通过Handler的名字就可以找到对应的block。

#### 2、JavaScript注册Handler

JavaScript端注册Handler的逻辑和Native这边差不多，也是在全局有一个对象进行的记录，可以理解为就是一个可变的字典，记录了Handler的名称和具体的实现代码回调。

```
function registerHandler(handlerName, handler) {
    messageHandlers[handlerName] = handler;
}
```

#### 3、Native调用JavaScript的Handler

看完了Native/JavaScript端的注册Handler的逻辑，接下来看下Native端是如何调用JavaScript端的Handler的。

首先会调用下面的`callHandler`方法：

```
- (void)callHandler:(NSString *)handlerName {
    [self callHandler:handlerName data:nil responseCallback:nil];
}

- (void)callHandler:(NSString *)handlerName data:(id)data {
    [self callHandler:handlerName data:data responseCallback:nil];
}

- (void)callHandler:(NSString *)handlerName data:(id)data responseCallback:(WVJBResponseCallback)responseCallback {
    [_base sendData:data responseCallback:responseCallback handlerName:handlerName];
}
```

最终会调用`WebViewJavascriptBridgeBase`的`sendData...`的方法：

```
- (void)sendData:(id)data responseCallback:(WVJBResponseCallback)responseCallback handlerName:(NSString*)handlerName {
    //构造消息
    NSMutableDictionary* message = [NSMutableDictionary dictionary];
    //消息体
    if (data) {
        message[@"data"] = data;
    }
    //回调
    if (responseCallback) {
        NSString* callbackId = [NSString stringWithFormat:@"objc_cb_%ld", ++_uniqueId]; //回调的ID唯一标识符
        self.responseCallbacks[callbackId] = [responseCallback copy];
        message[@"callbackId"] = callbackId;
    }
    //handler的名称
    if (handlerName) {
        message[@"handlerName"] = handlerName;
    }
    [self _queueMessage:message];
}
```

该方法先将传进来的参数进行处理构造成一条消息(message), 消息可以理解成是一个JSON:

```
{
	"data": data /* 必选，发送到JavaScript端的数据 */
	"callbackId": "objc_cb_xxxx", /* 可选，如果有回调responseCallback会全局创建一个唯一的回调ID，并在responseCallbacks记下来对应的关系*/
	"handlerName": "handlerName" /* 可选，handler的名称*/
}
```

消息结构构造完成之后就调用`_queueMessage`方法

```
//WVJBMessage => typedef NSDictionary WVJBMessage;
- (void)_queueMessage:(WVJBMessage *)message {
    //startupMessageQueue启动bridge的时候的初始化的消息队列，默认bridge启动完成后会自动调用并清空
    if (self.startupMessageQueue) {
        [self.startupMessageQueue addObject:message];
    } 
    else {
        [self _dispatchMessage:message];
    }
}
```

先跳过`startupMessageQueue`直接看`_dispatchMessage`, 该方法就是处理Native发往JavaScript端的消息的具体实现：

```
- (void)_dispatchMessage:(WVJBMessage *)message {
    //将NSDictionary转化为JSON字符串, 并转义一些特殊的字符
    NSString *messageJSON = [self _serializeMessage:message pretty:NO];
    [self _log:@"SEND" json:messageJSON];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\\" withString:@"\\\\"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\"" withString:@"\\\""];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\'" withString:@"\\\'"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\n" withString:@"\\n"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\r" withString:@"\\r"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\f" withString:@"\\f"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\u2028" withString:@"\\u2028"]; //行分隔符
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\u2029" withString:@"\\u2029"]; //段落分隔符
    //拼接要调用的JavaScript代码，参数是JSON字符串
    NSString* javascriptCommand = [NSString stringWithFormat:@"WebViewJavascriptBridge._handleMessageFromObjC('%@');", messageJSON];
    //切换到主线程执行JS代码
    if ([[NSThread currentThread] isMainThread]) {
        [self _evaluateJavascript:javascriptCommand];

    } 
    else {
        dispatch_sync(dispatch_get_main_queue(), ^{
            [self _evaluateJavascript:javascriptCommand];
        });
    }
}

// 将NSDictionary转化为JSON字符串, 是否是展开的格式
- (NSString *)_serializeMessage:(id)message pretty:(BOOL)pretty {
    return [[NSString alloc] initWithData:[NSJSONSerialization dataWithJSONObject:message options:(NSJSONWritingOptions)(pretty ? NSJSONWritingPrettyPrinted : 0) error:nil] encoding:NSUTF8StringEncoding];
}

// 执行JS代码， 其实最终还是调用的是`stringByEvaluatingJavaScriptFromString`
- (void) _evaluateJavascript:(NSString *)javascriptCommand {
    //self.delegate => WebViewJavascriptBridge对象的实例
    [self.delegate _evaluateJavascript:javascriptCommand];
}

- (NSString*) _evaluateJavascript:(NSString*)javascriptCommand {
    return [_webView stringByEvaluatingJavaScriptFromString:javascriptCommand];
}
```

到这里Native发送消息到JavaScript了，那么接下来看下JavaScript是如何接收消息会响应的，首先从上面的代码可以看出来发送消息其实就是在WebView上面调用`WebViewJavascriptBridge._handleMessageFromObjC`这个JS的方法，参数是一个JSON字符串。

```
function _handleMessageFromObjC(messageJSON) {
	_dispatchMessageFromObjC(messageJSON);
}
```

`_handleMessageFromObjC`在JS端的实现其实最终调用的是`_dispatchMessageFromObjC`这个函数：

```
function _dispatchMessageFromObjC(messageJSON) {
    if (dispatchMessagesWithTimeoutSafety) {
        setTimeout(_doDispatchMessageFromObjC);
    } 
    else {
        _doDispatchMessageFromObjC();
    }
		
	function _doDispatchMessageFromObjC() {
		 // 解析消息的JSON字符串
        var message = JSON.parse(messageJSON);
        var messageHandler;
        var responseCallback;
        
        if (message.responseId) {
        	  // 对于JavaScript端的消息，判断是否有responseId这个字段，表示需要进行回调处理
            responseCallback = responseCallbacks[message.responseId];
            if (!responseCallback) {
                return;
            }
            responseCallback(message.responseData);
            delete responseCallbacks[message.responseId];
        } 
        else {
        	  // 对于Native端来的消息，判断是否有callbackId这个字段，表示需要进行响应
            if (message.callbackId) {
                var callbackResponseId = message.callbackId;
                responseCallback = function(responseData) {
                    _doSend({handlerName: message.handlerName, responseId: callbackResponseId, responseData: responseData});
                };
            }
			  
			  // 执行对应的Handler, 这个就是之前JS端注册的Handler的名称，从全局的配置里面获取到对应的函数实现
            var handler = messageHandlers[message.handlerName];
            if (!handler) {
                console.log("WebViewJavascriptBridge: WARNING: no handler for message from ObjC:", message);
            } 
            else {
            	   // 这里的handler其实就是前面注册的handler，执行完成之后会执行responseCallback
                handler(message.data, responseCallback);
            }
        }
    }
}
```

`responseCallback`里面最终会调用一个`_doSend`方法, 该方法会把消息加入到消息发送队列里面，然后通过修改iframe的src属性来达到向Native端发送消息的目的。

```
function _doSend(message, responseCallback) {
    if (responseCallback) {
        var callbackId = 'cb_'+(uniqueId++)+'_'+new Date().getTime();
        responseCallbacks[callbackId] = responseCallback;
        message['callbackId'] = callbackId;
    }
    sendMessageQueue.push(message);
    messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://' + QUEUE_HAS_MESSAGE;
}
```

#### 4、JavaScript调用Native的Handler

JavaScript调用Native的Handler就相比上面的简单一些了，就是直接调用`_doSend`方法，根据是否有回调函数然后执行完成之后就从全局的回调里面拿出来执行。

```
function callHandler(handlerName, data, responseCallback) {
    if (arguments.length == 2 && typeof data == 'function') {
        responseCallback = data;
        data = null;
    }
    _doSend({ handlerName:handlerName, data:data }, responseCallback);
}
```

### 核心逻辑

上面只介绍了怎么发送和怎么接收，其实最终主要是如下两个方面的实现：

#### 1、UIWebView -> Native

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    if (webView != _webView) { return YES; }
    NSURL *url = [request URL];
    __strong id<UIWebViewDelegate> strongDelegate = _webViewDelegate;
    if ([_base isWebViewJavascriptBridgeURL:url]) {
        if ([_base isBridgeLoadedURL:url]) {
            //__bridge_loaded__:// 注入JS代码
            [_base injectJavascriptFile];
        }
        else if ([_base isQueueMessageURL:url]) {
            //__wvjb_queue_message__:// 消息处理
            NSString *messageQueueString = [self _evaluateJavascript:[_base webViewJavascriptFetchQueyCommand]];
            [_base flushMessageQueue:messageQueueString];
        }
        else {
            //未知的消息类型
            [_base logUnkownMessage:url];
        }
        return NO;
    }
    else if (strongDelegate && [strongDelegate respondsToSelector:@selector(webView:shouldStartLoadWithRequest:navigationType:)]) {
        return [strongDelegate webView:webView shouldStartLoadWithRequest:request navigationType:navigationType];
    }
    else {
        return YES;
    }
}
```

初始化的代码如下：

```
- (void)injectJavascriptFile {
    NSString *js = WebViewJavascriptBridge_js();
    [self _evaluateJavascript:js];
    if (self.startupMessageQueue) {
        NSArray* queue = self.startupMessageQueue;
        self.startupMessageQueue = nil;
        for (id queuedMessage in queue) {
            [self _dispatchMessage:queuedMessage];
        }
    }
}
```

#### 2、Native -> UIWebView

```
- (NSString*) _evaluateJavascript:(NSString *)javascriptCommand {
    return [_webView stringByEvaluatingJavaScriptFromString:javascriptCommand];
}
```

### 参考资料

1、[WebViewJavascriptBridge主页](https://github.com/marcuswestin/WebViewJavascriptBridge)

2、[WebViewJavascriptBridge浅析](https://www.cnblogs.com/LeeGof/p/8143408.html)