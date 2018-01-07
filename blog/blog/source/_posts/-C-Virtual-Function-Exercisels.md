title: "[C++][Virtual Function] Exercises"
date: 2015-03-10 23:07:33
categories: C++
tags:
- C++
- Virtual Function
- Dynamic Binding
toc: true
---

## 目的
最近在練習基本的clss的用法, 突然想到一些問題. 

1. 我們利用關鍵字virtual 來使用extend 的function member 的時候,
   base class 是否需要有相同引數的function member? 
   
   -->  ___答案是要的___

2. extend class 要如何存取 base class 的 member data

   --> ___將base class的member data 從private 改成protected___

3. extend class 在用 virtual 關鍵字的時候, 是否需要在function 宣告的前面加上 virtual

   --> ___不用___

## Exercise

底下是間單的測試code.

```
//***********************************
//**********************************
// Arthur : Sheldon Peng
// Date : 2015.3.10
// Purpose : Try the understand the virtual function in C++
//*********************************
//**********************************

#include <iostream>
using namespace std;

class base_class 
{
  public :
      base_class(void);
      virtual void set_value(int);
      virtual void set_value(int, int);
      void print_value(void);

  protected : 
      int v1; 
      int v2; 
};

base_class :: base_class(void)
{
  v1 = 0;
  v2 = 0;
  cout << "base_class constructor !!" <<endl;
}

void base_class :: set_value(int a)
{
  cout << "base class to set_value1" <<endl;
  v1 += a ; 
}

void base_class :: set_value(int a, int b)
{
  cout << "base class to set_value" <<endl;
  v1 = 0;
  v2 = 0;
} 

void base_class :: print_value(void)
{
  cout << "v1 = " << v1 <<endl;
  cout << "v2 = " << v2 <<endl;
}

class extend_class : public base_class
{
  public :
    extend_class(void);  
    void set_value(int, int);
};

extend_class :: extend_class(void) : base_class()
{
  cout << "extend clss constructor" << endl; 
}

void extend_class::set_value(int a, int b)
{
  cout << "extend class to set_value" <<endl;
  v1 += a;
  v2 += b;
}

int main(void)
{
  base_class BC;
  extend_class EC;

  base_class* base_ptr;

  // base class
  base_ptr = &BC;
  base_ptr->set_value(10);
  base_ptr->print_value();

  // extended class
  base_ptr = &EC;
  base_ptr->set_value(5, 5);
  base_ptr->print_value();

  return 0;
}

```


