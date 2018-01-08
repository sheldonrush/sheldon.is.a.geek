title: "[Raspberry Pi][Cam Module] 行車記錄器"
date: 2015-03-01 07:19:58
categories: Raspberry Pi 
tags: 
- Cam Module
- Raspberry Pi
toc: true
---

## 目的

自從桃園一場大水以後, 

{% image http://sheldonrush.github.io/sheldon.is.a.geek/imgs/11245.jpg %}

我的小佛浮起來了,行車記錄器也跟着消失了...
最近一直聽到有關行車糾紛,心理怕怕,但是又不想花錢買行車記錄器.
所以想說用免錢的PI camera module 來玩玩.

## 方法

___1.製作免費的case,在[上一篇文章](http://sheldonrush.github.io/sheldon.is.a.geek/2015/02/26/-Raspberry-Pi-%E8%A3%BD%E4%BD%9CPI%E7%9A%84%E8%A1%A3%E6%9C%8D-Case/)中, 可以參考一下. ___


___2. 我主要[參考這篇](http://dreamgreenhouse.com/projects/2013/picar/) ___

這篇功能超級多...但是我現在其實只要錄制的功能, 所以我把大部分的東西都拔掉了.
假如想要玩更高階的行車記錄器,可以仔細研究聯結.

```
#!/bin/sh
# Purpose : Just test for recording video on cam module of raspberry pi
# Author : Sheldon
echo `date +%D` ", Start the test for cam module !!"

cd /home/pi
# set limit of rolling videos to 16GB
limit=4000000

while [ true ]; do 
        # Video disk space used
        used=$(du Video | tail -1 | awk '{print $1}')
        #echo `date +%s` "U Video" $used
        
        # Free up disk space if needed
        while [ $limit -le $used ]
        do
                remove=$(ls -1tr Video | grep .h264 | head -n 1)
                echo `date +%D` "-" $remove
                rm Video/$remove
                # Calculate disk space used
                used=$(du Video | tail -1 | awk '{print $1}')
        done

        # New file name
        current=$(date +"%Y_%m_%d_%H:%M:%S.h264")
        echo `date +%s` "+" $current

        # Capture a 5 minute segment of video
        raspivid -n -b 9000000 -w 1280 -h 720 -o Video/$current -t 300000
done

```


## 成果

Raspberry Pi 放反了,..主要是來測試cam module, 基本功能看起來work...XDD

{% youtube dSsiKKEKirw %}

