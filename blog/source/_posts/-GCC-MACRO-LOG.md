title: "[GCC][MACRO-LOG]"
date: 2015-01-29 08:21:34
categories: GCC
tags:
- GCC
toc: true
---

以前一直是用printf來debug, 每次都寫差不多格式的log,
所以去查了一下, 在巨集裏面要如何把default foramt 添加上去,
是時後要學點GDB了.... XD.

```
#define DBG(fmt, vars...) printf("[%s][%d] "fmt, __FUNCTION__, __LINE__, ##vars)

```
