---
layout: post
title: "iOS中Cordova开发初探"
date: 2016-04-01 22:08
comments: true
tags: iOS
---

跨平台开发是我们长久以来都在关注的话题，前段时间写了一篇关于React Native的文章[《React Native开发初探》](http://blog.devzeng.com/blog/hello-react-native.html)。也在公司的项目中部分使用React Native，虽然React Native的出现带来了很多变革，但是光一个环境搭建就把很多人挡在了门外，另外React Native更新的频率过快差不多两周一个版本，所以目前还是处于观望阶段。

![cordova-logo](/images/hello-cordova-ios/cordova-logo.png)

Cordova是一个开源的移动应用开发框架，是Adobe贡献给Apache的开源项目，是从PhoneGap中抽出的核心代码，是驱动PhoneGap的核心引擎。Cordova架构如下所示：

![cordova_app_architecture.png](/images/hello-cordova-ios/cordova_app_architecture.png)

从上图可以看出Cordova是基于WebView的，主要是负责Native/OS和Web之间的交互。我们可以使用Cordova的插件机制对Cordova的功能进行扩展，本文主要是介绍Cordova的用法的插件的开发。

###在现有项目中集成Cordova

```
pod 'Cordova'
```

如果需要引入一些相关的插件，可以加入如下配置，下面的这些插件可以通过pod搜索到：

```
    pod 'CordovaPlugin-console'
    pod 'cordova-plugin-camera'
    pod 'cordova-plugin-contacts'
    pod 'cordova-plugin-device'
    pod 'cordova-plugin-device-orientation'
    pod 'cordova-plugin-device-motion'
    pod 'cordova-plugin-globalization'
    pod 'cordova-plugin-geolocation'
    pod 'cordova-plugin-file'
    pod 'cordova-plugin-media-capture'
    pod 'cordova-plugin-network-information'
    pod 'cordova-plugin-splashscreen'
    pod 'cordova-plugin-inappbrowser'
    pod 'cordova-plugin-file-transfer'
    pod 'cordova-plugin-statusbar'
    pod 'cordova-plugin-vibration'
```

上面的插件对应着架构图中右边的Plugins，具体的使用方式参考官方的示例。

###如何使用Cordova加载网页

建议先下载一份phonegap的示例文件，项目地址是`https://github.com/phonegap/phonegap-webview-ios`，可以通过pod查询到(在终端中执行`pod search phonegap`):

![phonegap-template.png](/images/hello-cordova-ios/phonegap-template.png)

本人的项目是基于上面的这个模板项目进行开发的，修改的步骤如下：

####（1）添加config.xml的配置文件

参见示例项目中的config.xml。

####（2）html中引入cordova js库文件

参见示例项目中的js引入。

####（3）index.js文件内容：

```
var app = {
    initialize: function () {
        this.bindEvents();
    },
    bindEvents: function () {
        document.addEventListener('deviceready', this.onDeviceReady, false);
    },
    onDeviceReady: function () {
        app.receivedEvent('deviceready');
        document.getElementById("button").onclick = app.testConnection;
    },
    receivedEvent: function (id) {
        console.log('Received Event: ' + id + ', DEVICE READY FIRED!!!');
    },
    testConnection: function () {
        var networkState = navigator.connection.type;
        var states = {};
        states[Connection.UNKNOWN] = 'Unknown connection';
        states[Connection.ETHERNET] = 'Ethernet connection';
        states[Connection.WIFI] = 'WiFi connection';
        states[Connection.CELL_2G] = 'Cell 2G connection';
        states[Connection.CELL_3G] = 'Cell 3G connection';
        states[Connection.CELL_4G] = 'Cell 4G connection';
        states[Connection.CELL] = 'Cell generic connection';
        states[Connection.NONE] = 'No network connection';
        console.log('Network Info plugin test - connection type ' + states[networkState]);
    }
};
app.initialize();
```

以上摘自示例项目中的index.js

####（4）使用CDVViewController

```
CDVViewController *viewController = [CDVViewController new];
viewController.wwwFolderName = @"myfolder";
viewController.startPage = @"mypage.html"
```

其中使用CDVViewController通常需要设置wwwFolderName的目录名称，和startPage首页的名称即可。

###Cordova自定义插件开发

####（1）创建插件的实现类需要继承CDVPlugin

*MyBrowserPlugin.h：*

```
@interface MyBrowserPlugin : CDVPlugin

- (void)open:(CDVInvokedUrlCommand *)command;

@end
```

*MyBrowserPlugin.m：*

```
- (void)open:(CDVInvokedUrlCommand *)command {
    CDVPluginResult* pluginResult = nil;
    if(!command.arguments || command.arguments.count <= 0) {
        pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_ERROR];
    }
    else {
        NSString* url = [command.arguments objectAtIndex:0];
        if (url != nil && url.length > 0) {
            //TODO:打开网页浏览器
            pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_OK];
        }
        else {
            pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_ERROR];
        }
    }
    [self.commandDelegate sendPluginResult:pluginResult callbackId:command.callbackId];
}
```

如果上面的操作需要耗时很长时间可以在后台运行，示例代码如下：

```
- (void)myPluginMethod:(CDVInvokedUrlCommand*)command {
    // Check command.arguments here.
    [self.commandDelegate runInBackground:^{
        NSString* payload = nil;
        // Some blocking logic...
        CDVPluginResult* pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_OK messageAsString:payload];
        // The sendPluginResult method is thread-safe.
        [self.commandDelegate sendPluginResult:pluginResult callbackId:command.callbackId];
    }];
}
```

####（2）实现插件JS端的代码

```
cordova.define("com.devzeng.cordova.mybrowser", function(require, exports, module){
    var exec = require('cordova/exec');
    module.exports = {
        open: function(url, callback) {
               exec(function(param){callback('success');}, function(){callback('error');}, "MyBrowserPlugin", "open", [url]);
        }
    };
});
```

其中上面的exec函数中的第四个参数open是指的MyBrowserPlugin的open方法。使用的时候直接下面的方式调用：

```
mybrowser.open('url地址', function(message){...});
```

####（3）在web端(cordova_plugin.js)配置插件

```
{
	"file": "plugins/cordova-plugin-mybrowser/www/plugin.js",
	"id": "com.devzeng.cordova.mybrowser",
	"pluginId": "cordova-plugin-mybrowser",
	"clobbers": [
		"mybrowser"
	]
}
```

说明：

1）file指的是插件js文件的路径，相对于cordova_plugin.js文件

2）id指的是上面定义的id字符串，pluginId可以设置为插件的名字

3）clobbers中设置的数据是用来自动注册到window中的对象，我们在使用的时候直接使用该命令即可。

####（4）在项目中(config.xml)配置插件

```
<feature name="MyBrowserPlugin">
	<param name="ios-package" value="MyBrowserPlugin" />
</feature>
```

说明：

1）feature中的name对应着前面`exec(success, failure, service, action, params)`中的service

2）param中的name在iOS中固定写成ios-package, 后面的value就对应着该插件的入口实现类

###参考资料

1、[《iOS Plugin Development Guide》](http://cordova.apache.org/docs/en/6.x/guide/platforms/ios/plugin.html)

2、[《Cordova 3.x 基础（12） -- Plugin开发》](http://rensanning.iteye.com/blog/2029362)

3、[《Cordova Documentation》](http://cordova.apache.org/docs/en/latest/guide/overview/)