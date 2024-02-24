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

## 修改FS的contact

```
<param name="apply-nat-acl" value="nat.auto"/>
<action application="export" data="sip_contact_user=${sip_from_user}" />
```

## FS 修改由客户端刷新 session
```xml
<param name="enable-timer" value="true"/>
<param name="session-timeout" value="1800"/>
```

## FS 修改语音舒适噪音问题
防止hold后一直播放铃音，导致无法unhold。sdp 中也有 cng 相关配置，具体请看 sdp
```xml
<param name="suppress-cng" value="true"/>
```

## FS 回复指定的SIP错误信息
```lua
local response_code = "500" -- 响应码为500，表示Internal Server Error
local reason_phrase = "Internal Server Error" -- 响应原因短语
session:execute("respond", response_code .. " " .. reason_phrase)
···
