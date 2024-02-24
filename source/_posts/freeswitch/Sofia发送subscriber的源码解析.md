---
title: Sofia发送subscriber的源码解析
date: 2023-12-23 20:48:25
tags: Sofia
category: freeswitch
---

本文将对 sofia 发送 subscriber 这个信令进行源码分析其中会对一些涉及到的其它源码进行解析，用以后续学习

## Sofia 的 handle

这里对 handle 进行一定的解释。这个 handle 用于管理向谁发送什么样的数据，这个谁一般指定 uac 。 一个 handle 会生成一个新的 call-id，产生一个新的 session。
+ `nua_handle_destroy()`: 用于销毁这个 handle。
    + 小知识：FreeSWITCH 一般在创建后 `nua_handle_bind(fnh, &mod_sofia_globals.destroy_private)`，后续不用后会自动销毁, 这个销毁主要依赖于 our_sofia_event_callback 
    中的以下代码：
    ```c
        if ((sofia_private && sofia_private == &mod_sofia_globals.destroy_private)) {
        		nua_handle_bind(nh, NULL);
        		nua_handle_destroy(nh);
        		nh = NULL;
        }
    ```
+ `nua_handle_bind`: 绑定给回调函数一个上下文。在发生 sip 的状态机变化的时候，nua_create 注册的回调函数会回调，其中的会带有这个绑定的值。解绑传值为 NULL
+ `nua_handle_ref`: 增加了 handle 的 sub_ref, 未知
+ `nua_handle_unref`: 减少了 handle 的 sub_ref，未知
```
nua.c:304 
```
## Sofia 的 subscribe 流程

```
nua.c:701 nua_subscribe NUA_SIGNAL(nh, nua_r_subscribe, tag, value)

```
