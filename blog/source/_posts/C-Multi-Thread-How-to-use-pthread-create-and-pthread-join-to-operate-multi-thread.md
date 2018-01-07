title: "[C][Multi-Thread] How to use pthread_create and pthread_join to operate multi-thread"
date: 2015-03-23 21:22:47
categories:
- Multi-Thread
tags:
- C
- pthread_create
- pthread_join
- multi-thread
toc: true
---

# Purpose

今天在練習如何使用pthread_create和pthread_join來練習multi-thread的操作.

multi-thread 的code的好處, 在於你假如用multi-process 的話, CPU會在去多做context-switch,

整體的效能下降. 但是由於每個thread都共用相同的memory 大小區塊, 所以有可能造成

每個thread access 到其他人thread要用的資料, 底下有兩個sample code, 第一個sample code,

就出現這樣的錯誤.

# Compiler

```
gcc main.c -lpthread -o main
```

由於pthread是在libpthread.so的lib裏面, 所以我們在compiler 的時候, 必須將pthread lib include 進去. 

.c 檔裏面, 也記得要include pthread.h.

# Sample Code 1 

你可以從底下的結果看出一個奇特的現象,

我產生了10個thread, 但是印出thread id 的時候， thread id =5, 卻印了4次. 

這代表, 我create thread 的時侯, 帶下去的thread id (i), 被其他thread access到了. 造成了這個問題, 

修這個bug的最簡單方法, 可以參考Sample Code 2.

關於 pthread_join 的使用, 也很單純. 

他就wait 那裡,等待他要wait 的pthread id 被終止, 假如沒有終止的話, 那就GG了, 系統就會hang在那裏. 


___main.c___

```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define THREAD_NUM 10
#define SR_DBG(fmt,arg...) printf("[%s][%d] "fmt, __FUNCTION__,__LINE__, ##arg)

void* thread_routine(void* argv)
{
  int my_id = *((int*)argv);
  SR_DBG("hello, i'm new thread, my name is id[%d] \r\n",my_id);
  return;
}

int create_multi_thread_test(void)
{
  int i;

  pthread_t thread_id_array[THREAD_NUM];
 
  for (i=0; i<THREAD_NUM; i++)
  {
    if (0 != pthread_create(&thread_id_array[i], NULL, thread_routine, &i))
    {
      SR_DBG("OMG, create thread FAILLL!!! \r\n");
    }
  }

  for (i=0; i<THREAD_NUM; i++)
  {
    SR_DBG("Wait for termination of thread id [%d] \r\n", i);
    if (0 != pthread_join(thread_id_array[i], NULL))
    {
      SR_DBG("OMG, Mother thread cannot wait for thread id [%d]\r\n", i);
    }
  }
  

  return 0;
}

void main(void)
{
  SR_DBG("This is multi-thread test code \r\n");
  
  if (-1 == create_multi_thread_test())
  {
    SR_DBG("the multi thread test is FAILLLLL!!! \r\n");
  }

}

```

___Result1___

{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/pthread_v1.png %}

# Sample Code 2 #

這個smaple code, 就是針對上面去做一些改良, 多用一個 array 來存要帶進去thread_routine 裏面的引數

避免被其他thread access 倒.但是這一定不是最佳的方法... 

___main_v2.c___

```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define THREAD_NUM 10
#define SR_DBG(fmt,arg...) printf("[%s][%d] "fmt, __FUNCTION__,__LINE__, ##arg)

void* thread_routine(void* argv)
{
  int my_id = *((int*)argv);
  SR_DBG("hello, i'm new thread, my name is id[%d] \r\n",my_id);
  return;
}

int create_multi_thread_test(void)
{
  int i;

  pthread_t thread_id_array[THREAD_NUM];
  int thread_i[THREAD_NUM];
 
  for (i=0; i<THREAD_NUM; i++)
  {
    thread_i[i] = i;
    if (0 != pthread_create(&thread_id_array[i], NULL, thread_routine, &thread_i[i]))
    {
      SR_DBG("OMG, create thread FAILLL!!! \r\n");
    }
  }

  for (i=0; i<THREAD_NUM; i++)
  {
    SR_DBG("Wait for termination of thread id [%d] \r\n", i);
    if (0 != pthread_join(thread_id_array[i], NULL))
    {
      SR_DBG("OMG, Mother thread cannot wait for thread id [%d]\r\n", i);
    }
  }
  

  return 0;
}

void main(void)
{
  SR_DBG("This is multi-thread test code \r\n");
  
  if (-1 == create_multi_thread_test())
  {
    SR_DBG("the multi thread test is FAILLLLL!!! \r\n");
  }

}
```

___Result2___

{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/pthread_v2.png %}
