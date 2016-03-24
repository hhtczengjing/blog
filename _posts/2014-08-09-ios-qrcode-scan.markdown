---
layout: post
title: "iOS中的二维码扫描"
date: 2014-08-09 17:44:36 +0800
comments: true
tags: iOS
---

二维码（Two-dimensional code），又称二维条码，它是用特定的几何图形按一定规律在平面（二维方向）上分布的黑白相间的图形，是所有信息数据的一把钥匙。在现代商业活动中，可实现的应用十分广泛，如：产品防伪/溯源、广告推送、网站链接、数据下载、商品交易、定位/导航、电子凭证、车辆管理、信息传递、名片交流、wifi共享等。如今智能手机扫一扫功能的应用使得二维码更加普遍。

##ZXing
   [ZXing](https://github.com/zxing/zxing)是一个开源的条码生成和扫描库，支持众多的条码格式，而且有各种语言的实现版本，支持的语言包括Java、C++、C#、Objective-C等，但是目前已经不再维护。
  
1、下载ZXing源码，对源码工程进行处理
   首先去Google Code或Github将ZXing的代码下载下来，整个工程比较大，我们只需要其中涉及iOS的部分，所以最好做一些裁剪。简单来说，我们只需要保留`cpp`和`iphone`这两个文件夹，其余的文件和文件夹全部删掉。接着继续裁剪，对于cpp这个目录，只保留`cpp/core/src/zxing`和`cpp/core/src/bigint`下面的内容，其余内容也可以删掉了。但是整个目录结构必须保持原样。 完成之后如下图所示：
    
![ios_zxing_dir.png](/images/ios_qrcode/ios_zxing_dir.png)

2、配置Xcode
  
  (1)把裁剪后的`zxing`目录整个移动到iOS项目的目录下,把`ZXingWidget.xcodeproj`文件拖动到iOS工程中，如下图所示。
  
  ![zxing_xcode_profile.png](http://blog.devzeng.com/images/ios_qrcode/zxing_xcode_profile.png)
  
  (2)在Xcode的`Target`->`Build Phases`中添加如下配置：
  
   1)在`Target Dependencies`中添加`ZXingWidget(ZXingWidget)`
  
   2)在`Link Binary With Libraries`中添加`AVFoundation`、`AudioToolbox`、`CoreVideo`、`CoreMedia`、`libiconv`、`AddressBook`、`AddressBookUI`和`libZXingWidget.a`
  
  (3)在在Xcode的`Target`->`Build Settings`中添加如下配置：
  
   1)在Target`ZXingWidget`的`Build Settings`中调整`Build Active Architecture Only`为`NO`
   
   2)在主工程的Target的`Build Settings`的`Header Search Paths`添加下面两行：
   
   `./zxing/iphone/ZXingWidget/Classes`【recursive】
   
   `./zxing/cpp/core/src` 【non-recursive】

3、实现二维码的扫描

  将引用到ZXing的类的后缀名改成`.mm`

(1)直接使用ZXingWidgetController扫描二维码

```
- (void)onZxingClicked:(id)sender
{
    ZXingWidgetController *widController = [[ZXingWidgetController alloc] initWithDelegate:self showCancel:YES OneDMode:NO];
    NSMutableSet *readers = [[NSMutableSet alloc] init];
    QRCodeReader *qrcodeReader = [[QRCodeReader alloc] init];
    [readers addObject:qrcodeReader];
    widController.readers = readers;
    [self presentViewController:widController animated:YES completion:nil];
}
```

实现ZXingDelegate协议的代理方法

```
//扫描出结果调用的函数
- (void)zxingController:(ZXingWidgetController*)c didScanResult:(NSString *)result
{
    [c dismissViewControllerAnimated:YES completion:nil];
    NSLog(@"%@", result);
}

//取消处理回调的函数
- (void)zxingControllerDidCancel:(ZXingWidgetController*)controller
{
    [controller dismissViewControllerAnimated:YES completion:nil];
}
```

(2)使用AVFoundation获取Camera的实时图像然后对图像进行解析

1)获取Camera的实时图像

 获取Camera的实时图像的方法过长，会在接下来的博文中进行介绍。

2)解析图像的内容

```
//解析图像的内容
- (void)decoderQRCode:(UIImage *)image
{
    NSMutableSet *qrReaderConfig = [[NSMutableSet alloc ] init];
    QRCodeReader* qrcodeReader = [[QRCodeReader alloc] init];
    [qrReaderConfig addObject:qrcodeReader];
    Decoder *d = [[Decoder alloc] init];
    d.readers = qrReaderConfig;
    d.delegate = self;
    [d decodeImage:image];
}
```

```
//开始解析图片
- (void)decoder:(Decoder *)decoder willDecodeImage:(UIImage *)image usingSubset:(UIImage *)subset
{
    
}

//已经完成解析
- (void)decoder:(Decoder *)decoder didDecodeImage:(UIImage *)image usingSubset:(UIImage *)subset withResult:(TwoDDecoderResult *)result
{
    NSLog(@"【Decoder Result】->%@", result.text);
}

//解析失败
- (void)decoder:(Decoder *)decoder failedToDecodeImage:(UIImage *)image usingSubset:(UIImage *)subset reason:(NSString *)reason
{
    NSLog(@"【Decoder Error】->%@", reason);
}

- (void)decoder:(Decoder *)decoder foundPossibleResultPoint:(CGPoint)point
{
    
}
```

##iOS7中自带的AVFoundation

1、创建AVCaptureSession对象

```
- (AVCaptureSession *)setupCaptureSession
{
    NSError *error;
    AVCaptureDevice *captureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    
    AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:captureDevice error:&error];
    
    if (!input) {
        NSLog(@"%@", [error localizedDescription]);
        return NO;
    }
    
    AVCaptureSession *session = [[AVCaptureSession alloc] init];
    // Set the input device on the capture session.
    [session addInput:input];
    
    AVCaptureMetadataOutput *captureMetadataOutput = [[AVCaptureMetadataOutput alloc] init];
    [session addOutput:captureMetadataOutput];
    
    // Create a new serial dispatch queue.
    dispatch_queue_t dispatchQueue;
    dispatchQueue = dispatch_queue_create("com.devzeng.qrcode.queue", NULL);
    [captureMetadataOutput setMetadataObjectsDelegate:self queue:dispatchQueue];
    
    [captureMetadataOutput setMetadataObjectTypes:[NSArray arrayWithObject:AVMetadataObjectTypeQRCode]];
    //[captureMetadataOutput setMetadataObjectTypes:[NSArray arrayWithObjects:AVMetadataObjectTypeEAN13Code, AVMetadataObjectTypeEAN8Code, AVMetadataObjectTypeCode128Code, AVMetadataObjectTypeQRCode, nil]];
    
    [session startRunning];
    
    return session;
}
```

2、实现`AVCaptureMetadataOutputObjectsDelegate`代理方法

```
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection
{
    if(!self.isReading)
    {
        return;
    }
    
    if (metadataObjects != nil && [metadataObjects count] > 0)
    {
        AVMetadataMachineReadableCodeObject *metadataObj = [metadataObjects objectAtIndex:0];
        NSLog(@"%@", metadataObj.stringValue);
    }
}
```

##参考资料

1、[《在iOS中使用ZXing库》](http://www.devtang.com/blog/2012/12/23/use-zxing-library/)

2、[《iOS项目开发中使用了ZXing》](http://chenweihuacwh.iteye.com/blog/1918325)

3、[《在iOS和Android中使用二维码ZXing库及常见问题解决和整合后的代码》](http://thierry-xing.iteye.com/blog/1815295)

4、[《Undefined symbols for architecture armv7 when using ZXing library in XCode 4.5》](http://stackoverflow.com/questions/12968369/undefined-symbols-for-architecture-armv7-when-using-zxing-library-in-xcode-4-5)

5、[《解决Xcode5.1编译ZXing出错的问题》](http://blog.csdn.net/sing_sing/article/details/21512941)
