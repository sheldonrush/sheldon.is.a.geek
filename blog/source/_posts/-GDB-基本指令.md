title: "[GDB] 基本指令"
date: 2015-02-04 08:22:05
categories: 
- GDB
tags:
- GDB
toc: true
---

GDB 是一個很好的除錯指令, 這給了一個方法, 去快速知道你的程式發生了闍麼問題. 

而不要一直使用___printf___去做debug. gdb 可以用在 server/client 的online debug 模式,

或是僅僅offline 的觀察coredump. 

使用gdb 除錯, 你必須在編code的時候, 在gcc 的compiler option 多加___-g___

底下是我最近練習到的gdb指令.

```
(gdb) r     \\ 開始執行程式
(gdb) c     \\ 假如你現在停在breakpoint, 接着執行程式（continue） 
(gdb) b function  \\ 設定中斷點
(gdb) b file:line \\ 設定中斷點的另一個方法
(gdb) layout src  \\ 將現在程式跑到的位置,  跟src code做一個mapping.
(gdb) stack \\ 觀察現在的stack
(gdb) info all-registers \\ 觀察現在所有的registers
(gdb) p $r0      \\ 讀取現在的cpu $r0 registers
(gdb) threads    \\ 觀察program 裏面, 所有的thread 運行的情況卒. (有點像ps)
(gdb) s          \\ 單步執行, 遇到function的時候, 進入function的stack. 
(gdb) n          \\ 單步執行, 遇到function的時候, 不進入function的stack
(gdb) return     \\ 離開現在的fucntion, 返回calling
(gdb) jump line  \\ 將PC 指向程式的line的位置
(gdb) info locals \\ 列出現在的local變數
(gdb) info args \\列出引數
(gdb) bt \\ bracktrace

```
