title: "[Pointer] Simple concept"
date: 2015-01-29 23:10:14
categories: 
- pointer
tags:
- C
- pointer
toc: true
---

# (&\*ptr) 等於 (\*&ptr) ??

最近看到有人在問這個問題, 所以就簡單分析一下這個概念. 

```
int a = 10, *ptr = &a;
&*ptr 等於 &(*ptr) ??
// 假如 a 所在的位置是0x7fffffffd9e4
// 假如 ptr 的位置是0x7fffffffd9e8
```
因式分解 

___&*ptr___
1. *ptr  ->  這個指的是pointer ptr 所指向位置的值, 也就是10.
2. &(*ptr) = &(a) -> 也就是a的位置 0x7fffffffd9e4

___*&ptr___

1. &ptr -> pointer ptr 的位置0x7fffffffd9e8
2. *&ptr = *(&ptr) = *(0x7fffffffd9e8) = (將這個位置的值取出來) = &a = 0x7fffffffd9e4

因此
```
&*ptr 等於 *&ptr 
```


