title: "[Queue] Create the Queue structure by the Double Link List"
date: 2015-05-05 08:21:52
categories:
- queue
tags:
- data structure
- queue 
- double link list
toc: true
---

# Purpose

爲了鍛鍊自己的能力，所以就是想寫點東西，看到別人用雙向鏈接兜出了

一個queue 的結構,自己也想無中生有的產生一個出來.前一篇已經有完成了

雙向鏈結，這篇就有點想是表現組合的威力.基本上無論是queue或是stack

都是雙向鏈結的特例,所以真是基礎打的好，什麼都能迎刃而解.

# Explain the conecpt of the Queue Structure. 

Queue 其實就是先進先出.我們只有多包一層API,強迫caller使用API的時候,

Push都是將雙向鏈結的Node,推在前端,讓後將原本在queue 裏面的往後移.

Pop的話，就單純將queue裏面最後一個雙向鏈結的Node移除. 

槪念相當單純,由於我在雙向鏈結的實作,將Node所存的type,交給caller來決定,

所以在Queue 的 Demo code裏面,也是沿用這樣的概念. 

根據如何操作Queue,所以我開了底下這些API, 

實際如何包就請參考擺在最後面的code.

```
Queue* Queue_Create(void);
Ret Queue_Destroy(Queue*);
Ret Queue_Print_all(Queue*, DList_Print);
Ret Queue_Push(Queue*, DList* InsertNode);
Ret Queue_Pop(Queue*);
Ret Queue_Get_Length(Queue*, int* length);
```

由於Queue其實就是雙向鏈接的特例，所以結構上就是跟雙向鏈結一樣

只是會限制他的操作，所以Queue的結構就定義成底下

```
typedef struct _Queue
{
  DList* HeadNode;
}Queue;

```

# FULL CODE 

___1. queue.h___

```
#include "DList.h"

typedef struct _Queue
{
  DList* HeadNode;
}Queue;

Queue* Queue_Create(void);
Ret Queue_Destroy(Queue*);
Ret Queue_Print_all(Queue*, DList_Print);
Ret Queue_Push(Queue*, DList* InsertNode);
Ret Queue_Pop(Queue*);
Ret Queue_Get_Length(Queue*, int* length);
```

___2.queue.c___

```
#include <stdlib.h>
#include "queue.h"

Queue* Queue_Create(void)
{
  Queue* myQueue = (Queue*)malloc(sizeof(Queue));
  check_pointer_return_value(myQueue, NULL);

  myQueue->HeadNode = DList_Create();
  return myQueue;
}

Ret Queue_Destroy(Queue* myQueue)
{
  check_pointer_return_value(myQueue, Ret_Fail);
  if ( Ret_Fail == DList_Destroy( myQueue->HeadNode) )
  {
    SR_ERR("DList_Destroy FAILLL\n");
    return Ret_Fail;
  }
  SR_DBG("Free the QUEUE\n");
  free(myQueue);
  return Ret_Success;
}

Ret Queue_Print_all(Queue* myQueue, DList_Print Print)
{
  check_pointer_return_value(myQueue, Ret_Fail);
  return DList_Print_all(myQueue->HeadNode, Print);
}

Ret Queue_Push(Queue* myQueue, DList* InsertNode)
{
  // I think the Head Node doesn't belong to be the Data Node.
  check_pointer_return_value(myQueue, Ret_Fail);
  return DList_Insert_Node_ByIndex(myQueue->HeadNode, InsertNode, 1);
}

Ret Queue_Pop(Queue* myQueue)
{
  check_pointer_return_value(myQueue, Ret_Fail);
  int length =0;
  if (Ret_Fail == DList_Get_Length(myQueue->HeadNode, &length))
  {
    SR_ERR("Get the Length of DList \n");
    return Ret_Fail;
  }

  return (DList_Delete_Node_ByIndex(myQueue->HeadNode, length));

}

Ret Queue_Get_Length(Queue* myQueue, int* length)
{
  check_pointer_return_value(myQueue, Ret_Fail);
  return DList_Get_Length(myQueue->HeadNode, length);
}

```

___3.DList.h___

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
Ret DList_Destroy(DList*);
typedef Ret (*DList_Print)(DList*);
Ret DList_Print_all(DList*, DList_Print);
Ret DList_Insert_Node_ByIndex(DList* HeadNode, DList* InsertNode, int index);
Ret DList_Delete_Node_ByIndex(DList*, int index);
Ret DList_Get_NodeData_ByIndex(DList*, int index, void** data);
Ret DList_Get_Length(DList*, int* length);
```

___4.DList.c___

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

Ret DList_Destroy(DList* HeadNode)
{

  check_pointer_return_value(HeadNode, Ret_Fail);

  // Check the Double Link List is only Head Node ?
  if (HeadNode->DList_Next_Ptr == NULL && HeadNode->DList_Pre_Ptr == NULL)
  {
    printf("The Double Link List Is EMPTY \n");
    check_pointer_return_value(NULL, Ret_Fail);
  }

  DList* Temp_Ptr = HeadNode;

  while (1)
  {
      if (HeadNode == NULL)
      {
        printf("Success to Destroy the Double Link List\n");
        return Ret_Success;
      }

      HeadNode = HeadNode->DList_Next_Ptr;
      
      free(Temp_Ptr->data);
      free(Temp_Ptr);

      Temp_Ptr = HeadNode;
  }

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


Ret DList_Get_NodeData_ByIndex(DList* HeadNode, int index, void** data)
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
  do 
  {
    cnt++;
    Temp_Ptr = Temp_Ptr->DList_Next_Ptr; 
    if (cnt == index || Temp_Ptr->DList_Next_Ptr == NULL)
    {
      if (Temp_Ptr->DList_Next_Ptr == NULL)
      {
        // Tail Node
        printf("The index is larger than the length of Double Link List, so just get the data of Tail Node for you \n");
      }
      *data = (void*)(Temp_Ptr->data);
      return Ret_Success;
    }
  } while(1);
}

Ret DList_Get_Length(DList* HeadNode, int* length)
{

  check_pointer_return_value(HeadNode, Ret_Fail);
  
  // Check the Double Link List is only Head Node ?
  if (HeadNode->DList_Next_Ptr == NULL && HeadNode->DList_Pre_Ptr == NULL)
  {
    printf("The Double Link List Is EMPTY \n");
    *length = 0;
    check_pointer_return_value(NULL, Ret_Success);
  }

  *length=0;
  DList* Temp_Ptr = HeadNode;
  do 
  {
    (*length)++;
    Temp_Ptr = Temp_Ptr->DList_Next_Ptr; 
    if (Temp_Ptr->DList_Next_Ptr == NULL)
    {
      return Ret_Success;
    }
  } while(1);


}

```

___5.global_enum_macro.h___

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
                                                  { printf("[%s][%d] ERROR!!! #ptr# is NULL, return #value# \r\n",__FUNCTION__,__LINE__); \
                                                  return value;} \

```

___6.main.c___

```
#include <stdlib.h>
#include "queue.h"

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
  Queue* myQueue = Queue_Create() ; 

  // Push the Node to Queue
  Queue_Push(myQueue, Create_Node(10));
  Queue_Push(myQueue, Create_Node(20));
  Queue_Push(myQueue, Create_Node(30));
  Queue_Push(myQueue, Create_Node(40));
  Queue_Push(myQueue, Create_Node(50));
  
  // Print all the Nodes of Queue
  Queue_Print_all(myQueue, MyPrint);

  // Pop out  the Node from Queue
  Queue_Pop(myQueue);
  Queue_Pop(myQueue);

  // Print all the Nodes of Queue
  Queue_Print_all(myQueue, MyPrint);

  // Destroy the Queue
  if (Ret_Success == Queue_Destroy(myQueue))
  {
    SR_DBG("Destroy Queue -- Success !!!\n");
    myQueue = NULL;
  }

  Queue_Print_all(myQueue, MyPrint);

}
```
