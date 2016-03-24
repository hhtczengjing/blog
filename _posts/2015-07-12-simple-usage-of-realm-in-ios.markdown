---
layout: post
title: "iOS中Realm数据库的基本用法"
date: 2015-07-12 19:10:40 +0800
comments: true
tags: iOS
---

Realm是由`Y Combinator`公司孵化的一款支持运行在手机、平板和可穿戴设备上的嵌入式数据库（旨在取代CoreData和Sqlite）。Realm并不是对Core Data的简单封装，相反地，Realm并不是基于Core Data，也不是基于SQLite所构建的。它拥有自己的数据库存储引擎，可以高效且快速地完成数据库的构建操作。

![hello_realm.png](/images/realm_usage/hello_realm.png)

Realm可以轻松地移植到项目当中，并且绝大部分常用的功能（比如说插入、查询等等）都可以用一行简单的代码轻松完成！目前支持Objective-C、Swift和Java三种语言，也就是说能在iOS、Android和Mac上面跨平台使用。

综上，Realm主要有以下几个优点：

- Easy to Use(简单易用)：Core Data和SQLite冗余、繁杂的知识和代码足以吓退绝大多数刚入门的开发者，而换用Realm，则可以极大地减少学习代价和学习时间，让应用及早用上数据存储功能。

- Cross-Platform(跨平台)：现在绝大多数的应用开发并不仅仅只在iOS平台上进行开发，还要兼顾到Android平台的开发。为两个平台设计不同的数据库是愚蠢的，而使用Realm数据库，iOS和Android无需考虑内部数据的架构，调用Realm提供的API就可以完成数据的交换，实现“一个数据库，两个平台无缝衔接”。

- Fast(高效)：Realm相比使用CoreData和原生的SQLite来说速度更快更加高效，而且代码量更少。

###快速集成Realm

1、下载最新的[Realm](http://realm.io)更新包，解压zip文件

2、将`ios/static`目录下面的`Realm.framework`文件拖到项目里面（确保Copy items if needed选中）

3、在`target -> Build Phases -> Link Binary with Libraries`中添加`libc++.dylib`

说明：

（1）对于使用Swift的童鞋，请讲Swift/RLMSupport.swift文件拖到项目中（确保Copy items if needed选中）

（2）推荐使用Cocoapods进行安装，在Podfile中添加`pod 'Realm'`即可

（3）也可以自行到Github上面下载代码进行编译，此处不作过多的介绍

运行环境：

（1）支持 >= iOS7.0, >= OS X 10.9, 及WatchKit

（2）推荐使用Xcode 5以上的IDE，支持Swift

###辅助工具和插件的安装

####1、Realm Browser

Realm官方非常贴心的向开发者提供了一个用于查看喝编辑Realm数据的工具`Realm Browser`.

![browser.png](/images/realm_usage/browser.png)

在上面下载的更新包的`browser/`下面有个Realm Browser拖到Application文件夹或者是直接打开都行。另外可以使用菜单的`tool -> generate demo datebase`,生成测试数据用于测试Realm数据库的使用

####2、Xcode Plugin

在Realm中使用到最多的是Realm Model(继承自RLMObject的类，后面有介绍)。官方提供了一个Xcode的插件让我们在创建模型变得非常轻松

![plugin.png](/images/realm_usage/plugin.png)

安装使用：

（1）最简单的安装方式是通过Alcatraz,搜索`RealmPlugin`直接安装

（2）或者是打开zip文件夹下面的`plugin/RealmPlguin.xcodeproj`,build一下就安装好了

安装完后重启Xcode生效，在创建model的时候选择New File(或⌘N)，选择Realm按照要求输入model的名字就OK啦。

###Realm的使用

####1、构建数据库

Realm提供了三种方式创建数据库，一种是存储在默认路径下的数据库，一种是我们可以自己指定数据库文件的存储路径和只读属性，另外还可以使用内存数据库。

（1）默认Realm数据库

`RLMRealm *realm = [RLMRealm defaultRealm];`

可以通过：`[RLMRealm defaultRealmPath]`查看默认存储的路径。

（2）自定义Realm数据库

```
NSString *docPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
NSString *dbPath = [docPath stringByAppendingPathComponent:@"db/db.realm"];
RLMRealm *realm = [RLMRealm realmWithPath:dbPath];
```
或者是

```
NSString *docPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
NSString *dbPath = [docPath stringByAppendingPathComponent:@"db/db.realm"];
RLMRealm *realm = [RLMRealm realmWithPath:dbPath readOnly:YES error:nil];
```
其中readOnly表示创建的数据库是只读数据库。

（3）内存数据库

正常的Realm数据库是存储在硬盘上的， 但你也可以通过使用`+ (instancetype)inMemoryRealmWithIdentifier:(NSString *)identifier;`来创建一个内存数据库。

```
RLMRealm *realm = [RLMRealm inMemoryRealmWithIdentifier:@"test"];
```

注意：内存数据库在每次程序退出时不会保存数据。如果某个内存Realm实例没有被引用，所有的数据在实例对象释放的适合也会被释放。建议你在app中用强引用来钳制所有新建的内存Realm数据库实例。

####2、数据模型

Realm的数据模型是用传统的Objective-C接口（interface）和属性（@property）定义的。 只要定义 `RLMObject`的一个子类或者一个现成的模型类，你就能轻松创建一个Realm的数据模型对象。Realm模型对象和其他的Objective-c的功能很相似–你可以给它们添加你自己的方法和protocol然后和其他的对象一样使用。 唯一的限制就是从它们被创建开始，只能在一个进程中被使用。

如果已经安装了Realm Xcode插件，在`New File`对话框中会有一个很漂亮的样板，你可以用它来创建interface和implementation文件。

用一个对象来表示一篇文章（Articl）,创建的数据模型如下：

- `Article.h`

```
@interface Article : RLMObject

@property NSString *num;//序号
@property NSString *title;//标题
@property NSString *link;//链接地址
@property NSString *author;//作者
@property NSString *tag;//标签分类
@property NSInteger weight;//权重

@end

RLM_ARRAY_TYPE(Article)
```

- `Article.m`

```
@implementation Article

//主键
+ (NSString *)primaryKey {
    return @"num";
}

//需要添加索引的属性
+ (NSArray *)indexedProperties {
    return @[@"title"];
}

//默认属性值
+ (NSDictionary *)defaultPropertyValues {
    return @{@"author":@"zengjing"};
}

//忽略的字段
+ (NSArray *)ignoredProperties {
    return @[@"weight"];
}

@end
```

说明：

（1）Realm支持以下的属性（property）种类：BOOL, bool, int, NSInteger, long, float, double, CGFloat, NSString, NSDate 和 NSData。

（2）你可以使用`RLMArray<Object>`和`RLMObject`来模拟对一或对多的关系(Realm也支持RLMObject继承)

（3）Realm忽略了Objective-C的property attributes(如nonatomic, atomic, strong, copy, weak 等等）。 所以，推荐在创建模型的时候不要使用任何的property attributes。但是，假如你设置了，这些attributes会一直生效直到RLMObject被写入realm数据库。

（4）定义了`RLM_ARRAY_TYPE(Article)`这个宏表示支持`RLMArray<Article>`该属性

（5）另外Realm提供了以下几个方法供对属性进行自定义：

1）`+ (NSArray *)indexedProperties;`: 可以被重写来来提供特定属性（property）的属性值（attrbutes）例如某个属性值要添加索引。

2）`+ (NSDictionary *)defaultPropertyValues;`: 为新建的对象属性提供默认值。

3）`+ (NSString *)primaryKey;`: 可以被重写来设置模型的主键。定义主键可以提高效率并且确保唯一性。

4）`+ (NSArray *)ignoredProperties;`：可以被重写来防止Realm存储模型属性。

####3、数据增删改查

（1）存储数据

创建数据模型对象：

```
Article *article = [[Article alloc] init];
article.num      = @"1";
article.title    = @"iOS开发中集成Reveal";
article.link     = @"http://blog.devzeng.com/blog/ios-reveal-integrating.html";
article.tag      = @"iOS";
```

存储数据：

```
RLMRealm *realm = [RLMRealm defaultRealm];
[realm beginWriteTransaction];
[realm addObject:article];
[realm commitWriteTransaction];
```

（2）删除数据

1）删除指定的数据：

`- (void)deleteObject:(RLMObject *)object;`

2）删除一组数据：

`- (void)deleteObjects:(id)array;`

3）删除全部的数据：

`- (void)deleteAllObjects;`

（3）修改数据

修改数据如果该条数据不存在则会新建一条数据。

1）针对单个数据进行的修改或新增：

`- (void)addOrUpdateObject:(RLMObject *)object;`

2）针对一组数据的修改或新增：

`- (void)addOrUpdateObjectsFromArray:(id)array;`

说明：对于增加、删除、修改必须要在事务中进行操作。

（5）查询数据

1）查询全部数据

```
RLMResults *results = [Article allObjects];
```

或指定Realm数据库：

```
NSString *path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
NSString *realmPath = [path stringByAppendingPathComponent:@"devzeng.realm"];
RLMRealm *realm = [RLMRealm realmWithPath:realmPath];
RLMResults *results = [Article allObjectsInRealm:realm];
```

2）条件查询

假设要查询所有分组是iOS和作者是zengjing的文章：

```
RLMResults *results = [Article objectsWhere:@"tag = 'iOS' AND author = 'zengjing'"];
```

也可以使用谓词查询：

```
NSPredicate *pred = [NSPredicate predicateWithFormat:@"tag = '%@' AND author = '%@'", @"iOS", @"zengjing"];
RLMResults *results = [Article objectsWithPredicate:pred];
```

3）条件排序

假设要查询所有分组是iOS和作者是zengjing的文章，然后筛选出来的结果按照num字段进行递增排序：

```
RLMResults *results = [[Article objectsWhere:@"tag = 'iOS' AND author = 'zengjing'"] sortedResultsUsingProperty:@"num" ascending:YES];
```
4）链式查询(结果过滤)

假设要查询所有所属分组是iOS的文章，然后从中筛选出作者是zengjing的数据：

```
RLMResults *results1 = [Article objectsWhere:@"tag = 'iOS'"];
RLMResults *results2 = [results1 objectsWhere:@"author = 'zengjing'"];
```

####4、通知

每当一次写事务完成Realm实例都会向其他线程上的实例发出通知，可以通过注册一个block来响应通知：

```
self.token = [realm addNotificationBlock:^(NSString *note, RLMRealm * realm) {
    [_listTableView reloadData];
}];
```

只要有任何的引用指向这个返回的notification token，它就会保持激活状态。在这个注册更新的类里，你需要有一个强引用来钳制这个token， 因为一旦notification token被释放，通知也会自动解除注册。

```
@property (nonatomic, strong) RLMNotificationToken *token;
```

另外可以使用下面的方式解除通知：

```
[realm removeNotification:self.token];
```

####5、数据库版本迁移

当你和数据库打交道的时候，时不时的你需要改变数据模型（model），但因为Realm中得数据模型被定义为标准的Objective-C interfaces，要改变模型，就像改变其他Objective-C interface一样轻而易举。举个例子，假设有个数据模型`Person`:

在v1.0中数据模型如下：

```
// v1.0
@interface Person : RLMObject
@property NSString *firstName;
@property NSString *lastName;
@property int age;
@end
```

升级到v2.0之后将firstName和lastName字段合并为一个字段fullName

```
// v2.0
@interface Person : RLMObject
@property NSString *fullName; // new property
@property int age;
@end
```
迁移的逻辑可以为：

```
[RLMRealm setSchemaVersion:2.0 forRealmAtPath:[RLMRealm defaultRealmPath] 
                         withMigrationBlock:^(RLMMigration *migration, 
                                              NSUInteger oldSchemaVersion) {
  [migration enumerateObjects:Person.className 
                        block:^(RLMObject *oldObject, RLMObject *newObject) {
    if (oldSchemaVersion < 2.0) {
      newObject[@"fullName"] = [NSString stringWithFormat:@"%@ %@", oldObject[@"firstName"], oldObject[@"lastName"]];
    }
  }];
}];
```

当版本升级到3.0时，添加新的属性email

```
// v3.0
@interface Person : RLMObject
@property NSString *fullName;
@property NSString *email;   // new property
@property int age;
@end
```
迁移的逻辑可以为：

```
[RLMRealm setSchemaVersion:2.0 forRealmAtPath:[RLMRealm defaultRealmPath] 
                         withMigrationBlock:^(RLMMigration *migration, 
                                              NSUInteger oldSchemaVersion) {
  [migration enumerateObjects:Person.className 
                        block:^(RLMObject *oldObject, RLMObject *newObject) {
    //处理v2.0的更新
    if (oldSchemaVersion < 2.0) {
      newObject[@"fullName"] = [NSString stringWithFormat:@"%@ %@", oldObject[@"firstName"], oldObject[@"lastName"]];
    }
    //处理v3.0的更新
    if(oldSchemaVersion < 3.0) {
    	newObject[@"email"] = @"";
    }
  }];
}];
```

###说明（摘自官方的FAQ）

1、realm的支持库有多大？

一旦你的app编译完成，realm的支持库应该只有1MB左右。我们发布的那个可能有点大（iOS ~37MB, OSX ~2.4MB）， 那是因为它们还包含了对其他构架的支持（ARM，ARM64，模拟器的是X86）和一些编译符号。 这些都会在你编译app的时候被Xcode自动清理掉。

2、我应该在正式产品中使用realm吗？

自2012年起，realm就已经开始被用于正式的商业产品中了。正如你预期，我们的objective-c & Swift API 会随着社区的反馈不断的完善和进化。 所以，你也应该期待realm带给你更多的新特性和版本修复。

3、我要付realm的使用费用吗？

不要， Realm的彻底免费的， 哪怕你用于商业软件。

###参考资料

1、[《Realm Document》](https://realm.io/docs/objc/latest)

2、[《Realm数据库基础教程》](http://www.jianshu.com/p/052c763d5693)
