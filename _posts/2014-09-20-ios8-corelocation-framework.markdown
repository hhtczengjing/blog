---
layout: post
title: "iOS8中使用CoreLocation定位"
date: 2014-09-20 20:43:46 +0800
comments: true
tags: iOS
---

在iOS8中，苹果已经强制开发者在请求定位服务时获得用户的授权，此外iOS状态栏中还有指示图标，提示用户当前应用是否正在使用定位服务。另外在iOS8中，苹果进一步改善了定位服务，让开发者请求定位服务时需要向用户提供更多的透明。此外，iOS8中还支持让应用开发者调用全新的“访问监控”功能，当用户允许后应用才能获得更多的定位数据。

![ios8_location_alert.png](/images/ios8_location/ios8_location_alert.png)

##iOS8以前使用CoreLocation定位

1、首先定义一个全局的变量用来记录CLLocationManager对象，引入`CoreLocation.framework`使用`#import <CoreLocation/CoreLocation.h>`

```
@property (nonatomic, strong) CLLocationManager  *locationManager;
```

2、初始化CLLocationManager并开始定位

```
self.locationManager = [[CLLocationManager alloc]init];
_locationManager.delegate = self;
_locationManager.desiredAccuracy = kCLLocationAccuracyBest;
_locationManager.distanceFilter = 10;
[_locationManager startUpdatingLocation];
```

3、实现CLLocationManagerDelegate的代理方法

(1)获取到位置数据，返回的是一个CLLocation的数组，一般使用其中的一个

```
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
{
    CLLocation *currLocation = [locations lastObject];
    NSLog(@"经度=%f 纬度=%f 高度=%f", currLocation.coordinate.latitude, currLocation.coordinate.longitude, currLocation.altitude);
}
```

(2)获取用户位置数据失败的回调方法，在此通知用户

```
- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error
{
    if ([error code] == kCLErrorDenied)
    {
        //访问被拒绝
    }
    if ([error code] == kCLErrorLocationUnknown) {
        //无法获取位置信息
    }
}
```

4、在`viewWillDisappear`关闭定位

```
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    [_locationManager stopUpdatingLocation];
}
```

##iOS8中使用CoreLocation定位

1、在使用CoreLocation前需要调用如下函数【iOS8专用】：

iOS8对定位进行了一些修改，其中包括定位授权的方法，CLLocationManager增加了下面的两个方法：

（1）始终允许访问位置信息

`- (void)requestAlwaysAuthorization;`

（2）使用应用程序期间允许访问位置数据

`- (void)requestWhenInUseAuthorization;`

示例如下：

```
self.locationManager = [[CLLocationManager alloc]init];
_locationManager.delegate = self;
_locationManager.desiredAccuracy = kCLLocationAccuracyBest;
_locationManager.distanceFilter = 10;
[_locationManager requestAlwaysAuthorization];//添加这句
[_locationManager startUpdatingLocation];
```

2、在Info.plist文件中添加如下配置：

（1）`NSLocationAlwaysUsageDescription`

（2）`NSLocationWhenInUseUsageDescription`

这两个键的值就是授权alert的描述，示例配置如下[`勾选Show Raw Keys/Values后进行添加`]：

![ios8_location_info_plist.png](/images/ios8_location/ios8_location_info_plist.png)

##参考资料

1、[《迎接iOS8 - CoreLocation的变化》](http://my.oschina.net/non6/blog/289150)

2、[《IOS开发之Core Location》](http://www.cnblogs.com/ios8/archive/2012/07/30/2614523.html)

3、[《IOS8下的定位授权》](http://blog.csdn.net/moclin23/article/details/38990257)


