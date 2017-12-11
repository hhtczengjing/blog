---
layout: post
title: "iOS开发中使用Mantle构建模型层"
date: 2017-12-10 13:08
comments: true
tags: Note
---

在iOS的开发中为了快速的实现产品的迭代和新功能的开发，常常会弱化Model的功能，NSDictionary作为承载业务的数据类型出现在各种地方（SQLite，Model Object,API Service...）,直接使用objectForKey的方式进行数据的读取，参数和值的正确性完全没有经过编译器检查，字符串很容易写错，极容易导致在运行阶段出现低级bug.

### 1、Property名称转换

由于API使用的开发语言与iOS所使用的Objective-C是截然不同的，所以可能将一些保留关键字作为property的名称(如id)，或者不小心override掉基类的属性(如description)。还有可能API中使用了一个很糟糕的名称，或者使用了不符合Objective-C命名规范的名称，这些我们都需要作转换。

只需要实现MTLJSONSerializing protocol并在+JSONKeyPathsByPropertyKey方法中定义好新旧名称的映射关系即可，Mantle会在序列化及反序列化时对属性名进行自动的转换。

```
+ (NSDictionary *)JSONKeyPathsByPropertyKey {
    return @{
              @keypath(model, xh)   : @"XH",
              @keypath(model, bbmc) : @"BBMC",
              @keypath(model, bbh)  : @"BBH",
              @keypath(model, bz)   : @"BZ",
              @keypath(model, xzdz) : @"XZDZ",
              @keypath(model, fbsj) : @"FBSJ"
    };
}
```

### 2、Property类型转换

#### （1）将NSString的字符串转换为NSURL：

iOS中处理URL使用的是NSURL类型，但JSON只支持基本的字符串，Mantle可以自动帮你转换成NSURL。

```
+ (NSValueTransformer *)URLJSONTransformer {
    return [NSValueTransformer valueTransformerForName:MTLURLValueTransformerName];
}
```

#### （2）将NSString的字符串映射为enum的成员

```
+ (NSValueTransformer *)stateJSONTransformer {
    return [NSValueTransformer mtl_valueMappingTransformerWithDictionary:@{
        @"open": @(GHIssueStateOpen),
        @"closed": @(GHIssueStateClosed)
    }];
}
```

#### （3）将字典转换为Objective-C的对象

```
+ (NSValueTransformer *)assigneeJSONTransformer {
    return [MTLJSONAdapter dictionaryTransformerWithModelClass:GHUser.class];
}
```

#### （4）实现JSON Value和Model Property的互相转换

1）比如将JSON字符串中的`2015-10-10 10:10`转换为NSDate类型，然后将NSDate类型的数据转换为`yyyy-MM-dd HH:mm`形式的字符串

```
+ (NSValueTransformer *)updatedAtJSONTransformer {
    return [MTLValueTransformer transformerUsingForwardBlock:^id(NSString *dateString, BOOL *success, NSError *__autoreleasing *error) {
        return [self.dateFormatter dateFromString:dateString];
    } reverseBlock:^id(NSDate *date, BOOL *success, NSError *__autoreleasing *error) {
        return [self.dateFormatter stringFromDate:date];
    }];
}
```

2）比如需要将`{"id":"xxx", "subject":"", "time":""}`这样一个JSON字符串转换为SearchListMailItemModel可以按照下面的方式进行转换

```
+ (NSValueTransformer *)mailJSONTransformer {
    return [MTLValueTransformer transformerUsingForwardBlock:^id(NSArray *value, BOOL *success, NSError *__autoreleasing *error) {
        if(!value || value.count <= 0) {
            return nil;
        }
        return [SearchListMailItemModel pdModelsWithArray:value error:nil];
    }];
}
```

### 3、统一空值处理

有的时候API响应的数据会出现空值，如status字段有些情况下是一个字符串，少数情况下返回的是null,如`{"status":null}`,Mantle在这种情况下会将status转换为nil,但如果是标量（NSInteger,CGFloat）的时候直接会抛出异常。

```
@interface MTLModel (PDNullableScalar)

@end

@implementation MTLModel (PDNullableScalar)

- (void)setNilValueForKey:(NSString *)key {
    [self setValue:@0 forKey:key];  
}

@end
```

### 4、便捷`NSDictionary->Model`, `Model->NSDictionary`转换

#### (1) 将`NSDictionary -> Model`

```
CustomModel *model = [MTLJSONAdapter modelOfClass:PDBaseModel.class fromJSONDictionary:dictionary error:error];
```

#### (2) 将`Model -> NSDictionary`

```
NSDictionary *dict = [MTLJSONAdapter JSONDictionaryFromModel:self error:error];
```

### 5、结合RPJSONValidator实现JSON格式的校验 

```
NSDictionary *dictionary = @{@"name":@"zhangsan", @"age":20};
NSDictionary *requirementsDictionary = @{@"name":RPValidatorPredicate.isString.isOptional, @"age":RPValidatorPredicate.isNumber.isOptional};
NSError *validError = nil;
BOOL valid = [RPJSONValidator validateValuesFrom:dictionary withRequirements:requirementsDictionary error:&validError];
if (!valid) {
        NSString *failingKeys = validError.userInfo[RPJSONValidatorFailingKeys];
        NSString *errorDescription = [NSString stringWithFormat:@"%@校验失败，错误信息：%@", NSStringFromClass(self), failingKeys];
        NSAssert(NO, errorDescription);
        return NO;
}
```

### 6、自动NSCoding/NSCopying Procotol的实现

如果需要将一个对象进行归档处理，在OC中需要实现NSCoding的Protocol，需要实现下面两个方法：

```
- (id)initWithCoder:(NSCoder *)coder {
    self = [self init];
    if (self == nil) return nil;
    _URL = [coder decodeObjectForKey:@"URL"];
    _number = [coder decodeObjectForKey:@"number"];
    _state = [coder decodeUnsignedIntegerForKey:@"state"];
    return self;
}
```

```
- (void)encodeWithCoder:(NSCoder *)coder {
    if (self.URL != nil) [coder encodeObject:self.URL forKey:@"URL"];
    if (self.number != nil) [coder encodeObject:self.number forKey:@"number"];
    [coder encodeUnsignedInteger:self.state forKey:@"state"];
}
```

如果需要将一个对象进行Copy处理，那么需要实现NSCopying的Protocol，需要实现下面两个方法：

```
- (id)copyWithZone:(NSZone *)zone {
    GHIssue *issue = [[self.class allocWithZone:zone] init];
    issue->_URL = self.URL;
    issue->_number = self.number;
    issue->_state = self.state;
    return issue;
}
```

但是使用Mantle之后NSCoding和NSCopyin Procotol是默认进行实现的，直接使用即可

### 参考资料

1、[为什么唱吧iOS 6.0选择了Mantle](http://www.iwangke.me/2014/10/13/Why-Changba-iOS-choose-Mantle/)

2、[Mantle官网](https://github.com/Mantle/Mantle)