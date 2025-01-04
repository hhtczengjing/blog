---
layout: post
title: "使用lycium编译鸿蒙上的OpenSSL库"
date: 2025-01-04 12:00:00 +0800
comments: true
tags: Note
---

近期想要实现 mars xlog 在 HarmonyOS NEXT 上面使用，项目使用到了 OpenSSL 需要先编译生成能在 OpenHarmony 上面运行的产物（具体依赖的版本为OpenSSL 1.0.2k，可以从 `https://github.com/Tencent/mars/blob/master/mars/openssl/openssl_lib_iOS/VERSION` 看到）。

当前在 HarmonyOS 上主流的 C/C++ 交叉编译方式主要以下几种：

- [CMake 编译构建](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-cmake-adapts-to-harmonyos-V5)

- [Configure编译构建](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-configure-adapts-to-harmonyos-V5)

- [Make编译构建](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-make-adapts-to-harmonyos-V5)

- [GN编译构建](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-gn-adapts-to-harmonyos-V5)

为了方便开发者便捷完成 C/C++ 三方库快速交叉编译，官方实现了一套交叉编译框架 [lycium](https://gitee.com/openharmony-sig/tpc_c_cplusplus) ，只需要设置对应 C/C++ 三方库的编译方式以及编译参数，通过 lycium 就能快速的构建出能在 HarmonyOS 上运行的二进制文件。

### 编译环境准备

1、下载command-line-tools

到官方[下载中心](https://developer.huawei.com/consumer/cn/download/)下载当前平台需要的Command Line Tools。

![download_command_line_tools](/images/ohos-build-openssl-by-lycium/download_command_line_tools.png)

下载完成后解压，放到一个指定的目录下面，如：

```
~
└── ohos
    └── command-line-tools
```

2、安装必要的环境

lycium 框架需要编译环境中包含以下几个基本编译命令：gcc, cmake, make, pkg-config, autoconf, autoreconf, automake。在macOS上可以使用下面的命令快速安装所需要的环境。

```
brew install cmake
brew install pkg-config
brew install autoconf
brew install ninja
brew install coreutils
brew install sha512sum
brew install wget
```

3、下载lycium

```
git clone https://gitee.com/openharmony-sig/tpc_c_cplusplus.git
```

4、配置lycium

主要是修改 tpc_c_cplusplus/lycium/script/envset.sh 这个文件。

(1) 配置支持x86_64架构

新增下面个方法：

```
setx86_64ENV() {
    export AS=${OHOS_SDK}/native/llvm/bin/llvm-as
    export CC=${OHOS_SDK}/native/llvm/bin/x86_64-unknown-linux-ohos-clang
    export CXX=${OHOS_SDK}/native/llvm/bin/x86_64-unknown-linux-ohos-clang++
    export LD=${OHOS_SDK}/native/llvm/bin/ld.lld
    export STRIP=${OHOS_SDK}/native/llvm/bin/llvm-strip
    export RANLIB=${OHOS_SDK}/native/llvm/bin/llvm-ranlib
    export OBJDUMP=${OHOS_SDK}/native/llvm/bin/llvm-objdump
    export OBJCOPY=${OHOS_SDK}/native/llvm/bin/llvm-objcopy
    export NM=${OHOS_SDK}/native/llvm/bin/llvm-nm
    export AR=${OHOS_SDK}/native/llvm/bin/llvm-ar
    export CFLAGS="-DOHOS_NDK -fPIC -D__MUSL__=1"
    export CXXFLAGS="-DOHOS_NDK -fPIC -D__MUSL__=1"
    export LDFLAGS=""
}

unsetx86_64ENV() {
    unset AS CC CXX LD STRIP RANLIB OBJDUMP OBJCOPY NM AR CFLAGS CXXFLAGS LDFLAGS
}
```

(2) 修改 CC/CXX 对应的路径

修改 setarm32ENV 函数：

```
export CC=${OHOS_SDK}/native/llvm/bin/armv7-unknown-linux-ohos-clang
export CXX=${OHOS_SDK}/native/llvm/bin/armv7-unknown-linux-ohos-clang++
```

修改 setarm64ENV 函数：

```
export CC=${OHOS_SDK}/native/llvm/bin/aarch64-unknown-linux-ohos-clang
export CXX=${OHOS_SDK}/native/llvm/bin/aarch64-unknown-linux-ohos-clang++
```

### 编译OpenSSL

1、在 tpc_c_cplusplus/thirdparty 下创建 openssl_1_0_2k 目录

默认提供了openssl 1.0.2u的编译配置，但是我们需要的是1.0.2k，基本是差不多的，只需要进行一些微调就可以了。将 tpc_c_cplusplus/community/openssl_1_0_2u 目录下面的 HPKBUILD 和 SHA512SUM 拷贝过来。

（1）修改 HPKBUILD 文件的 pkgname/pkgver 改成我们需要的版本

```
pkgname=openssl_1_0_2k
pkgver=OpenSSL_1_0_2k
```

(2) 获取 openssl-OpenSSL_1_0_2k.tar.gz 文件的 SHA512

可以通过命令获取

```
shasum -a 512 openssl-OpenSSL_1_0_2k.tar.gz
```

2、编译

```
export OHOS_SDK="$HOME/ohos/command-line-tools/sdk/default/openharmony" # 环境变量，为上面command-line-tools解压后的路径
cd tpc_c_cplusplus/lycium
./build.sh openssl_1_0_2k
```

最终的产物在 tpc_c_cplusplus/lycium/usr/openssl_1_0_2k 

完整的脚本可以参考：

```
git clone https://github.com/hhtczengjing/ohos_lycium_openssl.git --recurse-submodules
sh +x ./build.sh
```

### 参考资料

- 1、[CMake构建工程配置HarmonyOS编译工具链](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-cmake-adapts-to-harmonyos-V5)

- 2、[Configure构建工程配置HarmonyOS编译工具链](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-configure-adapts-to-harmonyos-V5)

- 3、[使用lycium工具快速编译三方库](https://developer.huawei.com/consumer/cn/doc/best-practices-V5/bpta-lycium-adapts-to-harmonyos-V5)