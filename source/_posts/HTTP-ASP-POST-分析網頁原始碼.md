title: "[HTTP][ASP][POST][表單] 分析網頁原始碼"
date: 2015-03-28 22:56:02
categories:
- POST
tags:
- POST
- HTTP
- ASP
toc: true
---

# Purpose

有一個好朋友，最近在研究如何自動的把股市資料抓下來, 以利後續的分析處理

他問了我一個問題， 要怎麼了解底下文章, 爲什麼這樣的URL就可以抓取想要個資料

[抓取鉅亨網個股歷史行情](http://ric1565.blogspot.tw/2013/12/web.html)

老實說，我不是做web應用的，XDDD,當下的反應是 "瓦阿哉". 所以爲了要保存一些顏面.

花了時間看了一下. 怎麼分析html....

# Introudction

他的目的是要抓取網頁中表單的資料, 

所以必須要知道HTTP如何處理表單的. 

在開始之前, 你必須要知道如何檢視html的原始碼,

這部份，最容易了.

先來安裝個chrome. 然後找一個有[表單的網頁](http://www.cnyes.com/twstock/ps_historyprice/1101.htm)

(按右鍵) -> (檢視網頁原始碼)

大功告成, Aha....

# 表單

html的標籤有許多種, 這裡的[聯結](http://w3school.com.cn/tags/tag_script.asp)作者很用心誒. 

但是現在我們其實只是要focus 在表單上.

表單標籤的keyword 其實就是 ___form___, 所以你只需要在原始碼中查詢___form___的關鍵字， 你就會發現表單的位置.

例如底下,

```
<form name="aspnetForm" method="post" action="/twstock/ps_historyprice/1101.htm" id="aspnetForm">
<div>
...
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE"
...
</form>  
```

標籤大多都是成對的出現,所以你發現了___form___的開頭,就可以看到一個___/form___的結尾

從上面的資訊可以看出一些東西, 組成這個表單標籤的元素, 其實有許多種, 

底下只是一個例子,

___1. name="aspnetForm"___

這個就是這個表單的名字

___2. method="post"___

當你按完___查詢___按鍵之後, 會利用post的方式將表單以及一些user自定的參數,帶給___action這個程式___, 
一般講到post的時後,都會提到get這個方法, 我大概只有一個概念, 似乎post會比get還要安全,XDD 這種粗淺的認知.
有興趣的可以看一下,底下的差異介紹.

[GET VS POST Method](http://blog.toright.com/posts/1203/%E6%B7%BA%E8%AB%87-http-method%EF%BC%9A%E8%A1%A8%E5%96%AE%E4%B8%AD%E7%9A%84-get-%E8%88%87-post-%E6%9C%89%E4%BB%80%E9%BA%BC%E5%B7%AE%E5%88%A5%EF%BC%9F.html)

___3. action="/twstock/ps_historyprice/1101.htm"___

如同上面所說,其實1101.htm應該就是web server上某一個程式, 當我們按完___查詢___以後, 會將一些資料（例如, 開始日期與結束日期)
送到___action___這裏. 去做相對應的處理. [例子](http://ric1565.blogspot.tw/2013/12/web.html), 其實就是將這其間內的資料從data base裏面撈出來
在顯示給user看.

# 如何用改變URL的方式, 來獲取各個表單的資訊.

在開始之前,應該有人,會有一些疑問,爲什麼網址不變,但是卻可以撈到不同的股市資料, 其實就是剛剛那個post的方式, 做到的. 

有人會問，要怎麼知道,新的表單需要在URL上多加什麼爾外的資訊. 才能獲得呢.

其實很直觀,你觀察一下網頁上的表單,會需要什麼資訊，這個例子是用, (開始日期與結束日期),你就直接在原始碼中找尋(開始日期與結束日期)

多們直觀對吧 kerker

```
<form name="aspnetForm" method="post" action="/twstock/ps_historyprice/1101.htm" id="aspnetForm">
...
<span class="srchyear2" >
開始日期<input name="ctl00$ContentPlaceHolder1$startText" type="text" value="2015/02/28" maxlength="10" id="ctl00_ContentPlaceHolder1_startText" style="width:72px;" />
結束日期<input name="ctl00$ContentPlaceHolder1$endText" type="text" value="2015/03/28" maxlength="10" id="ctl00_ContentPlaceHolder1_endText" style="width:72px;" />
<input type="submit" name="ctl00$ContentPlaceHolder1$submitBut" value="查詢" id="ctl00_ContentPlaceHolder1_submitBut" class="butn btnga" />
<span style="float:right">
</span>
</span>
...
</form>  

```

這樣就可以發現,在表單裏面，其實有一段標籤是描述（開始日期與結束日期),

跟剛剛一樣，假如要知道input這個標籤屬性在幹嗎，查閱一下 [HELP-HTML-標籤](http://w3school.com.cn/tags/tag_script.asp)

這邊稍爲帶一下，

1. name : 代表input 的名字
2. type : input 的屬性，text很明顯就是只文字.
3. value : 這個input 是什麼值. 看起來就像是日期.

有了以上的簡單的認知,我們就可以透過key URL 的方式，來改變我們要搜尋的股市歷史資料

首先， 由於我們這個是要股市歷史資料,你觀察一下,他的歷史資料的首頁,

```
http://www.cnyes.com/twstock/ps_historyprice.aspx
```

聯結進去預設的各股，是臺G電2330, 假如是要查聯發科2454的話.

你只要將code=2454 帶入URL中即可.像是底下.

要注意一個地方，假如我需要帶參數在URL中時，你就必須多加一個"？"在網址的最後面. 然後緊接着參數code

```
http://www.cnyes.com/twstock/ps_historyprice.aspx?code=2454
```

看起來真的很容易對吧...

再來就是，假如我需要知道某個區間內的歷史資料要怎麼做呢.

其實很間單,就是在多加時間的參數下去,對吧. 時間參數所要用的name 其實就是網頁原始碼中的name(可以參考上一段所說)

1. Name of 開始時間 : ctl00$ContentPlaceHolder1$startText
2. Name of 結束時間 : ctl00$ContentPlaceHolder1$endText

要注意兩個地方

1. 等於（=）後面,就是塞值進去，這個例子就是時間.
2. 帶入的每個參數之間都要有"&"將每個參數做區隔. 底下例子其實，就是三個參數.

```
http://www.cnyes.com/twstock/ps_historyprice.aspx?code=2454&ctl00$ContentPlaceHolder1$startText=2014/01/01&ctl00$ContentPlaceHolder1$endText=2014/01/28
```

