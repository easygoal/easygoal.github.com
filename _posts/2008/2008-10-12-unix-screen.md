---
layout: post
title: Unix 实用小工具——Screen
category: "spaces.live.com"
tags: ["tutorial"]
keywords: unix
ref: 2008-10-12
lang: zh-cn
---

2008-10-12 17:37:08 CET

Screen 是一个很古老又好用的工具，虽然我并不常用。不过有时在帮助别人做一些事情的同时，我们可以用screen 的分享功能，多人同时使用一个terminal。在demo，教学上实在是很方便。

```bash
sudo chmod u+s /usr/bin/screen

sudo chmod 755 /var/run/screen

screen

在另一个terminal 上screen -x
```

如要多人share
1. ^a + :multiuser on
2. ^a + :acladd 
3. 另一个人就可以 screen -x /[pid] 连上去

