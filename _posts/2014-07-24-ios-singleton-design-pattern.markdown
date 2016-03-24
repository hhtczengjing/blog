---
layout: post
title: "iOS中的单例设计模式"
date: 2014-07-24 23:16:28 +0800
comments: true
tags: iOS
---
  在一个iOS应用的生命周期中，有时候我们只需要某个类的一个实例。例如，iOS设备都有一个重力加速计硬件设备，要访问设备在X轴、Y轴和Z轴上的重力加速度，就必然有一个类能够与硬件设备沟通来实时获取这些数据，这个类就是UIAccelerometer。除了实时地获取数据该类还能保持X轴、Y轴和Z轴的状态，但这个类只要一个实例就够了，如果有多个实例就会占用过多的内存。

####单例模式简单实现


在objective-c中要实现一个单例类，至少需要做以下四个步骤：(推荐使用宏的方式来创建)

(1)为单例对象实现一个静态实例，并初始化，然后设置成nil:

```
static Singleton *_instance = nil;
```

(2)实现一个实例构造方法检查上面声明的静态实例是否为nil，如果是则新建并返回一个本类的实例;

```
@synchronized (self) {
	if(!_instance)
	{
   		_instance = [[Singleton alloc] init];
	}
}
```

(3)重写allocWithZone方法，用来保证直接使用alloc和init试图获得一个新实力的时候不产生一个新实例:

```
+ (id) allocWithZone:(NSZone *)zone
{
    @synchronized (self) {
        if (sharedObj == nil) {
            sharedObj = [super allocWithZone:zone];
            return sharedObj;
        }
    }
    return nil;
}
```

(4)适当实现allocWitheZone，copyWithZone，release和autorelease.

####GCD方式实现

自苹果引入了Grand Central Dispatch (GCD)（Mac OS 10.6和iOS4.0）后，创建单例又有了新的方法，那就是使用dispatch_once函数，具体创建代码如下：

```
+ (Singleton *)sharedInstance{
	static Singleton *_instance;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
   		_instance = [[Singleton alloc] init];
	});
	return _instance;
}
```


####单例模式的应用

(1)UIApplication获取AppDelegate

```
APPDelegate *delegate = [[UIApplication sharedApplication] delegate];
```

(2)NSUserDefaults

```
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
```

(3)NSNotificationCenter

```
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
```

(4)NSFileManager

```
NSFileManager *fm = [NSFileManager defaultFileManager];
```

(5)NSBundle

```
NSBundle *bundle = [NSBundle mainBundle];
```
 