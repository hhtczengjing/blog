---
layout: post
title: "Objective-C中的Method Swizzling"
date: 2014-07-30 22:59:19 +0800
comments: true
tags: iOS
---

Objective-C对象在收到消息之后会经过`Dynamic Message Dispatch System(动态消息派发系统)`来进行处理，该系统会查出消息对应的方法并执行其代码。那么对于给定`@selector`名称相对应的方法是否可以在运行期可以动态改变呢？如果能善用这个特性，则可发挥出巨大优势，因为我们可以不需要源码也不需要通过继承子类来覆写对应的方法就能改变这个类本身的功能。

没错，Objective-C中确实提供了这样的操作，这就是我们这里会介绍到的Method Swizzling(方法的调配)。在Objective-C中每个类都有一个方法列表，类的方法列表会把`@selector`映射到相关的方法实现之上，使得`Dynamic Message Dispatch System(动态消息派发系统)`能够根据这个找到应该调用的方法，这些方法均以函数指针（`IMP`）的形式来表示,其原型如下：

`id (*IMP)(id, SEL, ...)`


##应用实例
在实际的应用中，使用`Method Swizzling`直接来交换两个方法的实现的意义不大，但是可以通过这一手段来为既有的方法实现添加新增的功能，比如说在调用`lowercaseString`方法适合记录某些信息，这个时候可以通过交换方法实现来达成这个目标，我们新增一个方法在此方法中实现所需的附加功能，并调用原有的实现。

`NSString+Extension.h`

```
@interface NSString (Extension)

- (NSString *)devzeng_lowercaseString;

@end
```

`NSString+Extension.m`

```
@implementation NSString (Extension)

- (NSString *)devzeng_lowercaseString
{
    NSString *lowercase = [self devzeng_lowercaseString];
    NSLog(@"【Debug】%@->%@", self, lowercase);
    return lowercase;
}

@end
```

上面的代码看上去好像会陷入递归调用的死循环，其实不然。此方法是准备和`lowercaseString`互换的，所以在运行期`devzeng_lowercaseString`其实对应的是`lowercaseString`，最后通过下面的代码来实现互换。

`main.m`

```
int main(int argc, const char * argv[])
{

    @autoreleasepool {
       
        Method ori_Method =  class_getInstanceMethod([NSString class], @selector(lowercaseString));
        Method my_Method = class_getInstanceMethod([NSString class], @selector(devzeng_lowercaseString));
        method_exchangeImplementations(ori_Method, my_Method);
        
        NSString *string = @"Hello, World!";
        NSString *lowercaseString = [string lowercaseString];    
    }
    return 0;
}
```

输出结果：`【Debug】Hello, World!->hello, world!`

通过上面的示例可以看出，我们可以通过`Method Swizzling`为一些不透明的类的某些方法做“黑盒测试”,比如增加一些日志信息等，这样在一定的程度上有利于程序的调试。然而，此做法如果滥用会造成代码变得不易读而且不利于维护。

##Method Swizzling的陷阱

使用`Method Swizzling`就好比在厨房用一把锋利的小刀。有些人担心小刀会割伤自己而害怕使用，然而有些人却钟情于使用它。虽然说`Method Swizzling`能够让我们写出更好、更高效、更加可维护的代码但是它也可能给我们带来很多很麻烦的bug。下面是一些在使用`Method Swizzling`会遇到的让人头疼的陷阱。

- Method swizzling is not atomic

- Changes behavior of un-owned code

- Possible naming conflicts

- Swizzling changes the method's arguments

- The order of swizzles matters

- Difficult to understand (looks recursive)

- Difficult to debug

关于上面的内容的细致描述请移步[《Objective-C的hook方案（一）: Method Swizzling》](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)（中文版本）和 [《Swizzle method》](http://dreamume.blog.163.com/blog/static/184923719201411302817262/) (英文版本)

##深度阅读

1、[《Objective-C的hook方案（一）: Method Swizzling》](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)

2、[《Swizzle method》](http://dreamume.blog.163.com/blog/static/184923719201411302817262/)


