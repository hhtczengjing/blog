---
layout: post
title: "iOS中使用PLCrashReporter收集Crash"
date: 2016-05-21 23:40
comments: true
tags: iOS
---

iOS应用程序在上线过程中可能会出现各种闪退，如果APP经常出现闪退会让一部分用户选择卸载，这样会带来很大的损失，下图(来自于Bugly)可以看出APP出现崩溃后会带来的影响。

![bugly介绍](/images/ios-plcrashreporter/bugly_introduction_1.jpg)

对于这些崩溃信息怎么收集分析就是一个很大的问题，通过解析Crash文件可以帮助我们改善APP，修复bug等。收集Crash信息的方式有很多，比较常见的是使用第三方服务，比如友盟、百度统计、Bugly等。(下图为Bugly）

![bugly介绍](/images/ios-plcrashreporter/bugly_introduction_2.png)

###使用系统自带的Crash收集

```
//需要捕获的signal
static int s_fatal_signals[] = {
    SIGABRT,
    SIGBUS,
    SIGFPE,
    SIGILL,
    SIGSEGV,
    SIGTRAP,
    SIGTERM,
    SIGKILL
};

static int s_fatal_signal_num = sizeof(s_fatal_signals)/sizeof(s_fatal_signals[0]);

void UncaughtExceptionHandler(NSException *exception) {
    NSArray *arr = [exception callStackSymbols];//得到当前调用栈信息
    NSString *reason = [exception reason];//非常重要，就是崩溃的原因
    NSString *name = [exception name];//异常类型
}

void SignalHandler(int code) {
    NSLog(@"signal handler = %d",code);
}

void InitCrashReport() {
    // 1 linux错误信号捕获
    for (int i = 0; i < s_fatal_signal_num; ++i) {
        signal(s_fatal_signals[i], SignalHandler);
    }
    // 2 objective-c未捕获异常的捕获
    NSSetUncaughtExceptionHandler(&UncaughtExceptionHandler);
}

int main(int argc, char * argv[]) {
    @autoreleasepool {
        InitCrashReport();
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

###使用PLCrashReporter收集

[PLCrashReporter](https://www.plcrashreporter.org)是一个开源的崩溃日志收集的库，很多崩溃收集的第三方服务都是基于PLCrashReporter来实现的。

![plcrashreporter_logo.png](/images/ios-plcrashreporter/plcrashreporter_logo.png)

(1)使用CocoaPods的方式快速集成

`pod ‘PLCrashReporter’, ‘~> 1.2’`

在Podfile中加入上面的代码，然后执行pod install.

(2)启用PLCrashReporter收集Crash

引入头文件：

```
#import <CrashReporter/CrashReporter.h>
#import <CrashReporter/PLCrashReportTextFormatter.h>
```

在`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions;`中添加如下代码：

```
PLCrashReporter *crashReporter = [PLCrashReporter sharedReporter];
NSError *error;
// Check if we previously crashed
if ([crashReporter hasPendingCrashReport]) {
	[self handleCrashReport];
}
// Enable the Crash Reporter
if (![crashReporter enableCrashReporterAndReturnError: &error]) {
	NSLog(@"Warning: Could not enable crash reporter: %@", error);
}
```

(3)处理CrashReport

```
- (void)handleCrashReport {
    PLCrashReporter *crashReporter = [PLCrashReporter sharedReporter];
    NSData *crashData;
    NSError *error;
    
    // Try loading the crash report
    crashData = [crashReporter loadPendingCrashReportDataAndReturnError:&error];
    if (crashData == nil) {
        NSLog(@"Could not load crash report: %@", error);
        [crashReporter purgePendingCrashReport];
        return;
    }
    
    // We could send the report from here, but we'll just print out some debugging info instead
    PLCrashReport *report = [[PLCrashReport alloc] initWithData:crashData error:&error];
    if (report == nil) {
        NSLog(@"Could not parse crash report");
        [crashReporter purgePendingCrashReport];
        return;
    }
    
    //TODO:send the report
    NSLog(@"Crashed on %@", report.systemInfo.timestamp);
    NSLog(@"Crashed with signal %@ (code %@, address=0x%" PRIx64 ")", report.signalInfo.name, report.signalInfo.code, report.signalInfo.address);
    NSString *humanReadText = [PLCrashReportTextFormatter stringValueForCrashReport:report withTextFormat:PLCrashReportTextFormatiOS];
    NSLog(@"Crashed Format Text %@", humanReadText);
    
    [crashReporter purgePendingCrashReport];
    return;
}
```

###分析Crash Report

####（1）获取PLCrashReporter收集到的crash文件

通过PLCrashReporter的`- (NSData *)loadPendingCrashReportDataAndReturnError:(NSError **)outError;`方法获取到的NSData格式的数据是通过protobuf处理过的数据,需要使用工具进行转换处理。

转换代码如下：

`./bin/plcrashutil convert --format=iphone example_report.plcrash > crash.log`

说明：

- 1）工具下载地址：`https://www.plcrashreporter.org/static/downloads/PLCrashReporter-1.2.zip`

- 2）example_report.plcrash文件指的是前面的NSData保存的文件

####（2）获取dsym文件

Xcode编译项目后，我们会看到一个同名的dSYM文件，dSYM是保存16进制函数地址映射信息的中转文件，我们调试的symbols都会包含在这个文件中，并且每次编译项目的时候都会生成一个新的dSYM文件，位于`/Users/<用户名>/Library/Developer/Xcode/Archives`目录下，对于每一个发布版本我们都很有必要保存对应的Archives文件.

每一个`xxx.app`和`xxx.app.dSYM`文件都有对应的UUID，`crash`文件(指的是通过工具转换过的文件)也有自己的UUID，只要这三个文件的UUID一致，我们就可以通过他们解析出正确的错误函数信息了。

1）查看xxx.app 文件的 UUID，在terminal中输入命令 ：

`dwarfdump --uuid xxx.app/xxx` (xxx代表你的项目名)

2）查看xxx.app.dSYM文件的UUID，在terminal中输入命令：

`dwarfdump --uuid xxx.app.dSYM` (xxx代表你的项目名)

3）crash文件内第一行`Incident Identifier`就是该crash文件的UUID。

####（3）使用symbolicatecrash分析

Xcode 7.3中symbolicatecrash工具存放的路径是：

`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`

建议拷贝出来放到一个专门的文件夹下。

设置DEVELOPER_DIR:

`export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`

`./symbolicatecrash crash.log xxx.app.dSYM > result.log` (xxx代表你的项目名)

####（4）

`xcrun atos -o xxx.app/xxx -l 0x11111111` (xxx代表你的项目名, 0x11111111表示要分析的地址)，推荐使用`dSYM 文件分析工具`(下载地址：http://pan.baidu.com/s/1bnkxPvT)

![dSYM 文件分析工具](/images/ios-plcrashreporter/dsym_tool.png)

###参考资料

1.[《PLCrashReporter Documentation》](https://www.plcrashreporter.org/documentation)

2.[《iOS崩溃收集与分析，使用PLCrashReporter》](https://everettjf.github.io/2015/09/09/ios-plcrashreporter)

3.[《iOS开发技巧－崩溃调试》](http://www.jianshu.com/p/77660e626874)

4.[《dSYM 文件分析工具》](http://answerhuang.duapp.com/index.php/2014/07/06/dsym_tool/)