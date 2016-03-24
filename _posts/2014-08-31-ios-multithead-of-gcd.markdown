---
layout: post
title: "iOS多线程之GCD"
date: 2014-08-31 16:04:42 +0800
comments: true
tags: iOS
---

Grand Central Dispatch(简称GCD)是Apple在`Mac OS X 10.6`和`iOS4.0`中首次引入的一个多核并行运算的解决方案。GCD是基于C语言的，如果使用GCD完全由系统管理线程，开发者不需要编写线程代码，只需专注定义想要只需的任务，然后添加到适当的Dispatch Queue(调度队列)。

GCD是一个可用来替代`NSThread、NSOperation、NSOperataionQueue`等技术的一个非常强大的解决方案。GCD在工作时会自动利用更多的处理器核心以充分利用机器的可用资源。

##Dispatch Queue

GCD编程的核心就是dispatch队列，dispatch block的执行最终都会放进某个队列中去进行，它类似NSOperationQueue但更复杂也更强大，并且可以嵌套使用。所以说，结合block实现的GCD，把函数闭包（Closure）的特性发挥得淋漓尽致。

1、创建队列

(1)创建自定义串行队列

`dispatch_queue_t queue = dispatch_queue_create("com.devzeng.gcd.serial", DISPATCH_QUEUE_SERIAL);`
生成一个串行队列，队列中的block按照先进先出（FIFO）的顺序去执行，实际上为单线程执行。第一个参数是队列的名称，在调试程序时会非常有用，所有尽量不要重名了。

(2)创建自定义并发队列

`dispatch_queue_t queue = dispatch_queue_create("com.devzeng.gcd.concurrent", DISPATCH_QUEUE_CONCURRENT);`
生成一个并发执行队列，block被分发到多个线程去执行

(3)创建默认缺省的并发队列

`dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);` 

获得程序进程缺省产生的并发队列，可设定优先级来选择高、中、低三个优先级队列。由于是系统默认生成的，所以无法调用dispatch_resume()和dispatch_suspend()来控制执行继续或中断。需要注意的是，三个队列不代表三个线程，可能会有更多的线程。并发队列可以根据实际情况来自动产生合理的线程数，也可理解为dispatch队列实现了一个线程池的管理，对于程序逻辑是透明的。

官网文档解释说共有三个并发队列，但实际还有一个更低优先级的队列，设置优先级为DISPATCH_QUEUE_PRIORITY_BACKGROUND。Xcode调试时可以观察到正在使用的各个dispatch队列。

(4) 获取主线程的队列

`dispatch_queue_t queue = dispatch_get_main_queue();` 
获得主线程的dispatch队列，实际是一个串行队列。同样无法控制主线程dispatch队列的执行继续或中断。

2、使用队列

创建好队列后，使用`dispatch_async`和`dispatch_sync`这两个函数可以执行Block代码。Block函数不返回，一直等到block执行完毕。编译器会根据实际情况优化代码，所以有时候你会发现block其实还在当前线程上执行，并没用产生新线程。

(1)异步执行block，函数立即返回

```
dispatch_async(queue, ^{
　　//block具体代码
});
```

(2)同步执行block

```
dispatch_sync(queue, ^{
　　//block具体代码
});
```

(3)避免死锁

实际编程经验告诉我们，尽可能避免使用dispatch_sync，嵌套使用时还容易引起程序死锁。如果queue是一个串行队列的话，这段代码立即产生死锁：

```
dispatch_sync(queue, ^{
	dispatch_sync(queue, ^{
		//
	});
});
```
同样下面的代码也会出现死锁：

```
dispatch_sync(dispatch_get_main_queue(), ^{
　　//......
}); 
```

(4)最佳实践

1)网络请求数据处理示例：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
　　//子线程中开始网络请求数据
　　//更新数据模型
　　dispatch_sync(dispatch_get_main_queue(), ^{
　　　　//在主线程中更新UI代码
　　});
});
```

2)使用串行队列

dispatch队列是线程安全的，可以利用串行队列实现锁的功能。比如多线程写同一数据库，需要保持写入的顺序和每次写入的完整性，简单地利用串行队列即可实现：

`dispatch_queue_t queue1 = dispatch_queue_create("com.devzeng.dispatch.writedb", DISPATCH_QUEUE_SERIAL);
`

```
- (void)writeDB:(NSData *)data
{
　　dispatch_async(queue1, ^{
　　　　//write database
　　});
} 
```
下一次调用`writeDB:`必须等到上次调用完成后才能进行，保证`writeDB:`方法是线程安全的。

(5)dispatch队列还实现其它一些常用函数

1)重复执行block，需要注意的是这个方法是同步返回，也就是说等到所有block执行完毕才返回，如需异步返回则嵌套在dispatch_async中来使用。多个block的运行是否并发或串行执行也依赖queue的是否并发或串行。

`void dispatch_apply(size_t iterations, dispatch_queue_t queue, void (^block)(size_t)); `

2)这个函数可以设置同步执行的block，它会等到在它加入队列之前的block执行完毕后，才开始执行。在它之后加入队列的block，则等到这个block执行完毕后才开始执行

`void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);`

3)同上，除了它是同步返回函数

`void dispatch_barrier_sync(dispatch_queue_t queue, dispatch_block_t block);`

4)延迟执行block

`void dispatch_after(dispatch_time_t when, dispatch_queue_t queue, dispatch_block_t block);`

dispatch队列不支持cancel（取消），没有实现dispatch_cancel()函数，不像NSOperationQueue，不得不说这是个小小的缺憾。

##Dispatch Group

1、创建dispatch组

`dispatch_group_t group = dispatch_group_create(); `

2、启动dispatch队列中的block关联到group中

```
dispatch_group_async(group, queue, ^{ 
　　// 。。。 
}); 
```

3、等待group关联的block执行完毕，也可以设置超时参数

`dispatch_group_wait(group, DISPATCH_TIME_FOREVER); `

4、为group设置通知一个block，当group关联的block执行完毕后，就调用这个block。类似dispatch_barrier_async。

```
dispatch_group_notify(group, queue, ^{
　　// 。。。 
}); 
```

5、手动管理group关联的block的运行状态（或计数），进入和退出group次数必须匹配

`dispatch_group_enter(group);`和`dispatch_group_leave(group);`

通常使用的下面的写法

```
dispatch_group_async(group, queue, ^{ 
　　// 。。。 
}); 
```

等价于 

```
dispatch_group_enter(group);
dispatch_async(queue, ^{
　　//。。。
　　dispatch_group_leave(group);
});
```
所以，可以利用dispatch_group_enter、 dispatch_group_leave和dispatch_group_wait来实现同步,[具体例子](http://stackoverflow.com/questions/10643797/wait-until-multiple-operations-executed-including-completion-block-afnetworki/10644282#10644282)。

##参考资料

1、[《iOS多线程的初步研究（七）-- dispatch对象》](http://www.cnblogs.com/sunfrog/p/3281612.html)

2、[《iOS多线程的初步研究（八）-- dispatch队列》](http://www.cnblogs.com/sunfrog/p/3305614.html)

3、[《iOS多线程的初步研究（九）-- dispatch源》](http://www.cnblogs.com/sunfrog/p/3308766.html)

4、[《iOS多线程的初步研究（十）-- dispatch同步》](http://www.cnblogs.com/sunfrog/p/3313424.html)

5、[《多线程编程4 - GCD》](http://blog.csdn.net/q199109106q/article/details/8566300)

6、[《使用GCD》](http://blog.devtang.com/blog/2012/02/22/use-gcd/)
