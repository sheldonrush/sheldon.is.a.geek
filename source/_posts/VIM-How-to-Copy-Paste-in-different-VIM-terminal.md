title: "[VIM]How to Copy&Paste in different VIM terminal"
date: 2015-06-13 16:43:22
categories:
- VIM 
tags:
- VIM 
toc: true
---

# PURPOSE

Recent I want to copy&past in different VIM terminals, so I spent a few time to figure out how to do that. w

# How TO Do 

There is a simple way to do.

## Check your VIM information first 

1. open the shell terminal 
2. type the following command to check the vim information

```
   vim --version | grep xterm_clipboard
```

If you find the below message, it tells you that the xterm_clo[board of VIM is off

```
   -xterm_clipboard
```

## Install the VIM-Plugin to enable the feature.

```
   sudo apt-get install vim-athena
   sudo apt-get install vim-gnome
   sudo apt-get install vim-gtk
```
