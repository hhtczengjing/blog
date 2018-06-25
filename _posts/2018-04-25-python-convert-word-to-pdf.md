---
layout: post
title: "使用Python批量将Word文档转换为PDF"
date: 2018-04-25 11:46:29 +0800
comments: true
categories: Note
---

最近一直在整理数据，刚好有一批Word文档需要批量另存为PDF格式的文档，使用`win32com`操作Word，写了个Python的脚本用于批量进行转换。

### 1、环境准备

#### (1) 安装

```
pip install pywin32
```

#### (2) 初始化

由于我的机器上面安装的是`Office 2010`, 安装完成`pywin32`之后，进入到Python安装路径`\Lib\site-packages\win32com\client`的目录下面执行如下代码：

```
python makepy.py -d "Microsoft Word 14.0 Object Library"
```

#### (3) 引用模块

```
import os, sys
from win32com.client import Dispatch, constants
```

### 2、关键代码

#### （1）遍历目录获取全部的Word文档

```
def fetchAllFile(path):
    files = []
    for dirpath, dirnames, filenames in os.walk(path):
        for file in filenames:
            ext = os.path.splitext(file)[1].lower()
            if ext == '.docx' or ext == '.doc':
                fullpath = os.path.join(dirpath, file)
                files.append(fullpath)
    return files
```

### （2）将Word转换为PDF

```
def convertWordToPdf(docxPath, pdfPath):
    w = Dispatch("Word.Application")
    try:
        doc = w.Documents.Open(docxPath, ReadOnly=1)
        doc.ExportAsFixedFormat(pdfPath, constants.wdExportFormatPDF, Item=constants.wdExportDocumentWithMarkup, CreateBookmarks=constants.wdExportCreateHeadingBookmarks)
    except Exception, e:
        print e
    finally:
        w.Quit(constants.wdDoNotSaveChanges)
```

> 完整代码: https://github.com/hhtczengjing/convert-word-to-pdf