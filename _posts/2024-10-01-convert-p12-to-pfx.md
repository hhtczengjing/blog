---
layout: post
title: "使用OpenSSL转换p12到pfx格式"
date: 2024-10-01 10:00:00 +0800
comments: true
tags: Note
---

年中有个服务使用到的签名证书过期了，需要重新生成一个签名证书。这个服务使用的是 .pfx 格式的证书文件，需要将 .p12 格式的证书文件转换为 .pfx 格式的证书文件。

导出的方式比较繁琐：首先是在 macOS 上使用 Keychain 工具，将证书导出为 .p12 格式的证书文件。然后使用 Windows 管理主控台 工具，将 .p12 格式的证书文件转换为 .pfx 格式的证书文件。

![windows-mmc](/images/convert-p12-to-pfx/windows-mmc.png)

一般来说从Keychain导出的p12格式证书的证书链是不完整的，比如有的时候安装别人提供的 p12 证书，安装后提示”证书不受信任“，其实是缺少中间证书（Apple Worldwide Developer Relations Certification Authority， 可以到 [Apple PKI](https://www.apple.com/certificateauthority/) 网站下载） ，需要手动添加。

![wwdr-missing](/images/convert-p12-to-pfx/wwdr-missing.png)

上面的步骤实在是太麻烦了，而且在不同Windows版本下面操作的路径也还不太一样，所以研究了一下如何使用命令行工具在macOS上直接将 .p12 格式的证书文件转换为 .pfx 格式的证书文件。

### 转换 .p12 到 .pfx 格式并补全证书链

本文使用的 `openssl-1.1.1k` 这个版本(因为手头上刚好有这个版本的代码没验证过其他的版本了。推荐使用brew安装指定版本, 手动编译后面会附录编译的脚本)，使用 3.3.2 版本执行转换命令的时候会出现下面的错误提示：

```
Error outputting keys and certificates
000C82F401000000:error:0308010C:digital envelope routines:inner_evp_generic_fetch:unsupported:crypto/evp/evp_fetch.c:355:Global default library context, Algorithm (RC2-40-CBC : 0), Properties ()
```

这里记录一下使用 OpenSSL 工具将 .p12 格式的证书文件转换为 .pfx 格式的证书文件，并且完善证书链的步骤：

1、从 .p12（PKCS#12 格式）的证书文件中提取出客户端证书（PEM 格式）

```
openssl pkcs12 -in cert.p12 -out clientcert.pem -nodes -clcerts
```

会提示需要输入密码，这里需要输入 p12 证书对应的密码。

> 参数说明：

- `-in`：指定输入文件。在这里，cert.p12 是输入的 PKCS#12 格式的证书文件，它包含了要提取的客户端证书以及可能的私钥和其他相关信息。
- `-out`：指定输出文件。`clientcert.pem`是输出的证书文件的名称，PEM 是一种常用的用于存储证书、私钥等加密对象的文本格式，以 Base64 编码存储数据。
- `-nodes`：这个参数表示不加密私钥。如果没有这个参数，提取出来的私钥会被加密，控制台会提示输入密码。
- `-clcerts`：表示只输出客户端证书（client certificates）。在 PKCS#12 文件中可能包含多个证书（例如根证书、中间证书等），这个参数确保只提取出客户端证书部分到输出文件中。

2、将 DER 格式的 X.509 证书转换为 PEM 格式

```
openssl x509 -in AppleWWDRCAG3.cer -inform DER -out AppleWWDRCAG3.pem
openssl x509 -in AppleRootCA-G3.cer -inform DER -out AppleRootCA-G3.pem
```

`AppleWWDRCAG3.cer`/`AppleRootCA-G3.cer` 是从 [Apple PKI](https://www.apple.com/certificateauthority/) 网站下载的证书文件，都是 DER 格式的 X.509 证书。

![apple-pki](/images/convert-p12-to-pfx/apple-pki.png)

> 参数说明：

- `-in`：用于指定输入文件。在这里，`AppleWWDRCAG3.cer`/`AppleRootCA-G3.cer` 是要被转换格式的输入证书文件。
- `-inform`: 指定输入文件的格式。DER（Distinguished Encoding Rules）是一种二进制编码格式，用于表示 X.509 证书等 ASN.1（Abstract Syntax Notation One）结构的数据。
- `-out`：用于指定输出文件。

3、将三个 PEM 格式的证书文件的内容按顺序合并到一个新的文件

```
cat clientcert.pem AppleWWDRCAG3.pem AppleRootCA-G3.pem >> clientcertchain.pem
```

其实就是将三个 PEM 格式的证书文件的内容按顺序合并到一个新的文件，然后保存为 clientcertchain.pem。这样就构成了一个完整的证书链文件，即 clientcertchain.pem。

4、将 PEM 格式的证书链转换为 .pfx （PKCS#12格式）

```
openssl pkcs12 -export -in clientcertchain.pem -out clientcertchain.pfx
```

> 参数说明：

- `-export`：这个参数表示执行导出操作，即将输入的内容转换为 PKCS#12 格式并保存到指定的输出文件中。
- `-in`：指定输入文件。这里的 clientcertchain.pem 是输入的包含证书链的 PEM 格式文件。
- `-out`：指定输出文件。clientcertchain.pfx 是输出文件的名称，其格式为 PKCS#12 格式，这种格式常用于在不同系统或应用程序之间交换包含私钥、证书等的加密包。

### 手动编译 OpenSSL

1、下载

到官网下载指定的OpenSSL源码，下载地址：https://openssl-library.org/source/old/1.1.1/index.html

![download-openssl](/images/convert-p12-to-pfx/download-openssl.png)

2、编译

> 温馨提示：需要有 Xcode 和 cmake 的环境。

```
rm -rf build

make clean
./Configure darwin64-x86_64-cc --prefix="$(pwd)/build/openssl-x86_64" no-asm -mmacosx-version-min=10.15
make -j4
make install

make clean
./Configure darwin64-arm64-cc --prefix="$(pwd)/build/openssl-arm64" no-asm -mmacosx-version-min=10.15
make -j4
make install

rm -rf $(pwd)/build/openssl
mkdir -p $(pwd)/build/openssl/lib $(pwd)/build/openssl/bin

lipo -create $(pwd)/build/openssl-arm64/lib/libssl.a $(pwd)/build/openssl-x86_64/lib/libssl.a -output $(pwd)/build/lib/libssl.a
lipo -create $(pwd)/build/openssl-arm64/lib/libcrypto.a $(pwd)/build/openssl-x86_64/lib/libcrypto.a -output $(pwd)/build/lib/libcrypto.a

lipo -create $(pwd)/build/openssl-arm64/bin/openssl $(pwd)/build/openssl-x86_64/bin/openssl -output $(pwd)/build/openssl/bin/openssl
```

编译后的产物在 `build/openssl` 目录下。完整的编译脚本可以参考：https://github.com/hhtczengjing/openssl_builder

### 参考资料

- 1、[Create a .pfx/.p12 Certificate File Using OpenSSL](https://www.ssl.com/how-to/create-a-pfx-p12-certificate-file-using-openssl/)

- 2、[Export Certificates and Private Key from a PKCS#12 File with OpenSSL](https://www.ssl.com/how-to/export-certificates-private-key-from-pkcs12-file-with-openssl/)

- 3、[Error: Unsupported Algorithm When Extracting Public Certificate from PKCS#12 File](https://github.com/openssl/openssl/discussions/23089)

- 4、[How to generate .key and .crt from PKCS12 file](https://dev.to/okolilemuel/how-to-generate-key-and-crt-from-pkcs12-file-2k39)

- 5、[.pfx 证书和 .cer 证书](https://www.cnblogs.com/ljhdo/p/14109218.html)

- 6、[Adding certificate chain to p12(pfx) certificate](https://stackoverflow.com/questions/18787491/adding-certificate-chain-to-p12pfx-certificate)