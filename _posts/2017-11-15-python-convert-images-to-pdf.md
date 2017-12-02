---
layout: post
title: "使用Python合并图片生成PDF文件"
date: 2017-11-15 10:08
comments: true
tags: Note
---

最近做了一个小功能，将一个页面上面的所有图片下载下来生成一个PDF文件。发现了一个非常好用的库reportlab, pyPdf。只需要几行代码就能实现功能，如果没有安装可以通过pip安装：

```
pip install reportlab -i https://pypi.douban.com/simple
pip install pyPdf -i https://pypi.douban.com/simple
```
注: `-i`表示使用豆瓣的镜像服务

### 操作过程

下面记录下我的处理的过程：

（1）如果只是简单的需要将图片进行拼接成PDF可以直接使用如下的方式：

```
from reportlab.lib.pagesizes import A4, portrait, landscape
from reportlab.pdfgen import canvas

def convert_images_to_pdf(img_path, pdf_path):
    pages = 0
    (w, h) = portrait(A4)
    c = canvas.Canvas(pdf_path, pagesize = portrait(A4))
    l = os.listdir(img_path)
    l.sort(key= lambda x:int(x[:-4]))
    for i in l:
        f = img_path + os.sep + str(i)
        c.drawImage(f, 0, 0, w, h)
        c.showPage()
        pages = pages + 1
    c.save()
```

（2）如果需要根据不同的尺寸的图片设置横屏还是竖屏模式，可以考虑使用如下方式实现：

```
import os, shutil
from PIL import Image
from reportlab.lib.pagesizes import A4, portrait, landscape
from reportlab.pdfgen import canvas
from pyPdf import PdfFileWriter, PdfFileReader

def convert_image_to_pdf(img_path, pdf_path):
    img = Image.open(img_path)
    (w0, h0) = img.size
    if w0 > h0:
        (w, h) = landscape(A4)
        c = canvas.Canvas(pdf_path, pagesize = landscape(A4))
        c.drawImage(img_path, 0, 0, w, h)
        c.showPage()
        c.save()
    else:
        (w, h) = portrait(A4)
        c = canvas.Canvas(pdf_path, pagesize = portrait(A4))
        c.drawImage(img_path, 0, 0, w, h)
        c.showPage()
        c.save()

def convert_images_to_pdf(img_path, pdf_path):
    pages = 0
    tmp_path = '.' + os.sep + 'temp'
    if not os.path.exists(tmp_path):
        os.mkdir(tmp_path)
    list = os.listdir(img_path)
    list.sort(key=lambda x:int(x[:-4]))
    output = PdfFileWriter()
    for item in list:
        img = img_path + os.sep + str(item)
        pdf = tmp_path + os.sep + str(pages + 1) + ".pdf"
        convert_image_to_pdf(img, pdf)
        input = PdfFileReader(file(pdf, "rb"))
        pageCount = input.getNumPages()
        pages = pages + 1
        for iPage in range(0, pageCount):
            output.addPage(input.getPage(iPage))
    outputStream = file(pdf_path, "wb")
    output.write(outputStream)
    outputStream.close()
    shutil.rmtree(tmp_path)
```

### 参考资料

1、[ReportLab PDF Library User Guide](https://www.reportlab.com/docs/reportlab-userguide.pdf)

2、[python合并PDF文件](http://blog.csdn.net/zhangchilei/article/details/49642761)