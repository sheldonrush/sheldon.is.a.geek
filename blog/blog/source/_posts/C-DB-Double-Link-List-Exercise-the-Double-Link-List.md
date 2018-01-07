title: "[C][DB][Double Link List] Exercise the Double Link List"
date: 2015-05-02 21:20:04
categories:
- Double Link List
tags:
- C
- DB
- Double Link List
toc: true
---

# Purpose 

最近好奇要怎麼實作Stack 和 queue,看了一本書，他是用double link list來實作, 

爲了要自我鍛鍊一下,所以自己寫了一個double link list的實作.

之後，會再補上stack/heap的組合變化. 

# Explain what I DO

## Concept  

___1. DList.h___

我將API 對外的function都放在這裏.

基本上就是產生一個D_Link_List,和Print D_Link_List, 新增刪除Node.

新增和刪除node的方式,就是參考index. index 你可以把當作D_Link_List的Node個數

我現在實作的D_Link_List 最少有一個Head Node, index可以把他當作是0.

```
DList* DList_Create(void);
typedef Ret (*DList_Print)(DList*);
Ret DList_Print_all(DList*, DList_Print);
Ret DList_Insert_Node_ByIndex(DList* HeadNode, DList* InsertNode, int index);
Ret DList_Delete_Node_ByIndex(DList*, int index);
```

在DList 的struct裏面, 我特意將data 存成 pointer to void.
目的其實,就是由上層來決定Double Link List裏面的Node,資料是存什麼.

```
typedef struct _DList
{
  struct _DList* DList_Pre_Ptr;
  struct _DList* DList_Next_Ptr;
  void* data;
  int key;
}DList;
```

___2.DList.c___

大部分實作放在這裏.

___3. main.c___

由於我把data要存什麼type,交給上層來決定,
所以必須要實作存data和display data的function.
可以參考底下這兩個function,所以我要換Node 的data type也會非常方便.
```
Ret MyPrint(DList* MyDList);
DList* Create_Node(int data);
```
我在DList struct 裏面多存了一個key的東西,爲了日後方便,可以search關鍵字.
就可以找到你要的node,但是API還沒寫好.現在只支援Index的方式.XD

在main的後面,有多補一些測試的部分.


## FULL CODE

___1. DList.h___
```

#include "global_enum_macro.h"

typedef struct _DList
{
  struct _DList* DList_Pre_Ptr;
  struct _DList* DList_Next_Ptr;
  void* data;
  int key;
}DList;

DList* DList_Create(void);
typedef Ret (*DList_Print)(DList*);
Ret DList_Print_all(DList*, DList_Print);
Ret DList_Insert_Node_ByIndex(DList* HeadNode, DList* InsertNode, int index);
Ret DList_Delete_Node_ByIndex(DList*, int index);

```

___2. DList.c___

```
#include "DList.h"
#include "stdlib.h"


DList* DList_Create(void)
{
  // Create the Head Node for Double Link List
  DList* HeadNode = (DList*)malloc(sizeof(DList));
  
  check_pointer_return_value(HeadNode,NULL);

  HeadNode->DList_Pre_Ptr = NULL;
  HeadNode->DList_Next_Ptr = NULL;
  
  return HeadNode;

}

Ret DList_Print_all(DList* HeadNode, DList_Print Print)
{
  check_pointer_return_value(HeadNode, Ret_Fail);
  check_pointer_return_value(Print, Ret_Fail);

  // Check the Double Link List is only Head Node ? 
  if (HeadNode->DList_Next_Ptr == NULL && HeadNode->DList_Pre_Ptr == NULL)
  {
    printf("The Double Link List Is EMPTY \n");
    check_pointer_return_value(NULL, Ret_Fail);
  }

  DList* DList_Ptr = HeadNode;

  while (DList_Ptr->DList_Next_Ptr != NULL)
  {
    // if the node is HEAD, we skip that node
    if (DList_Ptr->DList_Pre_Ptr != NULL)
    {
      Print(DList_Ptr);
    }
      DList_Ptr = DList_Ptr->DList_Next_Ptr;
  }

  // Print the Tail Node's data
  Print(DList_Ptr);

}
Ret DList_Insert_Node_ByIndex(DList* HeadNode, DList* InsertNode, int index)
{
  check_pointer_return_value(HeadNode, Ret_Fail);
  check_pointer_return_value(InsertNode, Ret_Fail);

  // check the Double Link List is EMPTY ??
  if (HeadNode->DList_Pre_Ptr == NULL && HeadNode->DList_Next_Ptr == NULL)
  {
    HeadNode->DList_Next_Ptr = InsertNode;
    InsertNode->DList_Next_Ptr = NULL;
    InsertNode->DList_Pre_Ptr = HeadNode;
    printf("Because the Double Link List is EMPTY, so I force the index to be 1 \n");
    return Ret_Success;
  }
  
  int cnt=0;
  DList* Temp_Ptr = HeadNode;

  do
  {
    cnt ++;
    Temp_Ptr = Temp_Ptr->DList_Next_Ptr;
    if (cnt == index)
    {
      InsertNode->DList_Pre_Ptr = Temp_Ptr->DList_Pre_Ptr;
      InsertNode->DList_Pre_Ptr->DList_Next_Ptr = InsertNode;
      Temp_Ptr->DList_Pre_Ptr = InsertNode;
      InsertNode->DList_Next_Ptr = Temp_Ptr;
      return Ret_Success;
    }
    
  } while ( Temp_Ptr->DList_Next_Ptr != NULL);

  // If the index is large than the length of Double link list, just add the NODE into the tail of Double Link List.

  printf("Because the index is large than the length of Double Link List, just insert the NODE into the tail of Double LInke List \n");
  Temp_Ptr->DList_Next_Ptr = InsertNode;
  InsertNode->DList_Next_Ptr = NULL;
  return Ret_Success;
}

Ret DList_Delete_Node_ByIndex(DList* HeadNode, int index)
{
  check_pointer_return_value(HeadNode, Ret_Fail);

  // Check the Double Link List is only Head Node ?
  if (HeadNode->DList_Next_Ptr == NULL && HeadNode->DList_Pre_Ptr == NULL)
  {
    printf("The Double Link List Is EMPTY \n");
    check_pointer_return_value(NULL, Ret_Fail);
  }

  int cnt=0;
  DList* Temp_Ptr = HeadNode;
  while (Temp_Ptr->DList_Next_Ptr != NULL)
  {
    if (cnt == index)
    {
      Temp_Ptr->DList_Pre_Ptr->DList_Next_Ptr = Temp_Ptr->DList_Next_Ptr;
      Temp_Ptr->DList_Next_Ptr->DList_Pre_Ptr = Temp_Ptr->DList_Pre_Ptr;
      printf("Free the Node of Indx[%d] \n", index);
      free(Temp_Ptr->data);
      free(Temp_Ptr);
      return Ret_Success;
    }
    cnt ++;
    Temp_Ptr = Temp_Ptr->DList_Next_Ptr;
  }

  // Because the index is large than the length of Double Link List && the Node which you want to delete is TAIL Node. 

  printf("Free the TAIL NODE \n");
  Temp_Ptr->DList_Pre_Ptr->DList_Next_Ptr = NULL;
  free(Temp_Ptr->data);
  free(Temp_Ptr);
  return Ret_Success;
}

```

___3.main.c___

```
#include <stdlib.h>
#include "DList.h"

Ret MyPrint(DList* MyDList)
{
  check_pointer_return_value(MyDList, Ret_Fail);
  // This place tell how to print the DLinkList's data type
  printf("MyDList->key=%d, *(int*)(MyDList->data)=%d \r\n", MyDList->key, *(int*)(MyDList->data));
}

DList* Create_Node(int data)
{
  DList* myNode = (DList*)malloc(sizeof(DList));
  myNode->data = (int*)malloc(sizeof(int));
  *(int*)(myNode->data) = data; 
  return myNode;
}

void main(void)
{
  DList* HeadNode = DList_Create();

  // Create Node && Insert to the Double Link List
  DList_Insert_Node_ByIndex(HeadNode,Create_Node(10),1);
  DList_Insert_Node_ByIndex(HeadNode,Create_Node(20),1);
  DList_Insert_Node_ByIndex(HeadNode,Create_Node(30),1);
  DList_Insert_Node_ByIndex(HeadNode,Create_Node(40),3);
  DList_Insert_Node_ByIndex(HeadNode,Create_Node(50),1);

  // Delete the Node
  DList_Delete_Node_ByIndex(HeadNode, 2);
  DList_Delete_Node_ByIndex(HeadNode, 5);
  DList_Delete_Node_ByIndex(HeadNode, 1);

  // Print all the Nodes of Double Link List
  DList_Print_all(HeadNode, MyPrint);
}

```

___4.global_enum_macro.h___

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


