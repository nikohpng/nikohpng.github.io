---
title: FreeSWITCH中xml_rpc的使用技巧
date: 2023-03-27 21:12:18
tags: xml_rpc
category: FreeSWTICH
---

本文主要讲解在实际开发中，可能遇到的使用 xml_rpc 命令过程中的一些内容

## 查询 api 的命令

查看 xml_rpc 中的命令通过如下方式即可，在浏览器中打开如下链接：`http://127.0.0.1:8080/webapi/help`

## 调试命令

+ 通过 curl 调用 lua 程序，并传递参数，其中空格需要用 %20 替换
    ```shell script
    curl --user yw:yw@123 http://127.0.0.1:7652/webapi/luarun?call_phone.lua%20var
    ```
+ 
