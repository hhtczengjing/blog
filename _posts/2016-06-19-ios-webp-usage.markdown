---
layout: post
title: "在iOS项目中使用WebP格式图片"
date: 2016-06-19 11:40
comments: true
tags: iOS
---

WebP是Google开发的一种旨在加快图片加载速度的图片格式。图片压缩体积大约只有JPEG的2/3，并能节省大量的服务器带宽资源和数据空间。Facebook Ebay等知名网站已经开始测试并使用WebP格式。下图是Google已经和正在部署的WebP的产品。

![google-webp-product.jpg](/images/ios-webp-usage/google-webp-product.jpg)

与JPEG相同，WebP是一种有损压缩。Google表示这种格式的主要优势在于高效率。他们发现，“在质量相同的情况下，WebP格式图像的体积要比JPEG格式图像小40%，美中不足的是，WebP格式图像的编码时间“比JPEG格式图像长8倍”。目前Google Chrome浏览器已经支持webp格式，Opera在版本号Opera11.10后也增加了支持，然而火狐和ie暂时还不支持webp格式，可以采用flash插件来显示webp，当然这样会耗费一些性能。各个浏览器对WebP的格式支持如下所示：

![webp-support.png](/images/ios-webp-usage/webp-support.png)

关于WebP格式优势不在多说，推荐查看官方的一份研究报告。[点此查看](https://developers.google.com/speed/webp/docs/c_study#negative_compression_gain)

###WebP转换

Google提供了一套完整的工具集方便我们使用WebP。[点此下载](https://developers.google.com/speed/webp/docs/precompiled)

1、将普通图片格式转换为WebP格式

`cwebp input_file -o output_file.webp`

2、将WebP格式图片转换为普通图片格式

`dwebp input_file.webp -o output.png`

更多参数可以查看官方的文档，或者直接使用第三方的转换工具。

###在iOS项目中使用WebP

SDWebImage中支持WebP格式的，可以完成`UIImage -> Webp`和`WebP -> UIImage`的转换。直接通过CocoaPods的Podfile文件中加入`pod 'SDWebImage/WebP'`即可。

![sdwebimage-webp.png](/images/ios-webp-usage/sdwebimage-webp.png)

`SDWebImage/WebP`提供了`UIImage+WebP`的Category里面有个将WebP NSData的数据转换为UIImage的方法：

`+ (UIImage *)sd_imageWithWebPData:(NSData *)data;`

1.在Native中使用WebP格式图片

```
NSString *path = [[NSBundle mainBundle] pathForResource:@"logo" ofType:@"webp"];
NSData *data = [[NSData alloc] initWithContentsOfFile:path];
UIImage *img = [UIImage sd_imageWithWebPData:data];
self.imageView.image = img;
```

2.在UIWebView中使用WebP格式图片

推荐使用NSURLProtocol拦截UIWebVieW中的网络请求判断是否webp的请求如果是webp的话就将webp转码并缓存处理，具体的实现代码如下：

```
@interface MyWebPHandlerURLProtocol ()

@property (nonatomic, strong) NSURLConnection *connection;

@end

static NSString * const URLProtocolHandledKey = @"URLProtocolHandledKey";

@implementation MyWebPHandlerURLProtocol

#pragma mark - 初始化网络请求

+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
    //只处理http和https请求
    NSString *scheme = [[request URL] scheme];
    NSString *extension = [[request URL] pathExtension];
    if (([scheme caseInsensitiveCompare:@"http"] == NSOrderedSame || [scheme caseInsensitiveCompare:@"https"] == NSOrderedSame) && ([extension caseInsensitiveCompare:@"webp"] == NSOrderedSame)) {
        //看看是否已经处理过了，防止无限循环
        if ([NSURLProtocol propertyForKey:URLProtocolHandledKey inRequest:request]) {
            return NO;
        }
        return YES;
    }
    return NO;
}

+ (NSURLRequest *) canonicalRequestForRequest:(NSURLRequest *)request {
    return request;
}

- (void)startLoading {
    NSMutableURLRequest *mutableReqeust = [[self request] mutableCopy];
    //标示改request已经处理过了，防止无限循环
    [NSURLProtocol setProperty:@YES forKey:URLProtocolHandledKey inRequest:mutableReqeust];
    //判断是否缓存
    NSString *name = [NSString stringWithFormat:@"%@.jpg", [self.request.URL.absoluteString MD5Encode]];
    NSString *path = [NSTemporaryDirectory() stringByAppendingPathComponent:name];
    if([[NSFileManager defaultManager] fileExistsAtPath:path]) {
        mutableReqeust.URL = [NSURL fileURLWithPath:path];
        self.connection = [NSURLConnection connectionWithRequest:mutableReqeust delegate:self];
    }
    else {
        self.connection = [NSURLConnection connectionWithRequest:mutableReqeust delegate:self];
    }
}

- (void)stopLoading {
    [self.connection cancel];
    self.connection = nil;
}

#pragma mark - NSURLConnection Delegate Methods

- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
	NSString *extension = [connection.currentRequest.URL pathExtension];
    if ([extension caseInsensitiveCompare:@"webp"] == NSOrderedSame) {
        UIImage *imgData = [UIImage sd_imageWithWebPData:data];
        NSData *jpgData = UIImageJPEGRepresentation(imgData, 1.0f);
        NSString *name = [NSString stringWithFormat:@"%@.jpg", [connection.currentRequest.URL.absoluteString MD5Encode]];
        NSString *path = [NSTemporaryDirectory() stringByAppendingPathComponent:name];
        [jpgData writeToFile:path atomically:YES];
        [self.client URLProtocol:self didLoadData:jpgData];
    }
    else {
        [self.client URLProtocol:self didLoadData:data];
    }
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}

@end
```

使用MyWebPHandlerURLProtocol前需要注册，推荐在APPDelegate中的`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions;`的方法中加入如下代码：

`[NSURLProtocol registerClass:[MyWebPHandlerURLProtocol class]];`

这里的示例使用的是NSURLConnection，如果需要使用NSURLSession可按照需求提供。

###参考资料

1、[《iOS-WebP》](http://seanooi.github.io/iOS-WebP/)

2、[《webP 格式图片在 iOS 中的应用》](http://www.jianshu.com/p/ed7562a34af1)

3、[《WebP 探寻之路》](http://isux.tencent.com/introduction-of-webp.html)

4、[《WebP官网》](https://developers.google.com/speed/webp/)

5、[《15年双11手淘前端技术巡演 - H5性能最佳实践》](https://github.com/amfe/article/issues/21)

6、[《How To Reduce Image Size With WebP Automagically》](https://www.maxcdn.com/blog//how-to-reduce-image-size-with-webp-automagically/)

7、[《WebP Comparative Study》](https://developers.google.com/speed/webp/docs/c_study#negative_compression_gain)