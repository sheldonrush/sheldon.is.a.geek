title: "[AtoI/ItoA] Sample Code"
date: 2015-05-28 07:03:06
categories:
- c
tags:
- atoi
- itoa
toc: true
---

# Purpose 

每天寫一點小程式,雖然簡單,但是也讓我心情開心,慢慢的讓程式實作能力,變得更有自信.

昨天寫了基本常用的 atoi和itoa 的基本 c api.也算是面試常見用的考古題.

順便複習一下,也是好事情.

# My ATOi 

```
Ret my_atoi(int* num, const char* char_num) 
{
  // 檢查 poineter 是否爲空的, 是的話, return fail
  check_pointer_return_value(char_num, Ret_Fail);

  // init the total value
  int total = 0; 
  
  // 檢查是否爲負數
  int is_positive = 1;
  if (char_num[0] == '-')
  {
    //若爲負數的話,就記錄下來
    is_positive = -1;
    char_num ++;
  }

  // 將每個字元,減去'0',這樣就可以得到字元的value, 
  // 舉例: '0' = 30, '9' = 39 , 若是我要得到字元9的值, 就用 '9'-'0'=39-30=9 
  while ( *char_num != '\0')
  {
    // 將10進位的數往左推一個
    total *= 10;
    total += *char_num - '0';
    char_num++;
  }

  *num = total*is_positive;

  return Ret_Success;
}
```

# My IToA

```
Ret my_itoa(int num, char* char_num)
{
  check_pointer_return_value(char_num, Ret_Fail);

  int is_negative = 0;
  char* return_char = char_num;

  // 處理負數
  if (num < 0)
  {
    is_negative = 1;
    num *= -1;
    
  }

  // 處理0
  if (num == 0)
  {
    *char_num = '0';
    return Ret_Success;
  }

  while ( num != 0)
  {
    // 將0~9的值轉成字元,其實很間單,就是加上'0' 或是 30
    *char_num = '0' + (num % 10);
    num /= 10;
    char_num++;
  }
  
  if (is_negative)
  {
    // 若爲負數的話,將'-'加入字串中
    *char_num = '-';
    char_num++;
  }
  
  // Revert string 
  Revert(return_char);

  return Ret_Success;

}

Ret Revert(char* num_char)
{
  check_pointer_return_value(num_char, Ret_Fail);

  int length = strlen(num_char);
  
  int i;
  // 將字串中的字元整份顛倒
  for (i=0; i<length/2; i++)
  {
    char temp_char ;
    temp_char = num_char[i] ;
    // swap 
    num_char[i] = num_char[length-1-i];
    num_char[length-1-i] = temp_char;
  }

  return Ret_Success;
}

```

# Testing 

```
#include <stdio.h>
#include <string.h>

typedef enum _Ret
{
  Ret_Success,
  Ret_Fail,
  Ret_None
}Ret;

Ret my_atoi(int* num, const char*);
Ret my_itoa(int num, char* char_num);

#define check_pointer_return_value(ptr,value) if(ptr==NULL) \
                                              { printf(" pointer is NULL !!!"); \
                                                return value; } \

void main(void)
{
  char test_char[] = "-159";
  char test_char2[] = "269";
  int temp_num ;
  my_atoi(&temp_num,test_char);
  printf("my_atoi : strlen(test_char) = %zu, test = %d \r\n", strlen(test_char),temp_num );
  my_atoi(&temp_num,test_char2);
  printf("my_atoi : strlen(test_char2) = %zu, test2 = %d \r\n", strlen(test_char2), temp_num);

  char test_char3[255] = "";
  my_itoa(128, test_char3);
  printf("my_itoa : %s \r\n", test_char3);
  memset(test_char3,'\0',strlen(test_char3));
  my_itoa(-259, test_char3);
  printf("my_itoa : %s \r\n", test_char3);

}

```
