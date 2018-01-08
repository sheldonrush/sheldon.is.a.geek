title: "[Stack] Create the Stack Structure by Double Link List"
date: 2015-05-07 08:34:37
categories:
- stack
tags:
- stack
- double link list
- c
- data structure
toc: true
---

# Purpose

繼前前篇的實作queue結構,最後再補上如何透過組合double link list的方式,

實作stack strcutre的操作API. 實在是容易,其實就是雙向鏈接的特殊形式.

# Concept

Stack 就是Fist In Last Out, 也就是我們熟知的先進後出,或稱作後進先出. 

把握這個概念. 所以在Stack.h 下,我開出了底下對外的API.

```
// Stack.h

#include "DList.h"

typedef struct _Stack
{
  DList* HeadNode;
}Stack;

Stack* Stack_Create(void);
Ret Stack_Destroy(Stack*);
Ret Stack_Print_all(Stack*, DList_Print);
Ret Stack_Push(Stack*, DList* InsertNode);
Ret Stack_Pop(Stack*);
Ret Stack_Get_Length(Stack*, int* length);
Ret Stack_Get_Node(Stack*, DList**);
```

看吧, 自定的stack structure,其實就是指向雙向鏈結頭的指標.

Stack_Pop就是將一個雙向鏈結的Node（index=1） 移除. (我定義的雙向鏈結,Head Node,不能當作是data node).

所以總是清除掉最後進來的Node.

而Stack_Push,也是一樣,插入一個Node,到index=1的地方. 

所以總是將Node 放到最靠近Head Node的地方. 

多了一個 Stack_Get_Node, 這其實是將 index=1的Node 取出來,但是我並沒有清除掉index=1的Node.

所以大致就是這樣囉...

# FULLL CODE 

___1. stack.c___

```
#include <stdlib.h>
#include "stack.h"

Stack* Stack_Create(void)
{
  Stack* myStack = (Stack*)malloc(sizeof(Stack));
  check_pointer_return_value(myStack, NULL);

  myStack->HeadNode = DList_Create();
  return myStack;
}

Ret Stack_Destroy(Stack* myStack)
{
  check_pointer_return_value(myStack, Ret_Fail);
  if ( Ret_Fail == DList_Destroy( myStack->HeadNode) )
  {
    SR_ERR("DList_Destroy FAILLL\n");
    return Ret_Fail;
  }
  SR_DBG("Free the QUEUE\n");
  free(myStack);
  return Ret_Success;
}

Ret Stack_Print_all(Stack* myStack, DList_Print Print)
{
  check_pointer_return_value(myStack, Ret_Fail);
  return DList_Print_all(myStack->HeadNode, Print);
}

Ret Stack_Push(Stack* myStack, DList* InsertNode)
{
  // I think the Head Node doesn't belong to be the Data Node.
  check_pointer_return_value(myStack, Ret_Fail);
  return DList_Insert_Node_ByIndex(myStack->HeadNode, InsertNode, 1);
}

Ret Stack_Pop(Stack* myStack)
{
  check_pointer_return_value(myStack, Ret_Fail);
  // Beacuse nature of Stack, the node need to be popped out from the Noed which nearest Head Node.
  return (DList_Delete_Node_ByIndex(myStack->HeadNode, 1));

}

Ret Queue_Get_Length(Stack* myStack, int* length)
{
  check_pointer_return_value(myStack, Ret_Fail);
  return DList_Get_Length(myStack->HeadNode, length);
}

Ret Stack_Get_Node(Stack* myStack, DList** RetNode)
{
  check_pointer_return_value(myStack, Ret_Fail);

  // Because the Nature of Stack, we alwasy get the first Node.
  if ( Ret_Fail == DList_Get_Node_ByIndex(myStack->HeadNode, 1, RetNode))
  {
    SR_ERR("Fail to get Node from Stack \n");
    return Ret_Fail;
  }

}

```

___2.DList.h___

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
Ret DList_Get_Node_ByIndex(DList*, int index, DList**);
Ret DList_Get_Length(DList*, int* length);
```

___3.DList.c___

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

Ret DList_Get_Node_ByIndex(DList* HeadNode, int index, DList** RetNode)
{

  check_pointer_return_value(HeadNode, Ret_Fail);

  int cnt = 0;
  for (cnt = 0; cnt<=index; cnt++)
  {
    if (HeadNode->DList_Next_Ptr == NULL || cnt == index)
    {
      SR_DBG("This is the TAIL NODE or find the Node, so return NODE \n");
      goto RETURN_DONE;
    }
    HeadNode = HeadNode->DList_Next_Ptr;
  }

  goto RETURN_DONE;

RETURN_DONE :
  *RetNode = HeadNode;
  return Ret_Success;
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
                                                  { printf("[%s][%d] ERROR!!! Pointer is NULL \r\n",__FUNCTION__,__LINE__); \
                                                  return value;} \




```

___5.main.c___

```
#include <stdlib.h>
#include "stack.h"

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
  Stack* myStack = Stack_Create() ; 

  // Push the Node to Queue
  Stack_Push(myStack, Create_Node(10));
  Stack_Push(myStack, Create_Node(20));
  Stack_Push(myStack, Create_Node(30));
  Stack_Push(myStack, Create_Node(40));
  Stack_Push(myStack, Create_Node(50));
  
  // Print all the Nodes of Queue
  Stack_Print_all(myStack, MyPrint);

  // Pop out  the Node from Queue
  // Test  Stack_Get_Node 
  SR_DBG("=================\r\n");
  DList* D_Ptr = NULL;
  Stack_Get_Node(myStack, &D_Ptr);
  SR_DBG("data is %d \r\n",*(int*)(D_Ptr->data));
  SR_DBG(" Pop out Node\r\n");
  Stack_Pop(myStack);
  Stack_Get_Node(myStack, &D_Ptr);
  SR_DBG("data is %d \r\n",*(int*)(D_Ptr->data));
  SR_DBG(" Pop out Node\r\n");
  Stack_Pop(myStack);
  Stack_Get_Node(myStack, &D_Ptr);
  SR_DBG("data is %d \r\n",*(int*)(D_Ptr->data));

  // Print all the Nodes of Queue
  Stack_Print_all(myStack, MyPrint);

  // Destroy the Queue
  if (Ret_Success == Stack_Destroy(myStack))
  {
    SR_DBG("Destroy Stack -- Success !!!\n");
    myStack = NULL;
  }

  Stack_Print_all(myStack, MyPrint);

}
```

