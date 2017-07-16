---
layout: post
title: "iOS中使用Tesseract提取身份证号码"
date: 2017-07-15 11:08
comments: true
tags: iOS
---

`OCR` （`Optical Character Recognition`，光学字符识别）是指电子设备（例如扫描仪或数码相机）检查纸上打印的字符，通过检测暗、亮的模式确定其形状，然后用字符识别方法将形状翻译成计算机文字的过程。通俗来说就是通过对图像进行处理提取裁剪出来有字符的区域然后对字符进行识别翻译成文字。

![how-ocr.png](/images/ios-tesseract-ocr/how-ocr.png)

上面的图片是来自于[Baidu](https://cloud.baidu.com/product/ocr/idcard)的在线OCR识别。本文是基于[tesseract-ocr](https://github.com/tesseract-ocr)（Tesseract是一个开源的OCR引擎，可以识别多种格式的图像文件并将其转换成文本，目前已支持包括中文在内的60多种语言。）和 [OpenCV](http://www.opencv.org/releases.html)（OpenCV是一个开源的跨平台计算机视觉库）进行开发的。

### 环境准备

#### 1、配置Cocoapods Podfile

推荐使用Cocoapods的方式进行集成，在Podfile中添加如下两个库：

```
pod 'OpenCV', '~> 3.2.0'
pod 'TesseractOCRiOS', '~> 4.0.0'
```

如果下载OpenCV的库失败，可以手动的方式进行集成。到[官网](http://www.opencv.org/releases.html)下载最新版本的`OpenCV iOS Framework`(当前最新的版本是3.2.0）直接拖到项目里面。

![opencv-download.png](/images/ios-tesseract-ocr/opencv-download.png)

#### 2、下载tesseract的训练库

```
git clone https://github.com/tesseract-ocr/tessdata.git
```

下载各个语言包训练库完成之后，需要切换到`3.04.00`版本。

```
git checkout 3.04.00
```

由于本次只需要识别身份证号码使用英语的语言包训练库就可以了，删除其他的只保留`eng.traineddata`既可。然后以`Create folder reference`的方式拖到项目中（蓝色group），如下图所示。

![tessdata-group.png](/images/ios-tesseract-ocr/tessdata-group.png)

### 操作步骤

#### 1、图像处理

##### (1) 转化为灰度图像

```
cv::cvtColor(src, dest, cv::COLOR_BGR2GRAY);
```

##### (2) 二值化

```
cv::threshold(src, dest, 100, 255, CV_THRESH_BINARY);
```

##### (3) 图像腐蚀填充

将规范化的2值图像进行，因为之前进行了规范化，因此这里膨胀的幅度可以设为定值；（膨胀就是将黑点扩大范围，因此有字迹的地方将会连成一片，形成很多的contours）

```
cv::Mat erodeElement = getStructuringElement(cv::MORPH_RECT, cv::Size(26, 26));
cv::erode(src, dest, erodeElement);
```

##### (4) 轮廓检测

图片经过腐蚀操作后相邻点会连接在一起形成一个大的区域，这个时候通过轮廊检测就可以把每个大的区域找出来，这样就可以定位到身份证上面号码的区域。使用`findContours`方法可以找出其中所有的轮廓(contours),将返回一个列表，得到每个人contour的位置。函数原型如下：

```
CV_EXPORTS void findContours(InputOutputArray image, OutputArrayOfArrays contours, int mode, int method, Point offset = Point());
```

参数说明：

- image ：要寻找轮廓的图片，注意这里的轮廓会直接改变在src上(需要备份)；
- contours：输出检测到的轮廓
- mode：轮廓检索模式。CV_RETR_TREE：检索所有的轮廓
- method: 轮廓近似方法。CV_CHAIN_APPROX_SIMPLE ：表示去掉冗余信息
- offset: 搜索的偏移

示例代码如下：

```
std::vector<std::vector<cv::Point>> contours;//定义一个容器来存储所有检测到的轮廊
cv::findContours(src, contours, CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, cvPoint(0, 0));
```

##### (5) 身份证号码提取 

由于身份证号码所在位置固定，拍照方式合适，则可以根据contour的位置和其本身size，找到包含身份证号码的contour。然后将这一片从之前的二值化处理后的图像里分割出来，单独处理。

```
std::vector<cv::Rect> rects;
cv::Rect numberRect = cv::Rect(0,0,0,0);
std::vector<std::vector<cv::Point>>::const_iterator itContours = contours.begin();
for ( ; itContours != contours.end(); ++itContours) {
    cv::Rect rect = cv::boundingRect(*itContours);
    rects.push_back(rect);
    //算法原理: 宽度/高度 > 5
    if (rect.width > numberRect.width && rect.width > rect.height * 5) {
            numberRect = rect;
        }
    }
```

#### 2、信息提取

```
- (void)pd_recognizeImageWithTesseract:(UIImage *)image complete:(void (^)(BOOL status, NSString *result))complete {
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
        G8Tesseract *tesseract = [[G8Tesseract alloc] initWithLanguage:@"eng"];
        tesseract.image = [image g8_blackAndWhite];
        tesseract.charWhitelist = @"0123456789";
        BOOL status = [tesseract recognize];
        dispatch_async(dispatch_get_main_queue(), ^{
            complete(status, tesseract.recognizedText);
        });
    });
}
```

#### 3、说明：

- （1）测试素材可以从`https://cloud.baidu.com/product/ocr/idcard`获取
- （2）优化的方向有几个方面：
    - ① 调整提取号码区域的算法（腐蚀与填充、号码区域提取）
    - ② 手动进行样本训练

### 参考资料

1、[《OpenCV官网》](http://www.opencv.org/releases.html)

2、[《iOS身份证号码识别》](http://www.jianshu.com/p/ac4c4536ca3e)

3、[《tesseract在使用过程中的一些常见问题》](https://github.com/tesseract-ocr/tesseract/wiki/FAQ)

4、[《iOS实现身份证号码识别》](http://fengdeng.github.io/2016/08/18/iOS实现身份证号码识别/)

5、[《OpenCV学习开发笔记一(iOS9)》](http://www.jianshu.com/p/c544d62749ac)