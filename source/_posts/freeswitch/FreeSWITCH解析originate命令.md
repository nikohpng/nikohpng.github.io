---
title: FreeSWITCH解析originate命令
date: 2022-01-20 20:20:44
tags: FreeSWITCH源码
category: FreeSWTICH
---

我们在实现任何外部呼叫能力的时候无论如何都绕不开 originate 命令或者说 bridge。

当然为了简单起见，我这里先跟着 originate 进行源代码分析。 

## 环境

| 环境名 | 版本     |
|----   |----     |
| WIN   | WIN10   |
| FS    | 1.10.0  |
| VS    | vs2017  |

## `originate_function`

mod_commands 这个模块，集成 FreeSWITCH 大部分的命令，而 originate 命令就是其中的一个命令。位于 mod_commands.c 中，

其中代码如下 ：

```c
    #define ORIGINATE_SYNTAX                                                                                               \
        "<call url> <exten>|&<application_name>(<app_args>) [<dialplan>] [<context>] [<cid_name>] [<cid_num>] "            \
        "[<timeout_sec>]"
    SWITCH_STANDARD_API(originate_function)
    {
        switch_channel_t *caller_channel;
        switch_core_session_t *caller_session = NULL;
        // 定义解析命令参数的内容
        char *mycmd = NULL, *argv[10] = {0};
        int i = 0, x, argc = 0;
        char *aleg, *exten, *dp, *context, *cid_name, *cid_num;
        uint32_t timeout = 60;
        switch_call_cause_t cause = SWITCH_CAUSE_NORMAL_CLEARING;
        switch_status_t status = SWITCH_STATUS_SUCCESS;
    
        if (zstr(cmd)) {
            stream->write_function(stream, "-USAGE: %s\n", ORIGINATE_SYNTAX);
            return SWITCH_STATUS_SUCCESS;
        }
    
        // 判断是否曾经已经建立过连接（此处 session 是通过 SWITCH_STANDARD_API 扩展函数名得到）
        /* log warning if part of ongoing session, as we'll block the session */
        if (session) {
            switch_log_printf(SWITCH_CHANNEL_SESSION_LOG(session), SWITCH_LOG_NOTICE,
                              "Originate can take 60 seconds to complete, and blocks the existing session. Do not confuse "
                              "with a lockup.\n");
        }
    
        // 拷贝命令行参数并根据空格进行字符串分割
        mycmd = strdup(cmd);
        switch_assert(mycmd);
        argc = switch_separate_string(mycmd, ' ', argv, (sizeof(argv) / sizeof(argv[0])));
    
        if (argc < 2 || argc > 7) {
            stream->write_function(stream, "-USAGE: %s\n", ORIGINATE_SYNTAX);
            goto done;
        }
    
        for (x = 0; x < argc && argv[x]; x++) {
            if (!strcasecmp(argv[x], "undef")) { argv[x] = NULL; }
        }
        
        // 参数赋值到变量中
        aleg = argv[i++];
        exten = argv[i++];
        dp = argv[i++];
        context = argv[i++];
        cid_name = argv[i++];
        cid_num = argv[i++];
    
        switch_assert(exten);
    
        if (!dp) { dp = "XML"; }
    
        if (!context) { context = "default"; }
    
        if (argv[6]) { timeout = atoi(argv[6]); }
    
        // 发起呼叫
        if (switch_ivr_originate(NULL, &caller_session, &cause, aleg, timeout, NULL, cid_name, cid_num, NULL, NULL,
                                 SOF_NONE, NULL, NULL) != SWITCH_STATUS_SUCCESS ||
            !caller_session) {
            stream->write_function(stream, "-ERR %s\n", switch_channel_cause2str(cause));
            goto done;
        }
    
        caller_channel = switch_core_session_get_channel(caller_session);
    
        // 如果参数中包含 & ，表明是命令形式，那么将命令的解析。否则将进行 session 转移
        if (*exten == '&' && *(exten + 1)) {
            switch_caller_extension_t *extension = NULL;
            char *app_name = switch_core_session_strdup(caller_session, (exten + 1));
            char *arg = NULL, *e;
    
            if ((e = strchr(app_name, ')'))) { *e = '\0'; }
    
            if ((arg = strchr(app_name, '('))) { *arg++ = '\0'; }
    
            if ((extension = switch_caller_extension_new(caller_session, app_name, arg)) == 0) {
                switch_log_printf(SWITCH_CHANNEL_SESSION_LOG(session), SWITCH_LOG_CRIT, "Memory Error!\n");
                abort();
            }
            switch_caller_extension_add_application(caller_session, extension, app_name, arg);
            switch_channel_set_caller_extension(caller_channel, extension);
            switch_channel_set_state(caller_channel, CS_EXECUTE);
        } else {
            switch_ivr_session_transfer(caller_session, exten, dp, context);
        }
    
        stream->write_function(stream, "+OK %s\n", switch_core_session_get_uuid(caller_session));
    
        switch_core_session_rwunlock(caller_session);
    
    done:
        switch_safe_free(mycmd);
        return status;
    }
```

上诉函数主要做了两件事情，其一：解析命令参数，其二：根据命令参数发起呼叫。

+ 解析命令参数：参数的赋值这里不做赘述，这里简单描述下各个参数的含义，帮助初学者理解
    ```c
        aleg = argv[i++];       // aleg 就是呼叫 url。例如：sofia/internal/1001@192.168.1.110
        exten = argv[i++];      // exten 是 session 转移的参数或者其它命令配置。
                                //      其中 session 转移是修改呼叫方的 caller_number
                                //      至于其它命令可以参考 &hold()
        dp = argv[i++];         // 呼叫形式: 对应与 mod_dialplan_xml、mod_dialplan_directory、mod_dialplan_asterisk
        context = argv[i++];    // context 一般对应 config/dialplan 中的 default 或 public
        cid_name = argv[i++];   // 呼叫方显示到被叫方的名字
        cid_num = argv[i++];    // 呼叫方显示到被叫方的号码
    ```
+ 根据命令参数发发起呼叫：这里的主要执行是 `switch_ivr_originate()` 函数，通过此函数我们获取到一个通话的 session。
下面我将详细讲解此函数中执行内容。

## `switch_ivr_originate`

此函数位于 switch_ivr_originate.c 中，此函数的实现内容为发起呼叫的功能，这其中的代码非常复杂。

这里分成各个阶段分别分析其中的内容。

### 传入参数

以下是此函数的传入参数的解释，主要位于 switch_ivr.h 中：

```c
/*!
  \brief Make an outgoing call
  \param session originating session
  \param bleg B leg session
  \param cause a pointer to hold call cause
  \param bridgeto the desired remote callstring
  \param timelimit_sec timeout in seconds for outgoing call
  \param table optional state handler table to install on the channel
  \param cid_name_override override the caller id name
  \param cid_num_override override the caller id number
  \param caller_profile_override override the entire calling caller profile
  \param ovars variables to be set on the outgoing channel
  \param flags flags to pass
  \return SWITCH_STATUS_SUCCESS if bleg is a running session.
  \note bleg will be read locked which must be unlocked with switch_core_session_rwunlock() before losing scope
*/
/*!
  \brief 往外发起呼叫
  \param 正在发起会话的会话，可以简单理解为 a-leg，如果没有 a-leg 传入 NULL 即可
  \param 传入接收 b-leg 的 session 指针
  \param 传入接收呼叫结果的指针
  \param 想要呼叫的远端设备的链接
  \param 向外呼叫超时时间
  \param table optional state handler table to install on the channel
  \param 覆盖呼叫者的名字
  \param 覆盖呼叫者的号码
  \param 覆盖呼叫者的整个配置内容
  \param 设置在外呼通道上的变量
  \param flags flags to pass
  \retur 如果 b-leg 建立成功会返回 SWITCH_STATUS_SUCCESS
  \note bleg will be read locked which must be unlocked with switch_core_session_rwunlock() before losing scope
*/
```
