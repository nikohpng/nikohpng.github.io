---
title: FreeSWITCH中mod的状态变化
date: 2023-03-23 21:05:32
tags: channels
category: FreeSWTICH
---

本文主要讲述在 endpoint 的 mod 在启动关闭与呼叫、挂断等其中的状态变化内容

## mod 的启动关闭

此处的状态变化最为简单，只有两个回调函数，分别如下所示：

```c
    SWITCH_MODULE_LOAD_FUNCTION(mod_test_load);
    SWITCH_MODULE_SHUTDOWN_FUNCTION(mod_test_shutdown);
    SWITCH_MODULE_DEFINITION(mod_test, mod_test_load, mod_test_shutdown, NULL); // mod_test_runtime
```

一般 runtime 都没用，暂时不明白作用。有个这个定义后，再实现相应的启动关闭函数，如下所示：

```c
SWITCH_MODULE_LOAD_FUNCTION(mod_test_load) {
    /// 你想要实现内容，宏展开后有如下参数
    /// switch_loadable_module_interface_t **module_interface, switch_memory_pool_t *pool
}
SWITCH_MODULE_SHUTDOWN_FUNCTION(mod_janus_shutdown) {
    /// 你想要实现内容，宏展开后是个 void，没有参数值
}
```

## channel 的状态变化

channel 的状态变化较为复杂，此处先沿着外呼的流程走一遍所有流程，并会标明调用什么函数发生状态变化，以及状态变化涉及的含义。

此处先展示所有涉及的状态以及它们分别所属结构体

```c
switch_state_handler_table_t test_state_handlers = {
	/*.on_init */ channel_on_init,
	/*.on_routing */ channel_on_routing,
	/*.on_execute */ channel_on_execute,
	/*.on_hangup */ channel_on_hangup,
	/*.on_exchange_media */ channel_on_exchange_media,
	/*.on_soft_execute */ channel_on_soft_execute,
	/*.on_consume_media */ NULL,
	/*.on_hibernate */ NULL,
	/*.on_reset */ NULL,
	/*.on_park */ NULL,
	/*.on_reporting */ NULL,
	/*.on_destroy */ channel_on_destroy};

switch_io_routines_t test_io_routines = {
	/*.outgoing_channel */ channel_outgoing_channel,
	/*.read_frame */ channel_read_frame,
	/*.write_frame */ channel_write_frame,
	/*.kill_channel */ channel_kill_channel,
	/*.send_dtmf */ channel_send_dtmf,
	/*.receive_message */ channel_receive_message,
	/*.receive_event */ channel_receive_event};

//...

test_endpoint_interface->io_routines = &test_io_routines;
test_endpoint_interface->state_handler = &test_state_handlers;
```

+ 外呼 --> channel_outgoing_channel:

  + 参数如下所示：

    ```c
    static switch_call_cause_t channel_outgoing_channel(switch_core_session_t *session, switch_event_t *var_event,
    											switch_caller_profile_t *outbound_profile,
    											switch_core_session_t **new_session, switch_memory_pool_t **pool,
    											switch_originate_flag_t flags, switch_call_cause_t *cancel_cause) {
    
                                                }
    ```

  + 作用：创建媒体流、创建channel、初始化参数等

  + 状态变化函数：`switch_channel_set_state(channel, CS_INIT);`

  + 未知作用函数：`switch_set_flag_locked(tech_pvt, TFLAG_OUTBOUND);`

+ channel_outgoing_channel --> channel_on_init:

  + 参数如下所示：

    ```c
        static switch_status_t channel_on_init(switch_core_session_t *session)
    ```

  + 作用：一般用户初始化该通呼叫所需要的一些初始化参数

  + 返回值：状态值为 `SWITCH_STATUS_SUCCESS` 将自动进入下个状态，否则将跳过该此呼叫

  + 状态变化函数：无

  + 未知作用函数：`switch_set_flag_locked(tech_pvt, TFLAG_IO);`

+ channel_on_init --> channel_on_routing:

  + 参数如下所示：

    ```c
        static switch_status_t channel_on_routing(switch_core_session_t *session)
    ```

  + 作用：一般在此处进行路由相关处理，准备编解码器、创建媒体流端口、生成sdp并与被叫方建立连接等工作

  + 返回值：状态值为 `SWITCH_STATUS_SUCCESS` 将自动进入下个状态，否则将跳过该此呼叫

  + 状态变化函数：`switch_channel_pre_answer(channel)` 和 `switch_channel_answer(channel)`

  + 未知作用函数：`switch_channel_set_flag(channel, CF_AUDIO);`

+ channel_on_routing --> channel_on_execute: (可略)

+ channel_on_hangup:

  + 参数如下所示：

    ```c
        static switch_status_t channel_on_hangup(switch_core_session_t *session)
    ```

  + 作用：呼叫中断的时候，此处的函数就会被回调，无论是 a-leg 还是 b-leg, 一般用于销毁信令上的一些内容

  + 返回值：状态值为 `SWITCH_STATUS_SUCCESS` 将自动进入下个状态，否则将跳过该此呼叫

  + 状态变化函数：`switch_channel_hangup(channel, SWITCH_CAUSE_NORMAL_CLEARING);`

+ channel_on_hangup --> channel_on_destroy:

  + 参数如下所示：

    ```c
    static switch_status_t channel_on_destroy(switch_core_session_t *session)
    ```

  + 作用：在呼叫被 hangup 完成后被回调，用于销毁初始化的媒体流、端口等

  + 返回值：状态值为 `SWITCH_STATUS_SUCCESS` 将自动进入下个状态，否则将跳过该此呼叫



