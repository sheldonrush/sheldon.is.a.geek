title: "How to call C++ class member function from C"
date: 2015-06-27 06:18:41
categories:
- C
tags:
- C
- C++
- class
toc: true
---

#Purpose

整個晚上都睡不着,大概是因爲今天被一個無心的朋友傷到自信心

難道物理系不能寫出好的code嗎,雖然真的也是有一段時間沒有磨練自己了.

爲了捍衛自己的尊嚴,還是必須要嚴守每天寫點程式的習慣.

今天想到一件事情,要如何從C語言去呼叫C++的class member function.

這件事情,感覺挺有趣的,所以花了點時間看看要如何實作.

#HOW TO DO 

概念是這樣滴, C的code是無法直接呼叫C++ class的member,

必須要透過一層wrapper 層的來變相取得class的member, 要注意的是, 這個wrapper 層也必須是C++,

你會發現, 其實這個wrapper 只是利用物件的poiner 去操作, class member.

底下的例子，我會定一個class, 然後再把這個class 包成一個static lib.

接着就會用C code去呼叫他.

## core.cpp

如上面所說,我定義一個class並且放在core.cpp 裏面,裏面只有一個member function,

唯一的功用就是將兩個數字相加後.印出來.

```
#include <iostream>

using namespace std;

class MyCPP {
  public : 
    void PrintNumber(int num1, int num2); 
};

void MyCPP :: PrintNumber(int num1, int num2)
{
  cout << (num1+num2) <<endl;

}
```

## wrapper.h

再來就是實作wrapper層的api,這層非常的有意思,先來講一下這個header file,

由於c++的compiler對於symbol有Name Mangling的行爲(也就是會改變數函式名稱)

所以假如我們wrapper這層的api,要同時被C以及C++使用的話,我們必須讓g++ compiler symbol不去做name mangling.

要怎麼做呢？就是透過 extern "C" 這個關鍵字.

爲了避免gcc compiler 在include wrapper.h 出現compiler error, 我們再用cplusplus去隔開.
;
還有一件事情要注意壓,由於是要給C code 使用,但是C一定沒有class 這種結構,所以就會將class的poiner

先暫時指向void

```
#ifdef __cplusplus
extern "C" {
#endif
    void* api_create_class(void);
    void api_destory_class(void*);
    void api_print_sum(void*, int, int);
#ifdef __cplusplus
}
#endif
```

## wrapper.cpp

這三個wrapper api 的實作,非常容易.比較要注意的是,C++ 的形態轉換,是用 static_cast<CLASS>

```
#include "core.cpp"
#include "wrapper.h"

void* api_create_class(void)
{
  return new MyCPP();
}

void api_destory_class(void* thiz) 
{
  delete static_cast<MyCPP*>(thiz);
}

void api_print_sum(void* thiz, int num1, int num2)
{
  static_cast<MyCPP*>(thiz)->PrintNumber(num1,num2);
}

```

## main.c 

再來就是要進入測試成果的部分囉, 

先include wrapper.h, 然後在使用他囉!!!!

```
#include <stdio.h> 
#include "wrapper.h"

int main(int argc, char** argv)
{
  printf("Hello, This sample code teach us how to call CPP class function from C \r\n");
  
  printf(" New a Class \r\n");
  void* MyClass = api_create_class();

  printf(" Print the Sum of Numbers \r\n");
  api_print_sum(MyClass, 10, 20);

  printf(" Delete the Class \r\n");
  api_destory_class(MyClass);

  return 0;
}
```


# How To Compiler 

挖勒,剛講這麼多,到底要怎麼編code呀???

接下來就是要介紹如何 compiler. 

假如你都有照着作下來的話,你應該會有底下這些檔案. 

```
core.cpp  main.c  wrapper.cpp  wrapper.h
```

## 製作C++靜態lib 

```
g++ wrapper.cpp core.cpp -c    // 你換發現多了兩個 .o 檔
ar rvs libMyCPP.a wrapper.o core.o
```

## 製作C (main) 的object code

```
gcc main.c -c      // 你會發現多了一個main.o
```

## 利用g++ 去做LINK 

```
g++ main.o libMyCPP.a -o test
```

## 驗證成果 !!!

```
./test
```

```
Hello, This sample code teach us how to call the static of CPP from C 
New a Class 
Print the Sum of Numbers 
30
Delete the Class
```




