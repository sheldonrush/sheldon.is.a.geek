title: "[C][Mutex][Pthread][Interface] A sample code for pthread_mutex interface"
date: 2015-04-04 00:04:26
categories:
- Pthread_Mutex
tags:
- C
- Pthread
- Mutex
- Interface
toc: true
---

# Purpose

今天閱讀到一個章節，是在說明同步的，thread之間的同步一般都會提到mutex,

讓不同thread之間能夠共用share資源,但是又不會干擾對方的state, 就非常重要了. 

其中有個地方提到了，interface 的概念, 假如我的code是想要跑在各種平臺上面的話，我就應該將程式架構分成AP層, interface 層（API）, 

driver 層(偏硬體控制HW, 我們這邊就暫時不考慮它，embedded system 就很重要了) .

這邊的API層，提到了用大量的callback function,來達到跨平臺的需求,交由上層AP來決定,要運行在什麼平臺上.

底下的sample code, 是用pthread_mutex 來說明這個概念.

# The Concept of Sample Code

假如我們需要把mutex 這種東西實作成一個interface,要如何去做,又能滿足各種平臺.

___1. 抽象話___

   第一個要做的是讓mutex 變得抽象話, 實作的部分，都透過用calback的方式讓AP層來實作.
   這樣聽起來有點像是在C用struct 取代C++的class XDD... 
   可參考 mutex_interface.h 
```
struct _Locker 
{
  LockerLockFunc lock;
  LockerUnLockFunc unlock;
  LockerDestoryLockFunc destory;
  char lock_info[0];
};
```
   這邊定了三個function pointer, 還有一個lock_info 來定義我們這個lock的特性.
   char lock_info[0], 這個宣告其實是一個技巧, char lock_info[0]其實並沒有佔任何空間,
   假如你的私有資料長度是可變的,你就可以這樣宣告.
   
   後面在create lock 的時候,會有提到如何做這件事情,

___2. Create_Lock___

   爲什麼interface 沒有開出Create_Lock 的API給別任用呢, 書中提到, 這是因爲當你再做抽象話的描述時，你無法用
   抽象畫來產生一個真實的實體出來，這句話我到現在還沒體會XDD.

___3. Callback fucntion的引數都含有struct Locker___

   這個就是抽象話的好處,交由上層來決定我的實作方式，假如你是要用linux 的pthread_mutex 來實作的話. 
   請把這個東西的function pointer交給interface. 
   你看看mutex_interface.c 這個檔案,看官,是不是，多麼的清楚間單.

# Sample Code

___1.main.c___

```
#include <pthread.h>
#include "mutex_interface.h"

Ret PthreadLock(Locker* thiz);
Ret PthreadUnLock(Locker* thiz);
Ret PthreadDestory(Locker* thiz);


typedef struct _pthread_info
{
  pthread_mutex_t mutex_id;
} pthread_info;

Ret PthreadLock(Locker* thiz)
{
  check_ptr(thiz, Ret_Fail);

  pthread_info* pp_info = (pthread_info*)thiz->lock_info;
  if ( 0 == pthread_mutex_lock(&pp_info->mutex_id))
  {
    return Ret_Success;
  }
  else
  {
    SR_DBG("WTF!!!  ERROR !!!\r\n");
    return Ret_Fail;
  }

}

Ret PthreadUnLock(Locker* thiz)
{
  check_ptr(thiz, Ret_Fail);
  
  pthread_info* pp_info = (pthread_info*)thiz->lock_info;
  if ( 0 == pthread_mutex_unlock(&pp_info->mutex_id))
  {
    return Ret_Success;
  }
  else
  {
    SR_DBG("WTF!!! ERROR !!!\r\n");
    return Ret_Fail;
  }
}

Ret PthreadDestory(Locker* thiz)
{
  check_ptr(thiz, Ret_Fail);

  pthread_info* pp_info = (pthread_info*)thiz->lock_info;
  
  if ( 0 != pthread_mutex_destroy(&pp_info->mutex_id))
  {
    return Ret_Fail;
  }
  SR_DBG("FREE the mutex \n");
  free(thiz);
  return Ret_Success;
}

Locker* create_locker(void)
{
  SR_DBG("Start \n\r");
  // Allocate the memory for locker
  Locker* locker_thiz = malloc(sizeof(Locker) + sizeof(pthread_info));
  pthread_info* pp_info = (pthread_info*)locker_thiz->lock_info;

  if (NULL != locker_thiz)
  {
    // Init the callback function in class.
    locker_thiz->lock = PthreadLock;
    locker_thiz->unlock = PthreadUnLock;
    locker_thiz->destory = PthreadDestory;
    pthread_mutex_init(&(pp_info->mutex_id),NULL);
    return locker_thiz;
  }
  return NULL;
}


void main(void)
{
  Locker* myLocker ; 
  myLocker = create_locker();
  
  myLocker->destory(myLocker);
}


```
   
___2.mutex_interface.c___

```
#include "mutex_interface.h"

Ret locker_lock(Locker* thiz)
{
  check_ptr(thiz,Ret_Fail);

  return thiz->lock(thiz);
}

Ret locker_unlock(Locker* thiz)
{
  check_ptr(thiz,Ret_Fail);

  return thiz->unlock(thiz);
}

Ret locker_destory(Locker* thiz)
{
  check_ptr(thiz,Ret_Fail);

  return thiz->destory(thiz); 
}

```

___3.mutex_interface.h___

```
#include <stdio.h>
#include <stdlib.h>

struct _Locker;
typedef struct _Locker Locker;

typedef enum _Ret
{
  Ret_Success,
  Ret_Fail,
  Ret_None
}Ret;

typedef Ret (*LockerLockFunc)(Locker* thiz);
typedef Ret (*LockerUnLockFunc)(Locker* thiz);
typedef Ret (*LockerDestoryLockFunc)(Locker* thiz);

struct _Locker 
{
  LockerLockFunc lock;
  LockerUnLockFunc unlock;
  LockerDestoryLockFunc destory;
  char lock_info[0];
};


Ret locker_lock(Locker* thiz);
Ret locker_unlock(Locker* thiz);
Ret locker_destory(Locker* thiz);


#define check_ptr(ptr, var)  if(ptr==NULL)    \
                              { printf(" ["#ptr"] pointer is NULL\r\n");  \
                               return var; }  \

#define SR_DBG(fmt, arg...) printf("Sheldon Debug:[%s][%d]"fmt, __FUNCTION__, __LINE__, ##arg)

```

# How to compiler 

```
gcc -g main.c mutex_interface.c -lpthread -o main
```

# [GDB練習] 覺得有趣的事情,

不知道各位有沒有注意到底下這兩行程式.

```
  // Allocate the memory for locker
  Locker* locker_thiz = malloc(sizeof(Locker) + sizeof(pthread_info));
  pthread_info* pp_info = (pthread_info*)locker_thiz->lock_info;
```
這個地方就是剛剛提到的,若是一個可變的資料的話，要怎麼做呢.
分配locker_thiz指向的實體記憶體的時候，要多分配一塊 sizeof(pthread_info)，這個就是user自定的. 
看起來是不是很方便呢.XDDD

不知道大家有沒有宜個疑問，爲什麼locker_thiz->lock_info，是代表pp_info指向的位置呢.

底下是用gdb 觀察的一個結果. 

___1. 啓動GDB___

gdb main

___2. break 在 76行___

(gdb) b 76

___3. 執行程式___

(gdb) r

___4. 觀察變數的值___

```
(gdb) r
Starting program: /home/sheldon/programming/c_language/mutex_interface/main 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Sheldon Debug:[create_locker][64]Start 

Breakpoint 1, create_locker () at main.c:76
76      return locker_thiz;
(gdb) p locker_thiz->lock_info
$1 = 0x603028 ""
(gdb) p &locker_thiz->lock_info
$2 = (char (*)[]) 0x603028
(gdb) p *(locker_thiz->lock_info)
$3 = 0 '\000'
(gdb) p pp_info->mutex_id 
$4 = {__data = {__lock = 0, __count = 0, __owner = 0, __nusers = 0, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}}, 
  __size = '\000' <repeats 39 times>, __align = 0}
(gdb) p &pp_info->mutex_id 
$5 = (pthread_mutex_t *) 0x603028
```

locker_thiz->lock_info 其實就是會等於 locker_thiz+sizeof(Locker)

