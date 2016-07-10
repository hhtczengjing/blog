---
layout: post
title: "iOS中使用Protocol Buffers"
date: 2016-07-09 09:40
comments: true
tags: iOS
---

[`Google Protocol Buffer`](https://developers.google.com/protocol-buffers/)(简称`Protobuf`)是由Google推出的一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或RPC数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

> Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

Protobuf和XML相比同是数据交换协议，不过Protobuf更小、更快、也更简单。可以通过定义自己的数据结构，然后使用Protobuf的代码生成器生成代码，用生成的代码来读写这个数据结构。Protobuf具有如下几个优点：

(1)“向后”兼容性好。不必担心因为消息结构的改变而造成的大规模的代码重构或者迁移的问题，因为添加新的消息中的字段并不会引起已经发布的程序的任何改变。

(2)语义更清晰。Protobuf使用`.proto`文件描述数据交换的格式，然后Protobuf编译器会将`.proto`文件编译生成对应的数据访问类以对`Protobuf`数据进行序列化、反序列化操作），无需解释器之类的东西。

(3)简单易学。使用Protobuf无需学习复杂的文档对象模型，它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf比其他的技术更加有吸引力。

Protobuf开源并托管在Github，项目地址是：[https://github.com/google/protobuf/](https://github.com/google/protobuf/)。目前Google提供了三种语言的实现：java、c++ 和python，每一种实现都包含了相应语言的编译器以及库文件。

###Protocol协议格式说明

要通信，必须有协议，否则双方无法理解对方的码流。在Protobuf中，协议是由一系列的消息组成的。因此最重要的就是定义通信时使用到的消息格式。消息由至少一个字段组合而成，类似于C语言中的结构。每个字段都有一定的格式。字段格式：

`限定修饰符① | 数据类型② | 字段名称③ | = | 字段编码值④ | [字段默认值⑤]`

#####①．限定修饰符包含`required\optional\repeated`

- `Required`: 表示是一个必须字段，必须相对于发送方，在发送消息之前必须设置该字段的值，对于接收方，必须能够识别该字段的意思。发送之前没有设置required字段或者无法识别required字段都会引发编解码异常，导致消息被丢弃。

- `Optional`：表示是一个可选字段，可选对于发送方，在发送消息时，可以有选择性的设置或者不设置该字段的值。对于接收方，如果能够识别可选字段就进行相应的处理，如果无法识别，则忽略该字段，消息中的其它字段正常处理。因为optional字段的特性，很多接口在升级版本中都把后来添加的字段都统一的设置为optional字段，这样老的版本无需升级程序也可以正常的与新的软件进行通信，只不过新的字段无法识别而已，因为并不是每个节点都需要新的功能，因此可以做到按需升级和平滑过渡。

- `Repeated`：表示该字段可以包含0~N个元素。其特性和optional一样，但是每一次可以包含多个值。可以看作是在传递一个数组的值。

#####②．数据类型。

Protobuf定义了一套基本数据类型。几乎都可以映射到C++\Java\Objective-C等语言的基础数据类型。数据类型如下图所示：

![protobuf_field_type.png](/images/ios-protobuf/protobuf_field_type.png)

#####③．字段名称

字段名称的命名与C、C++、Java和Objective-C等语言的变量命名方式几乎是相同的。Protobuf建议字段的命名采用以下划线分割的驼峰式。例如`first_name`而不是`firstName`。

#####④．字段编码值

有了该值，通信双方才能互相识别对方的字段。当然相同的编码值，其限定修饰符和数据类型必须相同。编码值的取值范围为 1~2^32（4294967296）。

其中 1~15的编码时间和空间效率都是最高的，编码值越大，其编码的时间和空间效率就越低（相对于1-15），当然一般情况下相邻的2个值编码效率的是相同的，除非2个值恰好实在4字节，12字节，20字节等的临界区。比如15和16。1900~2000编码值为Protobuf系统内部保留值，建议不要在自己的项目中使用。Protobuf还建议把经常要传递的值把其字段编码设置为1-15之间的值。消息中的字段的编码值无需连续，只要是合法的，并且不能在同一个消息中有字段包含相同的编码值。

建议：项目投入运营以后涉及到版本升级时的新增消息字段全部使用optional或者repeated，尽量不使用required。如果使用了required，需要全网统一升级，如果使用optional或者repeated可以平滑升级。

#####⑤．默认值。

当在传递数据时，对于required数据类型，如果用户没有设置值，则使用默认值传递到对端。当接受数据是，对于optional字段，如果没有接收到optional字段，则设置为默认值。

#####⑥．关于import

Protobuf接口文件可以像C语言的h文件一个，分离为多个，在需要的时候通过import导入需要对文件。其行为和C语言的`#include`或者iOS中的的`#import`的行为大致相同。

#####⑦．关于package

避免名称冲突，可以给每个文件指定一个package名称，对于java解析为java中的包。对于C++则解析为名称空间。

#####⑧．关于message

支持嵌套消息，消息可以包含另一个消息作为其字段。也可以在消息内定义一个新的消息。

#####⑨．关于enum

枚举的定义和C++相同，但是有一些限制。枚举值必须大于等于0的整数。使用分号(;)分隔枚举变量而不是C++语言中的逗号(,)

###安装Protobuf Compiler

目前Protobuf稳定的版本是[`v2.6.1`](https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz),最新的版本是`v3.0.0-beta-3.1`估计很快就会推出3.0的正式版本了。本文所有的操作都是基于2.6.1的版本。

####1.检查是否安装Homebrew:

`brew -v`

如果没有安装可以通过下面的方式进行安装：

`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

####2.安装Protobuf Compiler

```
brew install automake 
brew install libtool 
brew install protobuf
```
####3.创建Protobuf Compiler的符号链接(可选步骤)

`ln -s /usr/local/Cellar/protobuf/2.6.1/bin/protoc /usr/local/bin`

####4.安装Objective-C的扩展

```
git clone https://github.com/alexeyxo/protobuf-objc.git
cd protobuf-objc
./scripts/build.sh
```

###集成Protobuf Runtime

推荐使用CocoaPods的方式进行集成，在Podfile中加入下面的代码：

`pod 'ProtocolBuffers', '~> 1.9.10'`

###示例

####1.创建`.proto`文件

```
option java_package = "com.devzeng.statistics.proto";
package Statistics;

message AppContextMessage {
	required string net_connetion_type = 1;    //网络连接类型,可选值是wifi/wwan/unknown
	required string screen_resolution  = 2;    //屏幕分辨率
	optional string device_token		= 3;	//推送设备号
	optional string mac_address			= 4;	//mac地址
	optional string carrier_name 		= 5;	//运营商名称
	optional string device_model 		= 6;	//设备型号
	optional string wifi_name 			= 7;	//wifi名称
	optional string device_name 		= 8;	//设备名称
	required string device_type 		= 9;	//设备类型，可选值是iPhone/iPad/Android
	required string system_version 		= 10;	//系统版本
	required string device_uuid 		= 11;	//设备号
	optional string gps 				= 12;	//经纬度(latitude,longitude)
	required string app_version 		= 13;	//当前安装的APP版本号
}
```

一个比较好的习惯是认真对待proto文件的文件名。比如将命名规则定于如下：
`packageName.MessageName.proto`

在上例中，package 名字叫做`Statistics`，定义了一个消息`AppContextMessage`。那么可以可以命名为`Statistics.AppContextMessage.proto`

####2.使用Protobuf编译器生成平台相关的代码

`protoc --plugin=/usr/local/bin/protoc-gen-objc *.proto --objc_out="./"`

####3.数据的序列化

```
AppContextMessageBuilder *builder = [AppContextMessage builder];
[builder setNetConnetionType:@"wifi"];
[builder setScreenResolution:@""];
[builder setMacAddress:@""];
[builder setCarrierName:@"中国联通"];
[builder setDeviceModel:@""];
[builder setWifiName:@"my_wifi(8c:21:a:44:f0:c)"];
[builder setDeviceType:@"iPhone"];
[builder setSystemVersion:@""];
[builder setDeviceUuid:@""];
[builder setDeviceName:@""];
[builder setGps:@""];
[builder setAppVersion:@"1.0.1_20"];
NSData *data = [builder build].data;
```

###参考资料

1.[《Protobuf's Documentation》](https://developers.google.com/protocol-buffers/)

2.[《Google Protocol Buffer 的使用和原理》](http://www.ibm.com/developerworks/cn/linux/l-cn-gpb/)

3.[《Protocol Buffer搭建及示例》](http://www.tanhao.me/code/150911.html/)

4.[《Protocol Buffers for Objective-C》](https://github.com/alexeyxo/protobuf-objc)