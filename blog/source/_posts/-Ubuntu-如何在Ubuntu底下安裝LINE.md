title: "[Ubuntu]如何在Ubuntu底下安裝LINE"
date: 2015-03-01 01:08:03
categories: Ubuntu
tags: 
- Line 
- Ubuntu
toc: true
---

## 目的

我每次要用line的時候,都需要把virtual box的windows打開，這樣才能使用
PC上的line,這樣真的太耗時了,所以今天研究了在ubuntu底下如何使用LINE.


___1. Ubuntu 版本___

```
sheldon$ cat /etc/issue
Ubuntu 14.04.2 LTS \n \l
```

___2. Wine 版本___

```
sheldon$ wine --version
wine-1.6.2
```

## 方法

我參考了這個[聯結](http://askubuntu.com/questions/517932/how-can-i-install-line)

___0. 安裝p7zip以及vcrun2008___

```
sudo apt-get install p7zip  # [p7zip](http://packages.ubuntu.com/zh-tw/lucid/utils/p7zip-full)是具有高壓縮率的套件
wget http://winetricks.org/winetricks # [winetricks](http://www.ubuntu-tw.org/modules/newbb/viewtopic.php?post_id=66666) 是wine自動安裝函式庫的好工具XD
chmod +x winetricks # 賦予執行的權利
./winetricks #執行winetricks

```

___參考底下圖片安裝vcrun2008___
{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/winetricks1.png %}
{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/winetricks2.png %}
{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/winetricks3.png %}


___1. 產生一個暫存的line_tmp, 並cd 進去___

```
mkdir line_tmp
cd line_tmp
```

___2. wget 最新的LINE version, 並且利用7z去extract 資料出來___

```
wget http://dl.desktop.line.naver.jp/naver/LINE/win/LineInst.exe
7z x -y LineInst.exe
# x      eXtract with full paths
# -y     Assume Yes on all queries
```

___3. 由於你7z extract 出來的三個資料夾都是亂碼, 所以用node number去將名字改成LINE/resources,並將resources 放到LINE folder中___

```
inode1=$(ls -ilab | awk 'FNR == 4 {print $1}')
inode2=$(ls -ilab | awk 'FNR == 6 {print $1}')
find . -inum $inode1 -exec mv {} LINE \;       #將inode1的folder更名成LINE
find . -inum $inode2 -exec mv {} resources \;  #將indoe2的folder更名成resources
mv ./resources/res ./LINE
mv ./LINE ../
cd ..
rm -R line_tmp

```

___4. 用wine執行 Line.exe___

```
wine Line.exe
```

