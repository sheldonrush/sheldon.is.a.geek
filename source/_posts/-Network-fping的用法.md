title: "[Network] fping的用法"
date: 2015-02-25 18:04:09
categories: 
- Network
tags: 
- network 
- ubuntu
toc: true
---

# 目的

之前遇到了一個問題, 假如我現在在一個封閉的區域網路內, 
我要如何得知在這個網域裏面, 有存在哪些其他電腦. 


# 做法

這時後就可以用ubuntu 裏面的fpring 來解決這個問題. 

___1.先利用ifconfig 來觀察, 你現在所在的網域.___

{% img http://sheldonrush.github.io/sheldon.is.a.geek/imgs/ifconfig.jpeg %}

___2. 再利用fping 來觀察其他電腦IP___

{% img http://sheldonrush.github.io/sheldon.is.a.geek/imgs/fping.jpeg %}

___-g addr/mask___

   Generate a target list from a supplied IP netmask, or a starting and ending IP.  Specify the netmask or start/end in the targets
   portion of the command line. If a network with netmask is given, the network and broadcast addresses will be excluded. ex. To ping the
   network 192.168.1.0/24, the specified command line could look like either:

   fping -g 192.168.1.0/24 or fping -g 192.168.1.1 192.168.1.254

