title: "[C][uinstd] 簡易使用 readlink "
date: 2015-03-18 00:56:50
categories:
- C
- uinstd
tags:
- C
- readlink
- ls
toc: true
---

## 目的

最近遇到一個問題, 假如我希望我的程式能夠知道某個symbolic link 的指向. 

但是我又不希望用 fork+exec or system 的方式, 去產生另一個process 來執行"ls -l"

直接單純的使用 ___uinstd.h___ 裏面的readlink來得到link的指向.

看起來fork+exec 的比較有趣困難 XDD, 過幾天再寫一下心得...哈

## Sample Code

___1. 產生一個symbolic link___

什麼是symbolic link呢？熟稱soft link, 有點像windows 底下的聯結, google一下巴...

```
sheldon$ ln -s ~/python link_test
```

我產生了一個link 稱作link_test, 這個link指向 ~/python

___2. example code___

___main.c___

```
//*****************************
// Purpose : this is an sample code to use the api of readlink
// Arthur : Sheldon Peng
// Date : 2015/3/18
//***************************

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

void main(void)
{
  char link_path[] = "/home/sheldon/programming/c_language/readlink/link_test";
  char target_path[256];
  memset(target_path, 0, sizeof(target_path));

  if ( -1 == readlink(link_path, target_path, sizeof(target_path)))
  {
    printf("Fail to read link path !!\r\n");
  }▫
  else
  {
    printf("link path is %s \r\n", target_path);
  }

}

```
