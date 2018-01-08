title: "[C++][Virtual Function/Inherit] My Animal World"
date: 2015-03-10 01:22:34
categories: C++
tags:
- Virtual Function
- C++
- Inherit
- Dynamical Binding
toc: true
---

## 目的
最近爲了學習QT, 所以重新練習C++的程式, C++的精華就在於封裝、繼承、多型. 
我寫了一個 My Animal World 來練習這些觀念. 這支程式也非常單純, 
可以來記錄你養寵物的體重. 以及每天都喂他飼料的話, 
他們的體重會變成幾公斤. 

## 練習

___1. main.h___

我在main.h 裏面放了base class 以及 extended class. 

分別帶表了動物和貓以及鼠鼠的屬性.

要注意底下的一些重點. 

1. overload 結構子
2. virutal function的用法. 
3. 我把實作都放到.c 黨裏面了

```
//***********************************
//**********************************
// Arthur : Sheldon Peng
// Date : 2015.3.9
// Purpose : Try the understand the virtual function in C++
//*********************************
//**********************************

#define MAX_ANIMAL_NUMBER 2
enum {
  BIG_CAT,
  SMALL_NAUGHTY,
  MAX_ANIMAL = MAX_ANIMAL_NUMBER,
};

class Animal
{
  public :
    Animal();
    Animal(double,double);
    void feed(int);
    double get_weight(void);
    double get_increase_per_day(void);
    virtual void show_name(void);

  private :
    double weight;
    double increase_per_day;
};

class Cat: public Animal
{
  public :
    Cat(double,double);
    virtual void show_name(void);
};

class Mouse: public Animal
{
  public :
    Mouse(double,double);
    virtual void show_name(void);
};


```

___2.1 main.c___

2.1 的部分, 我放了class 成員的實作, 很間單的體重以及增重的vlaue.

這裡很妙的是假如你 base和extended 有相同的member function (show_name)的話, 

你又想使用extended class 的member function (show_name), 你可以在member function 加上virtual

假如你只想用base class的member function, 就把virtual 去除掉吧.

底下這個範例是留下virtual, 畢竟我想要每個動物東有自己的名字XDDD

我這邊還多定義了一個 pointer array 是放所有的動物的pointer. 

這樣比較好操作. 要注意, 宣告array的member , 我是用 base class的poiner (重要)

這樣就非常方便了. 我就可以利用base class pointer 來操作所有extend class. 讃拉！！


```
//***********************************
//**********************************
// Arthur : Sheldon Peng
// Date : 2015.3.9
// Purpose : Try the understand the virtual function in C++
//*********************************
//**********************************

#include <iostream>
#include "main.h"

using namespace std;

Animal* animal_list[MAX_ANIMAL_NUMBER];

Animal::Animal()
{
  cout << "This is First Animal constructor" << endl; 
}

Animal::Animal(double weight_value, double increase_per_day_value)
{
  weight = weight_value;
  increase_per_day = increase_per_day_value ; 
  cout << "This is Second Animal constructor" << endl; 
}

void Animal::feed(int food)
{
  for (int i=0;i<food;i++)
  {
    weight += increase_per_day;
  }
}

double Animal::get_weight(void)
{
  return weight;
}

double Animal::get_increase_per_day(void)
{
  return increase_per_day;
}

void Animal::show_name(void)
{
  cout << "Hello, I don't have name, please give me a name ." << endl; 
}

Cat::Cat(double a,double b):Animal(a, b) 
{
  cout << "This is Cat's constructor" << endl;
}

void Cat::show_name(void)
{
  cout << "Hello, My name is Lazy Cat ." << endl;
}

Mouse::Mouse(double a,double b):Animal(a, b) 
{
  cout << "This is Mouse's constructor ." <<endl;
}

void Mouse::show_name(void)
{
  cout << "Hello, My name is Sweet mouse ." <<endl;
}

```

___2.2 main.c (main function)___

這是程式主題. 一開始會先init cat/mouse, 然後就進去while loop

等待主人詢問是否要餵食你的可愛動物 XDDD . 


```
int main(void)
{
  int type;
  int food_value;

  Animal* animal_ptr;
  Cat cat(7,0.05);
  Mouse mouse(0.05,0.001);
  animal_list[BIG_CAT] = &cat; 
  animal_list[SMALL_NAUGHTY] = &mouse;

  cout << " Welcom to my animal house " << endl;

  do  
  {
    cout <<" please chose the animal you want to feed " << endl;
    cout <<" [0] for cat, [1] for mouse, [2] check the weight of all cute animals !!" << endl;
    cin >> type ;

    if (type == 0)
    {
       animal_ptr = &cat;
       cout << " 喵喵 !!!" << endl;
       animal_ptr->show_name();
    }
    else if (type == 1)
    {
       animal_ptr = &mouse;
       cout << "唧唧 !!!" << endl;
       animal_ptr->show_name();
    }
    else if (type == 2)
    {
      cout << "======體重表======="<<endl;
      
      for (int i=0; i< MAX_ANIMAL_NUMBER; i++)
      {
        animal_ptr = animal_list[i];
        animal_ptr->show_name();
        cout << "我的重量是 " << animal_ptr->get_weight()<<"(kg)" <<endl;
      }
      cout << "==================="<<endl;
      continue;
    }
    else 
    {
       cout << " wow, qq, We don't have your animal !!" <<endl;
       continue; 
    }

    cout << " How many days you want to feed ?? " << endl;
    cin >> food_value; 
    animal_ptr->feed(food_value);

  }while(1);

  return 0;
}

```


