---
layout: post
title: "iOS多线程之NSOperation和NSOperationQueue"
date: 2014-08-24 18:31:30 +0800
comments: true
tags: iOS
---

   如果对Java或者与Java类似的语言熟悉的话，可以说NSOperation对象很像java.lang.Runnable接口。类似的，在Java的Runnable接口中，NSOperation对象被设计为可以扩展的。还是和Java一样，这里也有一个方法可以被重载次数的最小值。对于NSOpetation来说，这个方法就是`-(void)main`方法.使用NSOperation的最简单的一种方法是把它加入一个NSOperationQueue。一旦operation被加入了这个队列，队列就马上把它踢出开始运行。operation运行结束后，队列就马上将它释放。

##NSOperation

1、NSOperation

NSOperation实例封装了需要执行的操作和执行操作所需的数据，并且能够以并发或者非并发的方式执行这个操作，NSOperation本身是抽象基类，因此必须使用它的子类。NSoperation

(1)执行操作

NSOperation调用start方法即可开始执行操作，NSOperation对象默认按同步方式执行，也就是在调用start方法所处的线程中直接执行。

`[operation start]`

NSOperation对象的`- (BOOL)isConcurrent;`方法会返回当前操作相对于调用start方法的线程是同步还是异步执行，默认返回是NO，表示操作与调用线程同步执行。

`[operation isConcurrent]`

(2)取消操作

operation开始执行操作之后，默认会一直执行操作直到完成，可以调用cancel方法中途取消操作。

`operation cancel`

(3)执行完毕监听处理

如果想在一个NSOperation操作执行完毕之后做一些事情，可以调用NSOperation的setCompletionBlock方法来设置想要做的事情，代码如下：

```
[operation1 setCompletionBlock:^{
	//执行完毕处理  
}];
```

或者

```
operation.completionBlock = ^{
	//执行完毕处理
};
```

(4)重载main方法

```
- (void)main
{
	//操作执行的内容
}
```

(5)最佳实践

operation开始执行之后,会一直执行任务直到完成,或者显式地取消操作。取消可能发生在任何时候,甚至在operation执行之前。尽管NSOperation提供了一个方法,让应用取消一个操作,但是识别出取消事件则是我们自己的事情。如果operation直接终止, 可能无法回收所有已分配的内存或资源，因此operation对象需要检测取消事件,并优雅地退出执行NSOperation对象需要定期地调用isCancelled方法检测操作是否已经被取消,如果返回YES(表示已取消),则立即退出执行。不管是自定义NSOperation子类,还是使用系统提供的两个具体子类,都需要支持取消。isCancelled方法本身非常轻量,可以频繁地调用而不产生大的性能损失。以下地方可能需要调用isCancelled:

- 在执行任何实际的工作之前

- 在循环的每次迭代过程中,如果每个迭代相对较长可能需要调用多次

- 代码中相对比较容易中止操作的任何地方

实例代码：

```
- (void)main
{
    @autoreleasepool {
        if(self.isCancelled)
        {
            return;
        }
        //TODO:获取数据
        if(self.isCancelled)
        {
            //释放资源
            return;
        }
        //TODO:处理数据
        if(self.isCancelled)
        {
            //释放资源
            return;
        }
        //TODO:通知主线程处理
    }
}
```

2、NSInvocationOperation

```
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(runInBackground) object:nil];
[operation start];
```

3、NSBlockOperation

(1)使用NSBlockOperation执行一个操作

```
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
	//执行的操作
}];
[operation start];
```
上面的Block是同步执行的。

(2)使用addExecutionBlock方法关联多个操作

```
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
	//执行的操作1
}];
[operation addExecutionBlock:^{
   //执行的操作2
}];
[operation addExecutionBlock:^{
   //执行的操作3
}];
[operation start];
```
上面的几个Block是并发执行的，也就是在不同的线程中执行的。

##NSOperationQueue

一个NSOperation对象可以通过调用start方法来执行任务，默认是同步执行的。也可以将NSOperation添加到一个NSOperationQueue(操作队列)中去执行，而且是异步执行的。

1、创建一个操作队列

`NSOperationQueue *queue = [[NSOperationQueue alloc] init];`

2、添加NSOperation到NSOperationQueue中

(1)添加一个NSOperation

`[queue addOperation:operation]`

(2)添加一组NSOperation

`[queue addOperations:operations waitUntilFinished:NO]`

(3)添加一个block形式的Operation

```
[queue addOperationWithBlock:^{
	//执行一个Block的操作        
}];
```

3、设置队列的最大并发操作数量

队列的最大并发操作数量，意思是队列中最多同时运行几条线程。虽然NSOperationQueue类设计用于并发执行Operations,你也可以强制单个queue一次只能执行一个Operation。`setMaxConcurrentOperationCount:`方法可以配置queue的最大并发操作数量。设为1就表示queue每次只能执行一个操作。不过operation执行的顺序仍然依赖于其它因素,比如operation是否准备好和operation的优先级等。

控制每次只能执行一个操作，可以这样写：

`queue.maxConcurrentOperationCount = 1;`

或者

`[queue setMaxConcurrentOperationCount:1];`


4、取消operations

(1)单个NSOperation取消

`[operation cancel]`

(2)取消NSOperationQueue中的所有操作

`[queue cancelAllOperations]`

##参考资料

1、[《NSOperation and NSOperationQueue》](http://www.cnblogs.com/zhidao-chen/archive/2012/07/06/2579152.html)

2、[《iOS多线程编程之NSOperation和NSOperationQueue的使用》](http://blog.csdn.net/totogo2010/article/details/8013316)

3、[《多线程编程2 - NSOperation》](http://blog.csdn.net/q199109106q/article/details/8565923)

4、[《多线程编程3 - NSOperationQueue》](http://blog.csdn.net/q199109106q/article/details/8566222)
