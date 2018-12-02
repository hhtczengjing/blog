---
layout: post
title: "iOS中使用cryptopp进行加解密"
date: 2018-10-04 18:46:37 +0800
comments: true
tags: iOS
---

[Crypto++](https://www.cryptopp.com)是一个免费开源的加解密库，支持一些非常丰富的加解密算法(如AES/RSA等)。如果要考虑到实现一套跨平台多端加解密效果一致可以考虑使用该库，当然使用各自平台提供的api也能实现。

目前只提供了源码的方式，如果要集成到iOS的项目里面需要先编译成静态库，下面就`5.6.2`这个版本进行介绍：

### 1.编译静态库

#### (1) 下载源码

```
git clone https://github.com/weidai11/cryptopp.git
git checkout CRYPTOPP_5_6_2
```

#### (2) 准备一个`Makefile`文件

```
touch Makefile
```

文件内容为：

```
CXXFLAGS += -g -O2 -Wall -Wno-unused -Wno-unknown-pragmas -DNDEBUG -DCRYPTOPP_DISABLE_ASM -DCRYPTOPP_DISABLE_SSE2 -MMD -MT dependencies

TARGET = libcryptopp.a
SRCS = $(shell echo *.cpp)
OBJS = $(SRCS:.cpp=.o)

.phoney: clean

all: $(TARGET)

$(TARGET): $(OBJS)
	$(AR) $(ARFLAGS) $@ $(OBJS)
	$(RANLIB) $@

%.o : %.cpp
	$(CXX) $(CXXFLAGS) -c $<

clean:
	$(RM) $(TARGET) $(OBJS)
```

#### (3) 编写编译脚本

##### 1) iOS平台

创建脚本文件：

```
touch build-ios.sh
```

脚本内容为：

```
#!/bin/bash

XCODE_ROOT=`xcode-select -print-path`

echo "making for iOS"
ARCHS="x86_64 i386 armv7 armv7s arm64"
SDK_VERSION=`xcrun --sdk iphoneos --show-sdk-version 2> /dev/null`
MIN_SDK_VERSION=7.0

echo "sdk version is: "
echo ${SDK_VERSION}

IOS_STATIC_ARCHIVES=""
for ARCH in ${ARCHS}
do
    PLATFORM=""
    if [ "${ARCH}" == "i386" ] || [ "${ARCH}" == "x86_64" ]; then
        PLATFORM="iPhoneSimulator"
    else
        PLATFORM="iPhoneOS"
    fi

    export DEV_ROOT="${XCODE_ROOT}/Platforms/${PLATFORM}.platform/Developer"
    export SDK_ROOT="${DEV_ROOT}/SDKs/${PLATFORM}${SDK_VERSION}.sdk"
    export TOOLCHAIN_ROOT="${XCODE_ROOT}/Toolchains/XcodeDefault.xctoolchain/usr/bin/"
    export CC="clang -arch $ARCH -fembed-bitcode"
    export CXX=clang++
    export AR=${TOOLCHAIN_ROOT}libtool
    export RANLIB=${TOOLCHAIN_ROOT}ranlib
    export ARFLAGS="-static -o"
    export LDFLAGS="-arch ${ARCH} -isysroot ${SDK_ROOT}"
    export BUILD_PATH="IOS_BUILD_${ARCH}"
    export CXXFLAGS="-x c++ -arch ${ARCH} -isysroot ${SDK_ROOT} -I${BUILD_PATH} -miphoneos-version-min=${MIN_SDK_VERSION} -mios-simulator-version-min=${MIN_SDK_VERSION}"

    mkdir -pv ${BUILD_PATH}
    make -f Makefile
    mv *.o ${BUILD_PATH}
    mv *.d ${BUILD_PATH}
    mv libcryptopp.a ${BUILD_PATH}

    IOS_STATIC_ARCHIVES="${IOS_STATIC_ARCHIVES} ${BUILD_PATH}/libcryptopp.a"
done

echo "Creating universal library..."
mkdir -p bin/ios
lipo -create ${IOS_STATIC_ARCHIVES} -output bin/ios/libcryptopp.a

echo "removing thin archs"
for ARCH in ${ARCHS}
do
    directoryName=IOS_BUILD_${ARCH}
    rm -rf ${directoryName}
done

echo "Build done!"
```

##### 2) macOS平台

创建脚本文件：

```
touch build-macOS.sh
```

脚本内容为：

```
#!/bin/bash

XCODE_ROOT=`xcode-select -print-path`

echo "making for macOSX"
ARCHS="x86_64 i386"
SDK_VERSION=`xcrun --sdk macosx --show-sdk-version 2> /dev/null`
MIN_SDK_VERSION=10.10
PLATFORM="MacOSX"

echo "sdk version is: "
echo ${SDK_VERSION}

MACOSX_STATIC_ARCHIVES=""
for ARCH in ${ARCHS}
do
    export DEV_ROOT="${XCODE_ROOT}/Platforms/${PLATFORM}.platform/Developer"
    export SDK_ROOT="${DEV_ROOT}/SDKs/${PLATFORM}${SDK_VERSION}.sdk"
    export TOOLCHAIN_ROOT="${XCODE_ROOT}/Toolchains/XcodeDefault.xctoolchain/usr/bin/"
    export CC="clang -arch $ARCH -fembed-bitcode"
    export CXX=clang++
    export AR=${TOOLCHAIN_ROOT}libtool
    export RANLIB=${TOOLCHAIN_ROOT}ranlib
    export ARFLAGS="-static -o"
    export LDFLAGS="-arch ${ARCH} -isysroot ${SDK_ROOT}"
    export BUILD_PATH="MACOSX_BUILD_${ARCH}"
    export CXXFLAGS="-x c++ -arch ${ARCH} -isysroot ${SDK_ROOT} -I${BUILD_PATH} -mmacosx-version-min=${MIN_SDK_VERSION}"

    mkdir -pv ${BUILD_PATH}
    make -f Makefile
    mv *.o ${BUILD_PATH}
    mv *.d ${BUILD_PATH}
    mv libcryptopp.a ${BUILD_PATH}

    MACOSX_STATIC_ARCHIVES="${MACOSX_STATIC_ARCHIVES} ${BUILD_PATH}/libcryptopp.a"
done

echo "Creating universal library..."
mkdir -p bin/macosx
lipo -create ${MACOSX_STATIC_ARCHIVES} -output bin/macosx/libcryptopp.a

echo "removing thin archs"
for ARCH in ${ARCHS}
do
    directoryName=MACOSX_BUILD_${ARCH}
    rm -rf ${directoryName}
done

echo "Build done!"
```

### 2.使用

下面就MD5加密进行示例如何使用，更多其他的使用方式请参考官方的文档：

```
// md5
// 引用头文件`md5.h`
- (NSString *)getMD5String:(NSString *)str {
    NSData *data = [str dataUsingEncoding:NSUTF8StringEncoding];
    if(!data || data.length <= 0) {
        return nil;
    }
    CryptoPP::MD5 md5;
    byte digest[CryptoPP::MD5::DIGESTSIZE];
    md5.CalculateDigest(digest, (const byte*)[data bytes], [data length]);
    NSData *md5Dat = [NSData dataWithBytes:digest length:sizeof digest];
    NSMutableString *s = [NSMutableString string];
    unsigned char *hashValue = (byte *)[md5Dat bytes];
    int i;
    for (i = 0; i < [md5Dat length]; i++) {
        [s appendFormat:@"%02x", hashValue[i]];
    }
    return s;
}
```

### 参考资料

1、[Cryptopp iOS 使用 RSA加密解密和签名验证签名](https://www.cnblogs.com/cocoajin/p/6112562.html)

2、[官方文档](https://www.cryptopp.com/docs/ref/)