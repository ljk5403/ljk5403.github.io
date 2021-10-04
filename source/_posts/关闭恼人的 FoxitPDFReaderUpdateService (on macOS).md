---
title: 关闭恼人的 FoxitPDFReaderUpdateService (on macOS)
date: 2021-10-04 14:59:46
categories:
  
tags:
---

# 关闭恼人的 FoxitPDFReaderUpdateService (on macOS)



最近发现 `FoxitPDFReaderUpdateService` 进程会占用大约5%的CPU，同时带动 `opendirectoryd` 占用10%的CPU，而且不会自动退出，十分的离谱。

遂去网上寻找教程，发现了这个解释：[macOS: FoxitPDFReaderUpdateService consumes unacceptable amount of CPU](https://forums.foxitsoftware.com/forum/portable-document-format-pdf-tools/foxit-phantompdf/182946-macos-foxitpdfreaderupdateservice-consumes-unacceptable-amount-of-cpu)，应该是一个bug，符合我的特征，不过文件夹不太正确，本人电脑上的位置为：`~/Library/Application Support/Foxit Software/Addon/Foxit PDF Reader`

运行 


```bash
mv FoxitPDFReaderUpdateService.app FoxitPDFReaderUpdateService.app.bak
```

 之后似乎不会自动启动了。