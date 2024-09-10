---
title: 'C# 利用 Spire 替换 PDF 文件的指定文字'
slug: 'spire-replace-pdf-text'
date: 2024-09-10
tags: [.NET, PDF]
toc: true
summary: 本文介绍对于已导出的pdf文件，使用 Spire 替换其中的指定文字。
---


找了多个C#库，只有 Spire 的 PdfTextReplacer 类比较好用，能直接替换整段文本。

```cs
    private void ReplaceText(string sourceFile, string outputPdf)
    {
        // 创建PdfDocument 对象
        Spire.Pdf.PdfDocument doc = new();

        // 加载PDF 文件
        doc.LoadFromFile(sourceFile);

        // 获取特定的页面
        PdfPageBase page = doc.Pages[0];

        // 根据页面创建 PdfTextReplacer 对象
        PdfTextReplacer textReplacer = new PdfTextReplacer(page);

        // // 创建PdfTextReplaceOptions 对象
        // PdfTextReplaceOptions textReplaceOptions = new PdfTextReplaceOptions();

        // // 指定文本替换的选项
        // textReplaceOptions.ReplaceType = PdfTextReplaceOptions.ReplaceActionType.AutofitWidth;
        // textReplacer.Options = textReplaceOptions;

        // 将所有目标文本的出现替换为新文本
        textReplacer.ReplaceAllText("{{username}}", "John Doe");
        textReplacer.ReplaceAllText("{{date}}", "2024-Sept");

        // 将文档保存到另一个 PDF 文件中
        doc.SaveToFile(outputPdf);

        // 释放资源
        doc.Dispose();

    }
```

未授权版本会有水印，可以用另一篇文字介绍的方法去除，或购买正版授权。