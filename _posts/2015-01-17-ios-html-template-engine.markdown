---
layout: post
title: "iOS中使用模板引擎渲染HTML界面"
date: 2015-01-17 21:22:04 +0800
comments: true
tags: iOS
---

在iOS实际的开发中，使用UIWebView来加载数据使用的场景特别多。很多时候我们会动态的从服务器获取一段HTML的内容，然后App这边动态的处理这段HTML内容用于展示在UIWebView上。使用到的API接口为：

`- (void)loadHTMLString:(NSString *)string baseURL:(NSURL *)baseURL;`

由于HTML内容通常是变化的，所以我们需要动态生成HTML代码。通常我们从服务器端获取到标题、时间、作者和对应的内容，然后我们需要对这些数据处理之后拼接成一段HTML字符串。对于传统的做法是将上面的需要替换的内容填写一些占位符，放到指定的文件中如（content.html）,如下所示：

```
<!DOCTYPE html>
<html>
    <head>
        <title>key_title</title>
    </head>
    <body>
        <div>
            <div>
                 <h2>key_title</h2>
                 <div>key_date key_author</div>
                 <hr/>
            </div>
            <div>key_content</div>
       </div>
    </body>
</html>
```

然后在指定的地方使用如下的方式动态生成HTML代码：

```
- (NSString *)loadHTMLByStringFormat:(NSDictionary *)data
{
    NSString *templatePath = [[NSBundle mainBundle] pathForResource:@"content" ofType:@"html"];
    NSMutableString *html = [[NSMutableString alloc] initWithContentsOfFile:templatePath encoding:NSUTF8StringEncoding error:nil];
    [html replaceOccurrencesOfString:@"key_title" withString:data[@"title"] options:NSCaseInsensitiveSearch range:NSMakeRange(0, html.length)];
    [html replaceOccurrencesOfString:@"key_author" withString:data[@"author"] options:NSCaseInsensitiveSearch range:NSMakeRange(0, html.length)];
    [html replaceOccurrencesOfString:@"key_date" withString:data[@"date"] options:NSCaseInsensitiveSearch range:NSMakeRange(0, html.length)];
    [html replaceOccurrencesOfString:@"key_content" withString:data[@"content"] options:NSCaseInsensitiveSearch range:NSMakeRange(0, html.length)];
    return html;
}
```

在实际的使用中发现还是存在不少的问题，比如我们需要对数据进行预先处理的时候需要写大量的

```
- (NSUInteger)replaceOccurrencesOfString:(NSString *)target withString:(NSString *)replacement options:(NSStringCompareOptions)options range:(NSRange)searchRange;
```
这样的替换，而且对于一些特殊的字符还需要进行特殊处理等，实在不是太友好，这样就需要一个引擎来专门处理这些事情，本文主要介绍`MGTemplateEngine`和`GRMustache`的使用。

### 使用模板引擎

#### MGTemplateEngine的使用

[MGTemplateEngine](http://mattgemmell.com/mgtemplateengine-templates-with-cocoa/)是[`Matt Gemmell`](https://github.com/mattgemmell)的作品，它是一个比较流行的模板引擎，它的模板语言比较类似于`Smarty`、`FreeMarker`和`Django`。另外它可以支持自定义的Filter（以便实现自定义的渲染逻辑），需要依赖正则表达式的工具类`RegexKit`。

1、创建模板

```
<!DOCTYPE html>
<html>
    <head>
        <title>{{ title }}</title>
    </head>
    <body>
        <div>
            <div>
                <div>
                    <h2>{{ title }}</h2>
                    <div>{{ date }} {{ author }}</div>
                    <hr/>
                </div>
                <div>{{ content }}</div>
            </div>
        </div>
    </body>
</html>
```

2、渲染生成HTML字符串

```
- (NSString *)loadHTMLByMGTemplateEngine:(NSDictionary *)data
{
    NSString *templatePath = [[NSBundle mainBundle] pathForResource:@"template" ofType:@"html"];
    MGTemplateEngine *engine = [MGTemplateEngine templateEngine];
    [engine setMatcher:[ICUTemplateMatcher matcherWithTemplateEngine:engine]];
    [engine setObject:data[@"title"] forKey:@"title"];
    [engine setObject:data[@"author"] forKey:@"author"];
    [engine setObject:data[@"date"] forKey:@"date"];
    [engine setObject:data[@"content"] forKey:@"content"];
    return [engine processTemplateInFileAtPath:templatePath withVariables:nil];
}
```

3、说明

（1）MGTemplateEngine提供的示例程序是运行在Mac OS上的，如果要使用到iOS上面需要引入Foundation框架

（2）对于运行在Xcode6以上的环境下创建的工程由于没有PCH文件可能会报错，需要在MGTemplateEngine的各个头文件中引入Foundation框架

（3）MGTemplateEngine在GitHub上的地址为`https://github.com/mattgemmell/MGTemplateEngine`。
 
#### GRMustache的使用  

相比`MGTemplateEngine`来说`GRMustache`简单不少，

1、处理模板文件

模板文件和MGTemplateEngine的一样。

2、渲染生成HTML字符串

```
- (NSString *)loadHTMLByGRMustache:(NSDictionary *)data
{
    NSString *templatePath = [[NSBundle mainBundle] pathForResource:@"template" ofType:@"html"];
    NSString *template = [NSString stringWithContentsOfFile:templatePath encoding:NSUTF8StringEncoding error:nil];
    return [GRMustacheTemplate renderObject:data fromString:template error:nil];
}
```

3、说明

（1）renderObject使用的数据的key必须要和模板中的占位符一一对应起来

（2）GRMustache在GitHub上的地址为`https://github.com/groue/GRMustache`

### 参考资料

1、[《MGTemplateEngine - Templates with Cocoa》](http://mattgemmell.com/mgtemplateengine-templates-with-cocoa/)

2、[《MGTemplateEngine 模版引擎简单使用》](http://blog.csdn.net/crazy_srufboy/article/details/21748995)

3、[《GRMustache Document》](http://mustache.github.io/mustache.5.html)