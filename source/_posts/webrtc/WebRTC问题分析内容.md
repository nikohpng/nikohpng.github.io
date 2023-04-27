---
title: WebRTC问题分析内容
date: 2023-04-25 20:41:53
tags: Question
category: WebRTC
---

WebRTC 的问题分析分析主要记录一些使用 WebRTC 过程中出现的问题，并且当前没有成体系的内容用于总结，简单的零散分析

## WebRTC 的 oncandidate 时候失败，导致 setLocalDescription 耗时增加

+ 问题：如标题所诉，由于在收集本地网络环境的，一些网卡（虚拟网卡等）的 ip，不具有连接网络的能力，导致在收集 IP 的时候会出现失败。
+ 解决方案：监听来自 oncandidate 的事件，检测到有 relay 或 srly 的就可以直接调用 complete
    ```js
        session.on("icecandidate", function (event) {
            if (event.candidate.type === "srflx" &&
                event.candidate.relatedAddress !== null &&
                event.candidate.relatedPort !== null) {
                event.ready();
            }
        });
    ```
+ 查看错误：通过 `chrome://webrtc-internals` 即可查看 ice 收集失败的时候，对应的 ip 是多少

## 引用
+ [A 40 sec delay of SIP call initiation using JSSIP / WebRTC](https://stackoverflow.com/questions/61253322/a-40-sec-delay-of-sip-call-initiation-using-jssip-webrtc
)
