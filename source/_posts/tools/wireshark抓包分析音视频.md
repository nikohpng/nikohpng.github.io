---
title: wireshark抓包分析音视频
date: 2023-03-20 19:36:01
tags: opus、h264
category: capture
---

本文主要记录通过 wireshark 抓音视频的分析方法

## win上抓包分析
window上面主要以导入 lua 解析文件为主

+ 在 wireshark 安装目录下面，有个 init.lua， 在其中加入如下所示内容：
    ```lua
     dofile(USER_DIR..filename)
    ```
+ 导入成功后，重启wireshark， 在工具栏中就会出现导入的 lua 文件名字。

+ 随后到 **编辑->首选项->协议** 中根据需要解码的音视频，填写 payload 值

+ 最后，当获取到 *.pcap 文件后，点击 工具->lua的名字 运行即可

## linux上抓包分析opus

此处步骤仅用于在 linux 上，通过命令行的形式分析 opus

+ 获取 python 脚本，[opus_stream_tool](https://github.com/kamanashisroy/opus_stream_tool)
+ tcpdump -i eth0 port 1234 -w in.pcap
+ tshark -x -r in.pcap -Y "udp.srcport == myport" | cut -d " " -f 1-20 > tmp.txt
+ hex_to_opus.py -x tmp.txt --recordfile out.opus --rtpoffset 42
