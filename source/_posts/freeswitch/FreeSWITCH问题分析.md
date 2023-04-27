---
title: FreeSWITCH问题分析
date: 2023-04-27 20:47:49
tags: Question
category: FreeSWTICH
---

本文主要讲解在实际运维过程中容易出现的问题现象以及导致其出现问题的根本原因，可能附带有解决方法

## 30s挂断

呼叫通没问题，但是会出现 30s 挂断。一般原因有以下几个方面：
+ 外网端口与内网端口没有一对一映射：端口错误，导致客户端与 FS 的沟通出现网络无法连接问题，可能是 ACK 问题。
	+ 解决方案：将接口一对一映射
	
## FS与opensip使用tls对接时，2分钟 tcp 断连

opensips 本身有个参数 `tcp_connection_lifetime` 用于控制 tcp 的存活时长，此存活时长只能通过与服务端发送数据来延长时间，无法通过 tcp 本身的 `keepalive` 来进行保活。

最后，必须通过 fs 端发送 option 来延长 tcp 的存活时间。
想要实现在通话中发送 option，那么必须使用 gateway ，并在 gateway 中配置 `ping` 来发送 option。

其中，option 存在问题还没开始通话就有 option 进行发送，其中的端口与通话的端口有什么关系呢？根据抓包发现：当前没通话，有新的呼叫创建情况下，有option，那就直接用option的端口。因而网关的option没通话时的option与新来通话后的option是同一个端口。

> 注意：目前仅仅测试通过了 fs 作为 fresher 进行刷新保活，服务器作为 refresher 时，其 update 无法发送到 fs
