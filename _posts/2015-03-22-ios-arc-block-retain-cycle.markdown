---
layout: post
title: "iOS中ARC下block的循环引用"
date: 2015-03-22 18:07:19 +0800
comments: true
tags: iOS
---

在iOS4.2时，Apple推出ARC的内存管理机制。这是一种编译期的内存管理方式，在编译期间，编译器会判断对象的使用情况，并适当的加上retain和release，使得对象的内存被合理的管理。所以，从本质上说ARC和MRC在本质上是一样的，都是通过引用计数的内存管理方式。

使用ARC虽然可以简化内存管理，但是ARC并不是万能的，有些情况程序为了能够正常运行，会隐式地持有或者复制对象，如果不加以注意，便会造成内存泄露。在ARC下，当block获取到外部变量时，由于编译器无法预测获取到的变量何时会被突然释放，为了保证程序能够正确运行，让block持有获取到的变量。

本文主要通过一个例子来介绍在ARC情况下使用Block不当会导致的内存泄露的问题。

###示例代码

示例代码来源于《Effective Objective-C 2.0》(编写高质量iOS与OS X代码的52个有效方法)。

（1）`EOCNetworkFetcher.h`

```
typedef void (^EOCNetworkFetcherCompletionHandler)(NSData *data);

@interface EOCNetworkFetcher : NSObject

@property (nonatomic, strong, readonly) NSURL *url;

- (id)initWithURL:(NSURL *)url;

- (void)startWithCompletionHandler:(EOCNetworkFetcherCompletionHandler)completion;

@end
```

（2）`EOCNetworkFetcher.m`

```
@interface EOCNetworkFetcher ()

@property (nonatomic, strong, readwrite) NSURL *url;
@property (nonatomic, copy) EOCNetworkFetcherCompletionHandler completionHandler;
@property (nonatomic, strong) NSData *downloadData;

@end

@implementation EOCNetworkFetcher

- (id)initWithURL:(NSURL *)url {
    if(self = [super init]) {
        _url = url;
    }
    return self;
}

- (void)startWithCompletionHandler:(EOCNetworkFetcherCompletionHandler)completion {
    self.completionHandler = completion;
    //开始网络请求
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        _downloadData = [[NSData alloc] initWithContentsOfURL:_url];
        dispatch_async(dispatch_get_main_queue(), ^{
        	 //网络请求完成
            [self p_requestCompleted];
        });
    });
}

- (void)p_requestCompleted {
    if(_completionHandler) {
        _completionHandler(_downloadData);
    }
}

@end
```

（3）`EOCClass.m`

```
@implementation EOCClass {
    EOCNetworkFetcher *_networkFetcher;
    NSData *_fetchedData;
}

- (void)downloadData {
    NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    _networkFetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
    [_networkFetcher startWithCompletionHandler:^(NSData *data) {
        _fetchedData = data;
    }];
}
@end
```

代码分析：

1、`completion handler`块因为要设置`_fetchedData`实例变量的值，所以它必须捕获self变量，也就是说handler块保留了EOCClass实例；

2、而`EOCClass`实例通过strong实例变量保留了`EOCNetworkFetcher`，最后EOCNetworkFetcher实例对象又保留了handler块。

引用关系如下下图所示：

![arc_block_retain_cycle.png](/images/arc_block_retain_cycle/arc_block_retain_cycle.png)

###解决办法

要想打破保留环

1、方法一：使用完EOCNetworkFetcher对象之后就没有必要在保留该对象了，在block里面将对象释放即可打破保留环。

```
- (void)downloadData {
    NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    _networkFetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
    [_networkFetcher startWithCompletionHandler:^(NSData *data) {
        _fetchedData = data;
        _networkFetcher = nil;//加上此行，此处是为了打破循环引用
    }];
}
```

2、方法二：上面的方法需要调用者自己来将对象手动设置为nil，对于使用者来说会造成很多困恼，如果忘记将对象设置为nil就会造成循环引用。在运行完completion handler之后将block释放即可。

```
- (void)p_requestCompleted {
    if(_completionHandler) {
        _completionHandler(_downloadData);
    }
    self.completionHandler = nil;//加上此行，此处是为了打破循环引用
}
```

3、方法三：将引用的一方变成weak，从而避免循环引用。

```
- (void)downloadData {
   __weak __typeof(self) weakSelf = self;
   NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
   _networkFetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
   [_networkFetcher startWithCompletionHandler:^(NSData *data) {
   		//如果想防止 weakSelf 被释放，可以再次强引用
    	__typeof(&*weakSelf) strongSelf = weakSelf;
    	if (strongSelf) {
    		strongSelf.fetchedData = data;
    	}
   }];
}
```

###参考资料

1、[《ARC 下内存泄露的那些点》](https://www.zybuluo.com/MicroCai/note/67734)

2、[《Weakself的一种写法》](http://rocry.com/2012/12/18/objective-c-type-of/)

3、[《[iOS]ARC下循环引用的问题》](http://blog.cnbang.net/tech/2085/)

4、《Effective Objective-C 2.0》(Tips:40)、《iOS开发进阶》（P190）