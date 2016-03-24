---
layout: post
title: "iOS开发中的类簇设计模式"
date: 2014-09-21 00:11:49 +0800
comments: true
tags: iOS
---

在iOS开发中经常会使用NSArray、NSNumber这样的系统提供的类来组织我们的数据，就拿NSNumber来说，NSNumber有两个比较常用的类方法，如下所示的定义：

（1）用来把BOOL类型的数据包装成一个OC的对象：

`+ (NSNumber *)numberWithBool:(BOOL)value;`

（2）用来把int类型包装成一个OC对象：

`+ (NSNumber *)numberWithInt:(int)value;`

在Xcode中运行下面的代码片段：

```
NSNumber *boolNumber = [NSNumber numberWithBool:YES];
NSLog(@"%@", [[boolNumber class] description]);
```
执行上面的代码片段在Xcode的控制台会输出`__NSCFBoolean`

```
NSNumber *intNum = [NSNumber numberWithInt:1];
NSLog(@"%@", [[intNum class] description]);
```
执行上面的代码片段在Xcode的控制台会输出`__NSCFNumber`

`[instance class]`方法返回的当前的对象的类的名称，`[NSNumber numberWithInt:1]`和`[NSNumber numberWithBool:YES]`通过上面的验证可以看出明显不是同一个类，而且`__NSCFNumber`和`__NSCFBoolean`很明显是一个私有类。

上面出现的这种现象在Foundation Framework中是很常见的一种情况，我们称作为类簇设计模式。那为什么要这样做呢？以NSNumber为例，我们知道NSNumber可以存储很多类型的数据，如int、Float、Double等等，具体支持哪些数据类型可以到NSNumber的头文件中查看，具体支持的类型如下：

`char、unsigned char、short、unsigned short、int、unsigned int、long、unsigned long、long long、unsigned long long、float、double、BOOL、NSInteger、NSUInteger`

一般情况下实现类似的效果一种方式是把NSNumber作为基类，然后分别去实现各自的子类，如下图所示：

![iOS_Class_Cluster_1.png](/images/ios_class_cluster/iOS_Class_Cluster_1.png)

但是一旦需要实现的子类多起来之后就会发现这样需要继承的子类太多，比如如果要仿NSNumber需要写这样十多个类。

![iOS_Class_Cluster_2.png](/images/ios_class_cluster/iOS_Class_Cluster_2.png)

最好的就是把这些子类写成私有的类，所有对外都在NSNumber中调用即可，对于使用者来说就轻松很多。

##在iOS中的应用

现在很多应用需要同时兼容iOS6、iOS7目前还需要针对iOS8进行适配。适配iOS7不是简单的让APP在iOS7的系统上运行正常，而是需要按照苹果官方的推荐设计指南进行重新设计，这样就给开发者带来很多的工作量。对于iOS6之前应用的样式都差不多，而针对iOS7设计了新的样式，为了同时支持iOS6和iOS7，也在各个系统上显示的效果符合系统原生的风格，就可以采用类簇的设计进行设计。

1、定义一个名字为`DemoView`的类，继承`UIView`如下所示：

DemoView.h

```
@interface DemoView : UIView

@end
```

DemoView.m

```
@implementation DemoView

@end
```

2、在DemoView.m文件中创建三个私有的类，分别用来对三个系统进行单独处理：

```
////////////////iOS6适配代码////////////////////

@interface dz_DemoView_ios6 : DemoView

@end

@implementation dz_DemoView_ios6

- (void)drawRect: (CGRect)rect
{
    /* Custom iOS6 drawing code */
}

@end

////////////////iOS7适配代码////////////////////

@interface dz_DemoView_ios7 : DemoView

@end

@implementation dz_DemoView_ios7

- (void)drawRect: (CGRect)rect
{
    /* Custom iOS7 drawing code */
}

@end

////////////////iOS8适配代码////////////////////

@interface dz_DemoView_ios8 : DemoView

@end

@implementation dz_DemoView_ios8

- (void)drawRect: (CGRect)rect
{
    /* Custom iOS8 drawing code */
}

@end
```

3、在DemoView.m中重写alloc的方法：

```
+ (instancetype)alloc
{
    if([self class] == [DemoView class])
    {
        float v = [[[UIDevice currentDevice] systemVersion] floatValue];
        if(v > 7)
        {
            //iOS8
            return [dz_DemoView_ios8 alloc];
        }
        else if(v < 7 )
        {
            //iOS 6
            return [dz_DemoView_ios6 alloc];
        }
        else
        {
            //iOS 7
            return [dz_DemoView_ios7 alloc];
        }
    }
    else
    {
        return [super alloc];
    }
}
```

4、在需要使用该View的地方直接使用，调用者无需关注其他事情：

```
DemoView *demoView = [[DemoView alloc] init];
[self.view addSubView: demoView];
```

##参考资料

1、[《类簇在iOS开发中的应用》](http://limboy.me/ios/2014/01/04/class-cluster.html)

2、[《iOS 类簇(Class Cluster)使用心得》](http://blog.codingcoder.com/class-cluster/)

3、[《Concepts in Objective-C Programming》](https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html)

4、[《Strategies to support the new iOS7 UI》](http://www.noxeos.com/2013/06/18/strategies-support-ios7-ui/)