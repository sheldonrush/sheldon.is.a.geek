title: "[C][ELF] Attribute Section"
date: 2015-03-13 00:58:10
categories: ELF 
tags:
- C
- ELF
- Attribute Section
- readelf
toc: true
---

## 目的
這幾天看到有人的code, 在資料或是function前面多加 attribute 的關鍵字

所以很好奇， 這東東是幹什麼用的. 花了一點時間讀了相關的[blog](http://enginechang.logdown.com/posts/248172-linker-loader-library), 自己消化了一下, 寫了測試的code,  來驗證一下. 

自定section 的目的,  應該是要提高cache 的使用效率. 這樣的話, 但是相對的

你要code size會變大,  畢竟你多了section header 的資料. 

## Background
在開始前可能要先明白什麼叫做section, 什麼是objuct code, 什麼ELF檔, 

objuct code 就是我們.c 檔compiler 過後的檔案， 假如在透過linker 的話, 

就會將所有的.o檔, reloate 成elf. 

一個.o 檔其實就可以把他拆解成許多的section. 相同特性的資料, 就會擺成相同的secion上面. 

那我們是否可以自定自己的section 呢, 答案是可以的, 讃阿～～

就是透過attribute 來告知compiler, 我要自定自己的section.

## 快接近重點了XDD

首先我寫了一個程式, 兩個file, 每個file, 任一個function.

其實只是去印 global array 的值,  只是我將 int_array 放到我自定的section當中, 

我取名成 .sheldon.data 

___main.c___

```
//***********************
// Arthur : Sheldon Peng
// date : 2015/03/12
//************************

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define array_size 5
__attribute__((section(".sheldon.data"))) int int_array[array_size] = {0,1,2,3,4};

char test_bss[1024];

void main(void)
{
  int size = array_size;
  int_array[size-1] = 5;

  memset(test_bss, 'a', sizeof(test_bss));

  printf("Hello, this is attribute section test \r\n");
  extern void printf_array(int*,int);
  printf_array(int_array, size);
}
```
___printf_array.c___

```
#include <stdio.h>

void printf_array(int* ptr, int size)
{
  printf("[%s] \r\n", __FUNCTION__); 
  int i = 0;
  for (i=0; i<size; i++)
  {
    printf("%d, ", ptr[i]);
  }
  printf("\n done \n");
}

```


___常用指令___
1. readelf -s   : 讀這個elf/object 的symbol 
2. readelf -S   : 讀這個elf/object 的Section
3. readelf -a   : 讀所有的資訊


## 來驗證看看囉!!

先將兩個 .c 檔個別的compiler

```
gcc -c print_array.c              // 產生 print_array.o (object)
gcc -c main.c                     // 產生 main.o (object)
gcc main.o print_array.o -o main  // 產生 main   (elf)

```

透過下 readelf -S 來觀察自定的section. 

是不是多了 .sheldon.data XDD
{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/readelf_S.png %}

其實還有非常多東西可以看, 像是還沒link前的symble 有什麼不一樣.

某些default 的section各別代表什麼意義. 要如何找出， 你這個elf 有哪些dependency 的lib...

非常非常多,  我也搞的物煞煞. 哈, 所以就走一步算一步.


