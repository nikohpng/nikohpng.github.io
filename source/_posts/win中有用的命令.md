---
title: win中有用的命令
date: 2022-01-01 20:27:09
tags: win
category: 命令行
---

# win 测试内网中所有连接设备
```cmd
for /l %i in (1,1,255) do ping -n 1 -w 60 192.168.1.%i | find "回复" >> pingall.txt
```
