---
title: '利用 iTextSharp 去除 PDF 水印'
slug: 'pdf-remove-watermark'
date: 2024-09-10
tags: [.NET, PDF]
toc: true
summary: 当临时使用一些收费库处理pdf时，由于未获得授权，这些库会在 PDF 文件中添加水印。本文介绍如何移除这些水印。
---

当临时使用一些收费库处理pdf时，由于未获得授权，这些库会在 PDF 文件中添加水印。本文介绍如何移除这些水印。

```cs
    private void RemoveFirstLineText(string sourceFile, string outputPdf)
    {
        using (PdfReader reader = new PdfReader(sourceFile))
        using (PdfStamper stamper = new PdfStamper(reader, new FileStream(outputPdf, FileMode.Create)))
        {
            var content = stamper.GetOverContent(1);
            content.SetColorFill(BaseColor.WHITE);
            var pageSize = reader.GetPageSize(1);

            content.Rectangle(0, pageSize.Height - 30, pageSize.Width, 30);
            content.Fill();
        }
    }
```

原理是用 iTextSharp.text 创建一个白色矩形，覆盖在第一页的顶部，从而移除水印。比较奇怪的时，方位是从下往上计算的，所以顶部的高度是一页的高度减去边距。这方法只适合背景颜色是纯色的地方。