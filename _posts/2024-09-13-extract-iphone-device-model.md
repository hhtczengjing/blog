---
layout: post
title: "从Xcode中提取iPhone新机型信息"
date: 2024-09-13 21:40:00 +0800
comments: true
tags: Note
---

2024年9月10日，iPhone 16 系列正式发布，本次主要发布了iPhone 16、iPhone 16 Plus、iPhone 16 Pro和iPhone 16 Pro Max四款机型。

![iphone16-series](/images/extract-iphone-device-model/iphone16-series.png)

每年发布新机型都需要提取一下iPhone的新机型信息，方便后续查询展示使用。现有的历史的数据是从[Apple_mobile_device_types.txt](https://gist.github.com/adamawolf/3048717) 文件中提取的，新的数据的话就要更新之后才能获取到。

去年发布的 iPhone 15 的时候是从 Xcode 本地数据库中获取的，最近想要查数据的时候发现不记得怎么弄了，这里记录一下操作的路径。

首先需要安装最新的 Xcode 16.0.0 Release Candidate 版本，推荐使用 [Xcodes.app](https://www.xcodes.app/) 来管理多个Xcode版本。

![xcodes](/images/extract-iphone-device-model/xcodes.png)

1、在Xcode的安装目录下面找到 `device_traits.db` 这个文件，路径如下（如果Xcode命名不一样按照实际情况修改）：

```
/Applications/Xcode-16.0.0-Release.Candidate.app/Contents/Developer/Platforms/iPhoneOS.platform/usr/standalone/device_traits.db
```

2、使用SQL查询，获取 iPhone 16 系列机型信息

```sql
SELECT * FROM Devices WHERE ProductDescription LIKE 'iPhone 16%'
```

![execute-sql](/images/extract-iphone-device-model/execute-sql.png)

> 其中 `ProductType` 字段对应的是机型，`ProductDescription` 字段对应的是机型名称。

查询结果如下：

```
iPhone17,1 : iPhone 16 Pro
iPhone17,2 : iPhone 16 Pro Max
iPhone17,3 : iPhone 16
iPhone17,4 : iPhone 16 Plus
```

写了一个简单的脚本（根据自己的需要调整一下参数，参考[这里](https://gist.github.com/adamawolf/3048717?permalink_comment_id=4863478#gistcomment-4863478)实现）：

```shell
#!/bin/bash 

set -e

XCODE_APP="$(ls /Applications | grep Xcode- | sort -r | head -n 1)"
PREFIX="iPhone 16"
DEVICE_TRAITS_DATABASE="/Applications/${XCODE_APP}/Contents/Developer/Platforms/iPhoneOS.platform/usr/standalone/device_traits.db"
SQL_QUERY="SELECT DISTINCT CASE WHEN INSTR(ProductType, '-') > 0 THEN SUBSTR(ProductType, 1, INSTR(ProductType, '-') - 1) ELSE ProductType END AS DeviceType, ProductDescription AS DeviceDescription FROM DEVICES WHERE ProductDescription LIKE '${PREFIX}%';"

echo "DeviceType : DeviceDescription"
sqlite3 -readonly -separator $'\t' $DEVICE_TRAITS_DATABASE "$SQL_QUERY" | awk -F '\t' 'NR>0 { print ""$1" : "$2"" }'
echo "-----------------------------"
```

### 参考资料

- 1、[Apple_mobile_device_types.txt](https://gist.github.com/adamawolf/3048717)

- 2、[List of Apple products](https://en.wikipedia.org/wiki/List_of_Apple_products)

- 3、[Models](https://theapplewiki.com/wiki/Models#iPhone)