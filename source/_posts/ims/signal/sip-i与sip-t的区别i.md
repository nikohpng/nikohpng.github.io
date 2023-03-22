---
title: sip-i与sip-t的区别i
date: 2023-01-14 10:41:12
tags: signal
category: ims
---

全球语音网络正在向基于IP的通信系统迁移。然而，现有的PSTN网络将继续存在一段时间。因此，VOIP和PSTN之间的互连能力在如今的语音市场上扮演重要的角色。

就像两种语言之间的翻译（英语<->荷兰语），两个系统的映射作为基本的互连功能出现。

SIP-I和SIP-T是两种类似的方法，用于ISUP网络和SIP网络之间的互通，换句话说，就是PSTN和VoIP网络。具体来说，它们有助于通过SIP网络传送ISUP参数，这样在ISUP网络上发起和终止的呼叫就可以通过SIP网络转接，而不会丢失信息。

SIP-I和SIP-T都定义了SIP和ISUP网络之间的消息、参数和错误代码的映射。它们都可以与SIP网络上符合要求的SIP网络组件完全互操作。

SIP-I和SIP-T允许ISUP参数在SIP网络中透明传输的方式是，在入口处的PSTN网关将原始ISUP消息的字面副本附加到SIP消息中；这个ISUP消息作为另一个主体出现在SIP消息中。

## SIP-I 与 SIP-T 的不同点

+ SIP-I是由ITU在2004年开发的（定义在ITU-T Q.1912.5），而SIP-T是由开发SIP的IETF（互联网工程任务组）开发的。

+ SIP-I定义了从SIP到BICC以及ISUP的映射，而SIP-T只解决ISUP的问题。

+ SIP-T是为与本地SIP终端的互操作而设计的，而SIP-I只限于在PSTN网关之间使用。

+ SIP-I更准确，明确地定义了ISUP和SIP之间的参数，而且它详细地定义了电信互连的补充服务，这是SIP-T所不支持的。

+ SIP-I被制造商和运营商广泛接受，特别是软交换机和会话边界控制器（SBC）供应商。

## 引用
[RFC3261 (Session Initiation Protocol), ITU-T  Q.1912.5(Definition of SIP-I) and RFC3372 (SIP-T)](https://www.rfc-editor.org/rfc/rfc3261)
[Difference Between SIP-I and SIP-T](https://www.differencebetween.com/difference-between-sip-i-and-sip-t/)
