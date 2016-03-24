---
layout: post
title: "iOS多线程之NSThread"
date: 2014-07-27 01:23:37 +0800
comments: true
tags: iOS
---
   
   iOS创建线程的方式有三种，分别是`NSThread`、`NSOperation`和`GCD`。这样三种编程方式从上到下，抽象度层次是由低到高，抽象度越高的使用越简单，也是Apple最推荐使用的。这里主要是介绍NSThread的相关使用要点，后续会继续介绍`NSOperation`和`GCD`的使用方法。 

##创建线程
   对于多线程的开发，iOS系统提供了多种不同的接口，先谈谈iOS多线程最基础方面的使用。产生线程的方式姑且分两类，一类是显式调用，另一类是隐式调用。

###显式创建线程


(1)采用NSThread的`detachNewThreadSelector:toTarget:withObject:`的类方法来创建多线程

```
[NSThread detachNewThreadSelector:@selector(doSomethingInBackground:) toTarget:self withObject:nil];
```
参数意义：
(1)selector:线程执行的方法，该selector只能有一个参数，而且返回值是`void`
(2)target:selector消息发送的对象
(3)argument:传递给target的唯一参数，可以为nil

(2)采用NSThread的`initWithTarget:selector:object:`的实例方法来创建线程，可以获取线程对象方便日后终止线程

```
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(doSomethingInBackground:) object:nil];
```

(3)继承NSThread类，实现`main`方法，然后创建MyThread的对象调用`start`的方法调用线程

```
- (void)main
{
    //do something in background
}
```

调用

```
MyThread *thread = [[MyThread alloc] init];
[thread start];
```

###隐式创建线程
（1）在后台执行

```
- (void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg
```

（2）在指定的线程中执行

```
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait
```

（3）在主线程中执行，wait表示是否阻塞方法的调用，如果为YES表示等待主线程中运行方法结束，可用于在子线程中刷新UI

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait
```

##NSThread的其他方法
创建的线程是非关联线程（detached thread），即父线程和子线程没有执行依赖关系，父线程结束并不意味子线程结束。

(1)获得当前线程

```
//获得当前线程
 + (NSThread *)currentThread; 
```

(2)让线程休眠指定的时间间隔

```
//线程休眠
+ (void)sleepForTimeInterval:(NSTimeInterval)ti; 
```

(3)获取主线程（UI线程）

```
//主线程，亦即UI线程
+ (NSThread *)mainThread; 
```

(4)当前线程是否是主线程

```
//当前线程是否主线程
- (BOOL)isMainThread;
+ (BOOL)isMainThread; 
```

(4)线程是否正在运行

```
//线程是否正在运行
- (BOOL)isExecuting; 
```

(5)线程是否结束

```
//线程是否已结束
- (BOOL)isFinished; 
```

##非线程调用（NSObject的Category方法）

即在当前线程执行，注意它们会阻塞当前线程（包括UI线程）：

```
- (id)performSelector:(SEL)aSelector;
```

```
- (id)performSelector:(SEL)aSelector withObject:(id)object;
```

```
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```

 以下调用在当前线程延迟执行，如果当前线程没有显式使用NSRunLoop或已退出就无法执行了，需要注意这点：
 
```
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray *)modes;
```

```
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;
```

而且它们可以被终止：

```
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(id)anArgument;
```

```
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget;
```

##线程执行顺序

通常UI需要显示网络数据时，可以简单地利用线程的执行顺序，避免显式的线程同步：

（1）UI线程调用

```
[threadObj performSelectorInBackground:@selector(loadData) withObject:nil];
```

（2）子线程中回调UI线程来更新UI

```
- (void)loadData
{
    //query data from network
    //update data model
    //callback UI thread
    [uiObj performSelectorOnMainThread:@selector(updateUI) withObject:nil waitUntilDone:YES];
}
```
也可以使用NSThread实现同样的功能，loadData相当于NSThread的main方法。

##参考资料

1、[《iOS多线程编程之NSThread的使用》](http://blog.csdn.net/totogo2010/article/details/8010231)

2、[《iOS多线程的初步研究（一）-- NSThread》](http://www.cnblogs.com/sunfrog/p/3243230.html)