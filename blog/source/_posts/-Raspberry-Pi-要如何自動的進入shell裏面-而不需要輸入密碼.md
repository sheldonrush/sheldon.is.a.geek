title: "[Raspberry Pi] 要如何自動的進入shell裏面, 而不需要輸入密碼"
date: 2015-02-25 23:35:19
categories: Raspberry Pi
tags:
- Raspberry Pi
toc: true
---

## 目的

假如你寫了一程式， 但是你希望他能在Pi上, 從一上電後自己能夠執行. 
要怎麼做呢?

## 方法

我參考了這篇. [auto_login](http://dreamgreenhouse.com/projects/2013/picar/)

___1. 將/etc/inittab 裏面的, 下面這行mark.___

```
Line 54 ==> #1:2345:respawn:/sbin/getty --noclear 38400 tty1
```

___2. 並將底下這行附在剛剛mark的地方___

```
Line 55 ==> 1:2345:respawn:/bin/login -f pi tty1 </dev/tty1 >/dev/tty1 2>&1
```

不知道有沒有人會跟我一樣好奇, 爲什麼上面這行, 沒有帶密碼但是卻能夠, 自動的登入. 
我google了一下, 有[網友說](http://www.raspberrypi.org/forums/viewtopic.php?f=63&t=66251)

> The commands in inittab are run as root and root does not need passwords for many things.

也就是說， 在執行inittab的時候, 程式就是root了, 所以根本不需要密碼. 


___3. 將你要跑的script 放在.bashrc 裏面.___

例如底下

```
./your_script.sh &
```
