title: "[C][Dynamic Array] Exercise to operate the dyanmic array "
date: 2015-03-21 15:28:52
categories:
- Dynamic Array
tags:
- C
- Dynamic Array
- realloc
toc: true
---


## 目的

很久之前買了一本書, 有關於系統程式師的養成計劃, 但是一直沒有很他認真的實行...

哈, 最近又開始了新生活運動, 打算每個禮拜花一點時間upgrade一下技能, 已經打算

靠着寫程式過我下半生了, HA....今天看到了一個有趣的議題, 資料結構裏面有 雙向鏈結以及dynamic array 這種東西, 

我沒學過自料結構,但是雙向鏈結在interview的時候, 常會被問, 但是 dynamic array 這個我還是第一次各聽到

（是不是很虛， 沒辦法, 小弟不是資工的...走一步算一步吧!!） 

今天就花了時間, 學習了一下要如何操作 dynamic array... 

## 結論

先來下個結論, 一開始我以爲dynamic array 是指array 可以動態的改變, 將底下的array, 大小改變...

```
int static_array[10];
```

但是我做了實驗, 發現是不行的(有錯的話, 請糾正我吧！！) 

基本上就是透過 malloc 要一塊heap 的記憶體,然後再用 realloc 的方式, 去放大縮小你的heap空間.

底下的sample 主要做了底下這些事情. 一行一行做註解， 會殺了我...所以我列出concept, 讓入門人有個概念


1. 我用一個structure -->  DArray, 來當作動態的array. 主要就是操作裏面的data
2. 這個sample code有讓array長大的api, 以及讓array所小的api, 可以參考.h 檔去追
3. DArray 裏面有兩個siz,
      int array_size;           --> data array 的大小
      int allocated_array_size; --> 我們用 malloc 實際跟系統要的memory size
4. 爲什麼要分兩個size呢, 原因是因爲跟記憶體有關, 你應該不會希望, 當我們要新增刪除一個member 就要
   call一次 realloc 吧...這樣會造成你的memory不連續. 就失去了dynamic array 的好處之一...

## Sample Code

___main.c___

```
/*******************************
 *  Purpose : Exercise to operate the dynamic array
 *  Arthur  : Sheldon Peng
 *  Date    : 2015/3/21 
 * */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "main.h"

#define SR_DBG(fmt, vargs...) printf("[%s][%d]" fmt, __FUNCTION__, __LINE__, ##vargs)

#define CHECK_NULL(PTR)  if(!PTR)    \
                          {printf("ERROR!! the pointer is NULL \r\n");} 


static RET DArray_shrink(DArray* thiz, int shrink_size)
{
  CHECK_NULL(thiz);
  void* shrink_data = realloc((void*)thiz->data, shrink_size*sizeof(char));
  
  if ( NULL!=shrink_data)
  {
    thiz->data = (char*)shrink_data; 
    thiz->allocated_array_size = shrink_size; 
    SR_DBG(" SHRINK size to %d \r\n", shrink_size);
  }
  else
  {
    SR_DBG("FAIL to realloc array \r\n");
    return RET_FAIL; 
  }

  return RET_SUCCESS;

}

RET DArray_delete(DArray* thiz, int index)
{
  CHECK_NULL(thiz);
  
  if (index > thiz->array_size)
  {
    SR_DBG(" ERROR to delete the dynamic array member beacuse the index is not belong into dynamic array \r\n");
    return RET_FAIL;
  }

  // remove the member from the dynamic array
  int i;
  for (i=index;i<(thiz->array_size-1);i++)
  {
    thiz->data[i] = thiz->data[i+1];
  }
  thiz->array_size = thiz->array_size-1;
  
  // Shrink the dynamic array if the pool size is enough to use .
  int shrink_size = thiz->array_size + (thiz->array_size>>1);

  if (shrink_size < thiz->allocated_array_size)
  {
     if ( RET_FAIL == DArray_shrink(thiz, shrink_size) )
     {
       SR_DBG("FAILLL to shrink the dynamic array \r\n");
     }
  }
}

static RET DArray_enlarge(DArray* thiz, int enlarge_size)
{
  CHECK_NULL(thiz);
  void* enlarge_data = realloc((void*)thiz->data, enlarge_size*sizeof(char));

  if ( NULL!=enlarge_data )
  {
    thiz->data = (char*)enlarge_data;
    thiz->allocated_array_size = enlarge_size;
    SR_DBG(" ENLARGE size to %d \r\n", enlarge_size);
  }
  else
  {
    SR_DBG("FAIL to realloc array \r\n");
    return RET_FAIL; 
  }

  return RET_SUCCESS;
}

RET DArray_insert(DArray* thiz, int index, char insert_data)
{
  CHECK_NULL(thiz);

  if (index > thiz->array_size)
  {
    SR_DBG("Your index is wrong, because the index is not wihin the array \r\n");
    return RET_FAIL;
  }

  if (thiz->allocated_array_size < thiz->array_size+1)
  {
    SR_DBG("Because the space is not enough, so we need to enlarge the space for dynamic array \r\n");
    int enlarge_size = (thiz->array_size>>1) + thiz->array_size;
    if (RET_FAIL == DArray_enlarge(thiz, enlarge_size ))
    {
      SR_DBG("FAILLL to enlarge the space of dynamic array \r\n");
      return RET_FAIL;
    }
  }
  
  int i;
  for ( i=thiz->array_size; i>index; i--)
  {
    thiz->data[i] = thiz->data[i-1];
  }
  
  thiz->data[index] = insert_data;
  thiz->array_size = thiz->array_size+1;
}

void Display_DArray(DArray* DArray_ptr)
{
  int i;
  int size = DArray_ptr->array_size;

  SR_DBG("My Dynamic Array is : \r\n");
  SR_DBG("size is %d \r\n", size);
  SR_DBG("allocated size is %d \r\n", DArray_ptr->allocated_array_size);

  for ( i=0; i<size; i++)
  {
      printf("%d, ", DArray_ptr->data[i]);
  }
  printf("\r\n");
}

void main(void)
{
  // Init my dynamic array
  DArray myDArray;
  myDArray.array_size = MINIMUN_ARRAY_SIZE;
  myDArray.allocated_array_size = MINIMUN_ARRAY_SIZE+ (MINIMUN_ARRAY_SIZE>>1); 
  
  //char InitArray[myDArray.array_size];
  myDArray.data = (char*)malloc(sizeof(char)*myDArray.array_size);
  memset((void*)myDArray.data, 0x0, (myDArray.array_size)*sizeof(char));

  // Display all dynamic array members. 
  Display_DArray(&myDArray);

  // test
  SR_DBG("======== TEST for ENLARGing D_ARRAY \r\n");
  int i;
  for (i=0;i<10;i++)
  {
    DArray_insert(&myDArray, 0, i);
  }

  for (i=0;i<10;i++)
  {
    DArray_insert(&myDArray, 5, i);
  }
  Display_DArray(&myDArray);
  SR_DBG("======== TEST for SHRINking D_ARRAY \r\n");
  DArray_delete(&myDArray, 1);
  DArray_delete(&myDArray, 1);
  DArray_delete(&myDArray, 1);
  DArray_delete(&myDArray, 1);
  DArray_delete(&myDArray, 5);
  DArray_delete(&myDArray, 6);
  Display_DArray(&myDArray);
}
```

___main.h___

```
#define MINIMUN_ARRAY_SIZE 1

typedef enum
{
  RET_SUCCESS,
  RET_FAIL,
}RET;

typedef struct _DArray
{
  int array_size;
  int allocated_array_size;
  char* data;
}DArray;

RET DArray_insert(DArray* thiz, int index, char insert_data);
RET DArray_delete(DArray* thiz, int index);

```

