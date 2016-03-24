---
layout: post
title: "在iOS9中使用CoreSpotlight"
date: 2015-10-18 16:37:38 +0800
comments: true
tags: iOS
---

在iOS9之前，用户可以通过Spotlight中输入关键字来查找App。在iOS9中Apple随之发布了一套全新的iOS9 Search APIs之后，开发者不但可以自由的将App的部分内容建立索引，还能对Spotlight上的搜索结果以及点击不同的结果显示的内容进行设置。

![apple_search_demo.png](/images/ios9_corespotlight/apple_search_demo.png)

###三种搜索的API简介

####NSUserActivity

`NSUserActivity`是iOS8专为Handoff推出的API，在iOS9得到了提升。现在用户只需要提供元数据(metadata)就能搜索到不同的活动(Activity)了。也就是说Spotlight可以将Activity加入到索引，而NSUserActivity就好比网页浏览器的历史堆栈(history stack, 可以理解为最近使用的App，或者是近期常用联系人等)，用户可以通过Spotlight搜索到最近的Activity.

####Web Markup

`Web Markup`在网页上显示App的内容并建立Spotlight索引，如此一来即便没有安装某个App，苹果的索引器也能在网页上搜索特别的标记（markup），在Safari或Spotlight上显示搜索结果。

####CoreSpotlight

`NSUserActivity`帮助储存用户历史，而全新的`CoreSpotlight`则能为App中的任何内容创建索引，实质是在用户设备上提供基础的`CoreSpotlight`索引渠道，满足用户另外一个需求。最典型的一个例子是印象笔记和iOS自带的Note，用户无需打开对应的App通过Spotlight就能搜索到笔记的内容，然后快速打开。

###使用CoreSpotlight APIs

####1、引入CoreSpotlight.framework

![corespotlight_framework](/images/ios9_corespotlight/corespotlight_framework.png)

####2、创建索引

#####（1）创建索引所需的元数据

为了让内容可以被搜索，首先需要创建一个包含元数据的属性

![searchable_attributes.png](/images/ios9_corespotlight/searchable_attributes.png)

```
CSSearchableItemAttributeSet *attributeSet = [[CSSearchableItemAttributeSet alloc] initWithItemContentType:@"contact"];
attributeSet.title = @"标题";
attributeSet.contentDescription = @"内容";
attributeSet.keywords = @[@"关键字1", @"关键字2"];
attributeSet.thumbnailData = UIImagePNGRepresentation([UIImage imageNamed:@"缩略图"]);
```

#####（2）创建索引

```
CSSearchableItem *searchableItem = [[CSSearchableItem alloc] initWithUniqueIdentifier:@"" domainIdentifier:@"" attributeSet:attributeSet];
searchableItem.expirationDate = [NSDate dateWithTimeIntervalSinceNow:3600];
```
注意：

1)uniqueIdentifier:在应用程序中这个值是唯一的，由于这个可以用于索引的更新、删除索引是唯一的，推荐使用UUID或者是搜索的条目的主键。

2)domainIdentifier:一个可选的标识符，用来表示item的域(domain)或者所有者，这个可能用一个账户的邮箱作为identifier来索引数据，并且当账户删除的时候可以根据这个来删除数据，一般情况下domainIdentifier应该是这种格式<account-id>.<mailbox-id>并且不能包含时间。

3)expirationDate:过期的日期，默认过期的日期是一个月

#####（3）将索引加入到CoreSpotlight

```
[[CSSearchableIndex defaultSearchableIndex] indexSearchableItems:@[searchableItem] completionHandler:^(NSError * _Nullable error) {
	if(error) {
		NSLog(@"%@", [error localizedDescription]);
	}
}];
```

####3、配置用户点击搜索结果的处理动作

```
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void(^)(NSArray * __nullable restorableObjects))restorationHandler {
	if([userActivity.activityType isEqualToString:CSSearchableItemActionType]) {
		NSString *uniqueIdentifier = userActivity.userInfo[CSSearchableItemActivityIdentifier];
        //这里根据这个uniqueIdentifier可以跳转到详细信息页面
        return YES;
    }
    return YES;
}
```

###参考资料

1、[《App Search Programming Guide》](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/index.html)

2、[《iOS9 Day-by-Day :: Day 1 :: Search APIs》](https://www.shinobicontrols.com/blog/ios9-day-by-day-day1-search-apis)

3、[《CoreSpotlight.framework注释翻译》](http://blog.csdn.net/mengxiangyue/article/details/46575977)