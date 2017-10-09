---
title: 玩玩VimOS
date: 2016-12-26 21:52:45
tags:
---

最近发现Ubuntu 16.04版本自带xmodmap,自己又搞了一块poker键盘，这键盘又没有方向键。于是乎遍想能不能通过xmodmap把方向键映射成vim的`hjkl`。尝试了一阵之后，我将caps这种无用的键去了，之后进行上下左右操作只要`caps` + `hjkl`。先创建文件`~/.xmodmap.conf`,然后添加下面的配置：

```conf
clear Lock
keycode 66 = Mode_switch

! navigation shortcuts
keycode 43 = h H Left
keycode 44 = j J Down
keycode 45 = k K Up
keycode 46 = l L Right

```

保存后执行,就可以使用`caps`+`hjkl`进行上下左右移动了

```shell
xmodmap .xmodmap
```
