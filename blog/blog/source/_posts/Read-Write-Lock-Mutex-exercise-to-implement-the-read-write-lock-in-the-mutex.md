title: "[Read/Write Lock][Mutex] exercise to implement the read/write lock in the mutex"
date: 2015-04-13 01:11:59
categories:
- Mutex
- Read/Write Lock
tags:
- C
- Mutex
- Read/Write Lock
toc: true
---

# 目的

最近遇到一個問題，假如你現在程式裏面有許多thread,每個thread會存取相同的資料, 

一般來說,我們都會用mutex來做保護，避免讀資料的thread,再讀的時候,寫資料的thread就把資料改過了

以致於讀資料的thread,拿到錯誤的資訊.

但是假如程式裏面,大部分的thread都是讀資料,寫資料的thread其實不是很多,這樣的話,我們都用同一個mutex的話,

效率其實會下降,所以就有人提出了read/write lock,兩個鎖的方式,讀資料的thread們,除了第一個讀的thread要等以外

其他的都不用.利害吧XDD.

# Sample Code. 

這份sample code, 

1. 我將最原本的mutex_lock實作擺在
   basic_mutex_class.c, 
   而lock_interface.c 裡面有抽象的描述.

2. 有關 Read/Write Lock 的實作擺在 main.c. 
   其實也只是多生出兩個mutex, 稱作讀鎖以及寫鎖.
   在實作出讀寫鎖的使用方式.

___a. main,c___

```
#include <stdlib.h>
#include "lock_interface.h"
#include "basic_mutex_class.h"


Ret read_RW_lock_ipl(read_write_lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);
 
  // 再來Read Lock 來保護read_count
  if (api_my_lock(my_lock->RLock) == Ret_Success)
  {
    // 有thread 要讀的時候,read_count++, 記錄有幾個thread 還在用.
    my_lock->read_count++;

    // 嘗試獲得Write Lock, 禁止有人寫
    if (api_my_lock(my_lock->WLock) == Ret_Success)
    {
      // 記錄現在lock mode 爲read.
      my_lock->lock_mode = Read_Lock_Mode; 
    }
    // UnLock 掉read lock, 由於只有讀的時候,不需要lock read thread.
    if (api_my_unlock(my_lock->RLock) == Ret_Fail)
    {
      SR_ERR("\n");
      return Ret_Fail;
    }
  }
}

Ret write_RW_lock_ipl(read_write_lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);

  // 嘗試獲得Write Lock, 禁止有人寫
  if (api_my_lock(my_lock->WLock) == Ret_Success)
  {
      // 記錄現在lock mode 爲write.
      my_lock->lock_mode = Write_Lock_Mode;
      return Ret_Success;
  }
  return Ret_Fail;
}

Ret un_RW_lock_ipl(read_write_lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);

  if (my_lock->lock_mode == Read_Lock_Mode)
  {
    // 現在是讀的情況,需要要解鎖
    // 用read lock 來保護read_count
    if (api_my_lock(my_lock->RLock) == Ret_Success)
    {
      //檢查 read_count 是否爲0
      my_lock->read_count--;
      if (my_lock->read_count == 0)
      {
        // 由於現在已經沒有讀資料的thread了, 所以就是釋放Write Lock
        if (api_my_unlock(my_lock->WLock)==Ret_Fail)
        {
          SR_ERR("\n");
          return Ret_Fail;
        }
      }
      if (api_my_unlock(my_lock->RLock) == Ret_Fail)
      {
        SR_ERR("\n");
        return Ret_Fail;
      }

      SR_DBG("Success to unlock \n");
      return Ret_Success;
    }
  }
  else if (my_lock->lock_mode == Write_Lock_Mode)
  {
    // 寫的thread, 結束以後，解鎖
    if (api_my_unlock(my_lock->WLock)==Ret_Fail)
    {
      SR_ERR("\n");
      return Ret_Fail;
    }
    return Ret_Success;
  }
  else
  {
    SR_ERR("Lock Mode is Non, there is something wrong.... ");
  }
}

Ret destroy_RW_lock_ipl(read_write_lock* my_lock)
{

  check_pointer_return_value(my_lock, Ret_Fail);
  free(my_lock->RLock);
  free(my_lock->WLock);
  free(my_lock);
  SR_DBG("Success to destory read/write lock \n");
}

read_write_lock* create_read_write_lock(void)
{
  read_write_lock* my_read_write_lock = malloc(sizeof(read_write_lock));

  check_pointer_return_value(my_read_write_lock, NULL);

  my_read_write_lock->read_count = 0;
  my_read_write_lock->RLock = (My_Lock*)Init_Mutex_Pthread();
  my_read_write_lock->WLock = (My_Lock*)Init_Mutex_Pthread();
  my_read_write_lock->lock_mode = None_Lock_Mode; 
  my_read_write_lock->read_lock_Fun = read_RW_lock_ipl; 
  my_read_write_lock->write_lock_Fun = write_RW_lock_ipl;
  my_read_write_lock->un_read_write_lock = un_RW_lock_ipl;
  my_read_write_lock->destroy_read_write_lock = destroy_RW_lock_ipl;

  return my_read_write_lock;
}

void main(void)
{

  read_write_lock* my_RW_lock = create_read_write_lock();
  // Test for Read lock
  my_RW_lock->read_lock_Fun(my_RW_lock);
  my_RW_lock->un_read_write_lock(my_RW_lock);

  // Test for write lock
  my_RW_lock->write_lock_Fun(my_RW_lock);
  my_RW_lock->un_read_write_lock(my_RW_lock);

  my_RW_lock->destroy_read_write_lock(my_RW_lock);

}

```

___b.basic_mutex_class.h___

```
#include <pthread.h>

My_Lock* Init_Mutex_Pthread(void);
Ret Mutex_Pthread_lock(My_Lock* );
Ret Mutex_Pthread_unlock(My_Lock* );
Ret Mutex_Pthread_destory(My_Lock* );

typedef struct _mutex_info
{
  pthread_mutex_t mutex;
}mutex_info;


```

___c.basic_mutex_class.c___

```
#include <stdlib.h>
#include "lock_interface.h"
#include "basic_mutex_class.h"

Ret Mutex_Pthread_lock(My_Lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);
  
  mutex_info* my_mutex = (mutex_info*)my_lock->lock_info;

  if (pthread_mutex_lock(&my_mutex->mutex) == 0)
  {
    return Ret_Success;
  }
  else
  {
    SR_ERR("\n");
  }
}

Ret Mutex_Pthread_unlock(My_Lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);
  
  mutex_info* my_mutex = (mutex_info*)my_lock->lock_info;

  if (pthread_mutex_unlock(&my_mutex->mutex) == 0)
  {
    return Ret_Success;
  }
  else
  {
    SR_ERR("\n");
  }
}

Ret Mutex_Pthread_destory(My_Lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);
  
  mutex_info* my_mutex = (mutex_info*)my_lock->lock_info;

  if (pthread_mutex_destroy(&my_mutex->mutex) == 0)
  {
    free(my_lock);
    return Ret_Success;
  }
  else
  {
    SR_ERR("\n");
  }
}

My_Lock* Init_Mutex_Pthread(void)
{
  My_Lock* my_lock = malloc(sizeof(My_Lock)+sizeof(mutex_info));
  check_pointer_return_value(my_lock, NULL);

  mutex_info* my_mutex;
  my_mutex = (mutex_info*)my_lock->lock_info;
  pthread_mutex_init(&(my_mutex->mutex),NULL);

  my_lock->DoLock = Mutex_Pthread_lock;
  my_lock->DoUnlock = Mutex_Pthread_unlock;
  my_lock->DoDestory = Mutex_Pthread_destory; 
  
  return my_lock;

}

```

___d.global_enum_macro.h___

```
#include <stdio.h>

// Define GLobal Enum
typedef enum _Ret
{
  Ret_Success,
  Ret_Fail,
  Ret_None,
}Ret;

// Define Global Macro
#define SR_DBG(fmt,args ...) printf("[Sheldon debug][%s][%d] "fmt,__FUNCTION__,__LINE__,##args)
#define SR_ERR(fmt,args ...) printf("ERROR !!! [%s][%d ] "fmt, __FUNCTION__,__LINE__,##args)
#define check_pointer_return_value(ptr,value) if(ptr==NULL) \
                                                  { printf("ERROR!!! #ptr is NULL, return #value \r\n"); \
                                                  return value;} \




```

___e.lock_inter_face.h___

```
#include "global_enum_macro.h"

// For basic lock
struct _my_lock;
typedef struct _my_lock My_lock;

typedef Ret (*LockFun)(My_lock*);
typedef Ret (*UnLockFun)(My_lock*);
typedef Ret (*DestoryLockFun)(My_lock*);

typedef struct _my_lock
{
  LockFun DoLock;
  UnLockFun DoUnlock;
  DestoryLockFun DoDestory;
  char lock_info[0];
}My_Lock;


// For read/write lock
struct _read_write_lock;
typedef struct _read_write_lock read_write_lock;
typedef Ret (*ReadLockFun)(read_write_lock*);
typedef Ret (*WriteLockFun)(read_write_lock*);
typedef Ret (*UnReadWriteLockFun)(read_write_lock*);
typedef Ret (*DestroyReadWriteLockFun)(read_write_lock*);

typedef enum _ReadWriteLockMode
{
  Read_Lock_Mode,
  Write_Lock_Mode,
  None_Lock_Mode,
}ReadWriteLockMode;


typedef struct _read_write_lock
{
  int read_count;
  My_Lock* RLock;
  My_Lock* WLock;

  ReadWriteLockMode lock_mode;
  ReadLockFun read_lock_Fun;
  WriteLockFun write_lock_Fun;
  UnReadWriteLockFun un_read_write_lock;
  DestroyReadWriteLockFun destroy_read_write_lock;
}read_write_lock;

Ret api_my_lock(My_Lock* my_lock);
Ret api_my_unlock(My_Lock* my_lock);
Ret api_my_destory(My_Lock* my_lock);

```

___f.lock_interface.c___

```
#include "lock_interface.h"


Ret api_my_lock(My_Lock* my_lock)
{
 check_pointer_return_value(my_lock, Ret_Fail);
 if (my_lock->DoLock(my_lock) == Ret_Fail) 
 {
    SR_ERR("\n");
 }
 return Ret_Success;
}

Ret api_my_unlock(My_Lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail); 
  if (my_lock->DoUnlock(my_lock) == Ret_Fail)
  {
    SR_ERR("\n");
  }
  return Ret_Success;
}

Ret api_my_destory(My_Lock* my_lock)
{
  check_pointer_return_value(my_lock, Ret_Fail);
  
  if (my_lock->DoDestory(my_lock) == Ret_Fail)
  {
    SR_ERR("\n");
  }
  return Ret_Success;
}

```

