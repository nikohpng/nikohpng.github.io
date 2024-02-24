---
title: Sofia中的一些概念
date: 2023-12-23 21:36:32
tags: Sofia
category: freeswitch
---

本文主要对 Sofia 中的一些概念进行分析与解释，方便进行源码阅读。

## Sofia 概念

### 事件循环Event loop 与根对象 root object
+ NUA以事件反应器模式(也称为分发及通知模式)驱动事件系统(请参考[Using Design Patterns to Develop Reusable Object-oriented Communication Software, D.C. Schmidt, CACM October '95, 38(10): 65-74]一书)。
Sofia以任务作为编程模型的基本执行单元。根据编程模型，程序可以请求事件循环在特定事件触发时调用回调函数。具体事件包括I/O激活，定时器或其它任务传递的异步消息。
+ root 对象是应用软件中描述一个任务的句柄。
    + 透视事件的另一种方式是：root对象描述任务的主事件循环。通过root对象，任务代码可以访问它的上下文信息(magic)和线程同步，比如说等待对象、定时器，消息。
+ 使用NUA服务的应用必须创建一个root对象，并设置处理NUA事件的回调函数。
    + 调用 `su_root_create()`创建root对象，调用`nua_create()`函数注册回调函数。root 对象的数据类型是 su_root_t。

### Magic
+ magic 是一种描述上下文指针的术语，应用程序可以把它绑定到Sofia栈的各种对象上（比如说root对象和操作句柄）。当主事件循环调用注册的回调函数时，上下文指针会回传给应用层代码。
Sofia栈保留回调函数调用之间的上下文信息。应用层可以利用上下文信息存储处理事件所需要的任何信息

### 内存处理
+ 给定的任务需要分配许多小块内存时，使用home-based内存管理机制会很便利。内存通过home对象分配，它维护所有内存块的引用信息。当home对象释放时，所有相关内存一并释放。这个机制简化了应用逻辑，因为应用层代码不需要跟踪内存块的分配记录，也不需要独立释放每一个内存块。
 使用NUA服务的应用程序可以使用SU库提供的内存管理服务，但这不是强制要求的。
 

 

## 引用
+ [Sofia "nua"模块--高层UA库](https://blog.csdn.net/yetyongjin/article/details/105839864)
