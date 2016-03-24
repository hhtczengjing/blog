---
layout: post
title: "Objective-C中的Associated Objects"
date: 2014-07-28 21:50:11 +0800
comments: true
tags: iOS
---

`Associated Objects`(关联对象)或者叫做关联引用(`Associated References`)，是作为Objective-C 2.0运行时功能被引入到Mac OS 10.6 Snow Leopard(及iOS4)系统。与它相关在`<objc/rumtime.h>`中有3个C函数，他们可以让对象在运行时关联任何值：

(1)用给定的key和policy来为指定对象(object)设置关联对象值(value)

```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
```

(2)根据给定的key从指定对象(object)中获取相对应的关联对象值

```
id objc_getAssociatedObject(id object, const void *key)
```

(3)移除指定对象的全部关联对象

```
void objc_removeAssociatedObjects(id object)
```

开发者可以通过上面的几个函数在分类中给已存在的类中添加自定义的属性。


##关联对象的特性

在`#import <objc/runtime.h>`中关于objc_AssociationPolicy的定义如下：

```
enum {
    OBJC_ASSOCIATION_ASSIGN = 0,
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
    OBJC_ASSOCIATION_RETAIN = 01401,
    OBJC_ASSOCIATION_COPY = 01403
};
typedef uintptr_t objc_AssociationPolicy;
```

`objc_AssociationPolicy`是一个枚举类型的数据结构定义了`OBJC_ASSOCIATION_ASSIGN`、`OBJC_ASSOCIATION_RETAIN_NONATOMIC`、`OBJC_ASSOCIATION_COPY_NONATOMIC`、`OBJC_ASSOCIATION_RETAIN`和`OBJC_ASSOCIATION_COPY`这样五个关联对象特性，每个特性的描述如下：

- `OBJC_ASSOCIATION_ASSIGN`,给关联对象指定弱引用,相当于`@property(assign)`或`@property(unsafe_unretained)` 

- `OBJC_ASSOCIATION_RETAIN_NONATOMIC`,给关联对象指定非原子的强引用,相当于`@property(nonatomic,strong)或@property(nonatomic,retain)`

- `OBJC_ASSOCIATION_COPY_NONATOMIC`,给关联对象指定非原子的copy特性,相当于`@property(nonatomic,copy)`

- `OBJC_ASSOCIATION_RETAIN`,给关联对象指定原子强引用,相当于`@property(atomic,strong)或@property(atomic,retain)`

- `OBJC_ASSOCIATION_COPY`,给关联对象指定原子copy特性,相当于`@property(atomic,copy)`


##示例代码
创建一个NSObject名为AssociatedObject的Category，向其中关联一个叫做associatedObject的属性。

#####NSObject+AssociatedObject.h
```
@interface NSObject (AssociatedObject)

@property (nonatomic, strong) id associatedObject;

@end

```

#####NSObject+AssociatedObject.m

```
@implementation NSObject (AssociatedObject)

@dynamic associatedObject;

- (void)setAssociatedObject:(id)object
{
    objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)associatedObject
{
    return objc_getAssociatedObject(self, @selector(associatedObject),);
}

@end
```

说明：

(1)引入`#import <objc/runtime.h>`的头文件

(2)在头文件中使用`@property`，对应的.m文件中使用`@dynamic`

(3)key值可以使用`@selector(associatedObject)`,也可以使用
`static const void *AssociatedObjectKey = &AssociatedObjectKey;`,推荐使用前者

##小结

(1)使用关联，我们可以不用修改类的定义而为其增加存储空间，在对于无法访问到类的源码的时候非常有用

(2)关联是通过关键字来进行操作的，因而可以为任何对象增加任意多的关联，每个都使用不同的关键字即可。

(3)关联是可以保证被关联的对象在关联对象的整个生命周期都是可用的。

##延伸阅读

1、[《objc_setAssociatedObject通过alert传值》](http://blog.csdn.net/sijiazhentan/article/details/11772827)

2、[《自定义NSIndexPath — 給Category添加property》](http://blog.csdn.net/zhoutao198712/article/details/21598911)

3、[《当property遇上category》](http://www.cnblogs.com/tekkaman/p/3753629.html)
