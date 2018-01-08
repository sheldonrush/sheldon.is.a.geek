title: "安裝R語言"
date: 2015-06-21 18:02:40
categories:
- R
tags:
- Stock
- R
- Ubuntu
toc: true
---

# Purpose

最近股市賠錢,覺得應該是要改變自己對投資的態度,要更積極一點

而且需要明白如何透過一些分析的軟體來作投資策略分析.

在網路上尋找到一個數據分析的軟體, 稱作"R語言"

{% blockquote Wikipedia https://zh.wikipedia.org/wiki/R%E8%AF%AD%E8%A8%80 %}
R語言，一種自由軟體程式語言與操作環境，主要用於統計分析、繪圖、資料探勘。R本來是由來自紐西蘭奧克蘭大學的Ross Ihaka和Robert Gentleman開發（也因此稱為R），現在由「R開發核心團隊」負責開發。R是基於S語言的一個GNU計劃專案，所以也可以當作S語言的一種實作，通常用S語言編寫的代碼都可以不作修改的在R環境下執行。R的語法是來自Scheme。
{% endblockquote%}

# 安裝

## 添加新的套件source 到sources.list

```
sudo vim /etc/apt/sources.list
deb http://<my.favorite.cran.mirror>/bin/linux/ubuntu trusty/
```
<my.favorite.cran.mirror> 改成適當的mirror {% link 聯結 http://cran.r-project.org/mirrors.html %}

假如在臺灣的話,參考底下

___Taiwan___

http://ftp.yzu.edu.tw/CRAN/ Department of Computer Science and Engineering, Yuan Ze University
http://cran.csie.ntu.edu.tw/  National Taiwan University, Taipei

## 更新AP
輸入底下command
```
sudo apt-get update 
```

假如你遇到底下問題,無法正常update的話
```
W: GPG 錯誤: http://cran.csie.ntu.edu.tw trusty/ Release: 由於無法取得它們的公鑰，以下簽章無法進行驗證： NO_PUBKEY 51716619E084DAB9
```

將51716619E084DAB9改成你在terminal看到的key, 並輸入底下cmd.
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FAF69C646FF368B7
sudo apt-get update
```

## 安裝 R 

```
sudo apt-get install r-base
```

