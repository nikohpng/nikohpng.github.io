---
title: janus的admin/api详解
date: 2022-05-15 10:29:34
tags: janus-admin
category: janus
---

此处讲解的 API 是用于检索 janus 的服务器信息和进行一部分的操作能力。如果你对插件内部的异步通知感兴趣，请通过 janus_eventhandler 机理

你可以通过配置 janus.transport.http.jcfg 中关于 admin API 部分，配置开启与关闭。其中你可以采用密码的形式或其它形式对访问 admin API 进行限制

当你使用 websockets 的时候，必须使用 Janus -admin-protocol 作为子协议。（可以参考janus-protocol）



## Admin API Requests

以下是不同的请求方式

### 普通请求

其中除了 info 都采用 https://192.168.1.51:7889/admin post的形式。

+ info: 用于获取服务器的一些信息，不需要秘钥即可请求

  + request（get）

      ```json
      // 后面的数字为时间戳 
      https://192.168.1.51:7889/admin/info?_=1652582968747
      ```

  + response

    ```json
    {
       "janus": "server_info",							// janus的action
       "transaction": "NyCxLlMIe3W",
       "name": "Janus WebRTC Server",					// 服务器的名字，在 janus.c 中JANUS_NAME
       "version": 118,									// 版本数字，同上
       "version_string": "0.11.8",						// 版本字符串，同上
       "author": "Meetecho s.r.l.",						// 作者，在 janus.c 中 JANUS_AUTHOR
       "commit-hash": "not-a-git-repo",					// 提交hash
       "compile-time": "Fri Mar 25 20:42:34 PDT 2022",
       "log-to-stdout": true,							// 日志输出到标准输出
       "log-to-file": false,							// 日志输出到文件
       "data_channels": true,							// 主要用于点对点的文本、文件传输（例如：janus plugin text room）
       "accepting-new-sessions": true,					// 是否接受新的 sessions（例如：当服务器资源耗尽时，拒绝接受新的request）
       "session-timeout": 60,							// session 的过期时间
       "reclaim-session-timeout": 0,
       "candidates-timeout": 45,						// 处理webrtc（ice）的候选者连接超时时间（例如：音视频通道连接超过45秒就抛弃这个通道）
       "server-name": "MyJanusInstance",				// 服务器实例的名字，在 janus.c 中 JANUS_SERVER_NAME
       "local-ip": "192.168.1.51",
       "ipv6": false,
       "ice-lite": false,								// ice 的轻量版本，仅用于响应来自其它端的 ice 请求,且它等待其它端发送给他应该如何建立媒体流通道。（用于公网部署）
       "ice-tcp": false,								// ice 的连接采用 tcp 方式，有udp用 udp，没 udp 用 tcp。（注：libnice 小于  0.1.8 不支持 tcp）
       "ice-nomination": "aggressive",					// ice 的选举模式，即只要双方两端连接建立成功就立即通过优先级最高的连接发送音视频流。相应的regular模式会先check，然后根据双方网络状态等一系列因素选出最好的连接进行流传输
       "ice-keepalive-conncheck": false,				// ice 连接保活，在发送媒体之前会处理保活请求（注：由于 libnice 的问题，目前此参数有bug，不建议开启）
       "full-trickle": false,							// full 是连接的双方都进行连通性检查（注意：可能降低音视频通话连接建立速度，因为c与s端都要收集候选者并进行连通性检查）				
       "mdns-enabled": true,							// 启用Multicast-DNS，如果false会移除localhost的候选者，如果不是内网使用，建议设置为false
       "min-nack-queue": 200,							// 每个句柄存储用于重传的数据包的最小时间窗口
       "nack-optimizations": false,						// 发送关键帧前清空 nack 队列（注意：不建议打开）
       "twcc-period": 200,								// webrtc 拥塞算法。定期对接收方进行反馈
       "dtls-mtu": 1200,								// dtls 的最大传输单元
       "static-event-loops": 0,							// ice 的事件循环
       "api_secret": false,								// 所有请求都带有密码
       "auth_token": false,								// 所有请求都带有 token
       "event_handlers": false,							// 是否启动事件处理机制，用于 janus 内部事件处理
       "opaqueid_in_api": false,						// janus内部事件使用的id
       "dependencies": {
          "glib2": "2.56.1",
          "jansson": "2.10",
          "libnice": "0.1.18",
          "libsrtp": "libsrtp2 2.4.2",
          "crypto": "OpenSSL 1.0.2k-fips  26 Jan 2017"
       },
       "transports": {
          "janus.transport.http": {
             "name": "JANUS REST (HTTP/HTTPS) transport plugin",
             "author": "Meetecho s.r.l.",
             "description": "This transport plugin adds REST (HTTP/HTTPS) support to the Janus API via libmicrohttpd.",
             "version_string": "0.0.2",
             "version": 2
          },
          "janus.transport.websockets": {
             "name": "JANUS WebSockets transport plugin",
             "author": "Meetecho s.r.l.",
             "description": "This transport plugin adds WebSockets support to the Janus API via libwebsockets.",
             "version_string": "0.0.1",
             "version": 1
          }
       },
       "events": {},
       "loggers": {},
       "plugins": {
          "janus.plugin.voicemail": {
             "name": "JANUS VoiceMail plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a plugin implementing a very simple VoiceMail service for Janus, recording Opus streams.",
             "version_string": "0.0.7",
             "version": 7
          },
          "janus.plugin.audiobridge": {
             "name": "JANUS AudioBridge plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a plugin implementing an audio conference bridge for Janus, mixing Opus streams.",
             "version_string": "0.0.12",
             "version": 12
          },
          "janus.plugin.echotest": {
             "name": "JANUS EchoTest plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a trivial EchoTest plugin for Janus, just used to showcase the plugin interface.",
             "version_string": "0.0.7",
             "version": 7
          },
          "janus.plugin.recordplay": {
             "name": "JANUS Record&Play plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a trivial Record&Play plugin for Janus, to record WebRTC sessions and replay them.",
             "version_string": "0.0.4",
             "version": 4
          },
          "janus.plugin.textroom": {
             "name": "JANUS TextRoom plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a plugin implementing a text-only room for Janus, using DataChannels.",
             "version_string": "0.0.2",
             "version": 2
          },
          "janus.plugin.videoroom": {
             "name": "JANUS VideoRoom plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a plugin implementing a videoconferencing SFU (Selective Forwarding Unit) for Janus, that is an audio/video router.",
             "version_string": "0.0.9",
             "version": 9
          },
          "janus.plugin.videocall": {
             "name": "JANUS VideoCall plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a simple video call plugin for Janus, allowing two WebRTC peers to call each other through a server.",
             "version_string": "0.0.6",
             "version": 6
          },
          "janus.plugin.nosip": {
             "name": "JANUS NoSIP plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a simple RTP bridging plugin that leaves signalling details (e.g., SIP) up to the application.",
             "version_string": "0.0.1",
             "version": 1
          },
          "janus.plugin.streaming": {
             "name": "JANUS Streaming plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a streaming plugin for Janus, allowing WebRTC peers to watch/listen to pre-recorded files or media generated by an external source.",
             "version_string": "0.0.8",
             "version": 8
          },
          "janus.plugin.sip": {
             "name": "JANUS SIP plugin",
             "author": "Meetecho s.r.l.",
             "description": "This is a simple SIP plugin for Janus, allowing WebRTC peers to register at a SIP server and call SIP user agents through a Janus instance.",
             "version_string": "0.0.8",
             "version": 8
          }
       }
    }
    ```
    

+ ping：一个简单的ping/pong机制，客户端可用该接口来链路检查及预测协议的路由时间，也是不需要秘钥即可请求

  + request

    ```json
    {
        "janus": "ping",
        "transaction": "l1Mrk00izyzc"
    }
    ```

  + response

    ```json
    {
        "janus": "pong",
        "transaction": "l1Mrk00izyzc"
    }
    ```

    

+ loops_info: 返回每个静态事件循环当前负责多少个句柄的摘要，以防静态事件循环正在使用

  + request

    ```json
    {
        "janus": "loops_info",
        "transaction": "l1Mrk00izyzc",
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "loops": []
    }
    ```

### 配置相关请求

请求连接上同

+ get_status：返回当前服务器能被Admin API动态修改的配置变量

  + request:

    ```json
    {
    	"janus":	"get_status",
    	"transaction":	"l1Mrk00izyzc",
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "status": {
            "token_auth": false,					// 启用请求带 token
            "session_timeout": 60,					// session 过期时间
            "reclaim_session_timeout": 0,			// 过期后，任能唤醒 session 的时间
            "candidates_timeout": 45,				// 候选超时时间
            "log_level": 4,							// 日志输出等级
            "log_timestamps": false,				// 在每行日志前都加时间戳
            "log_colors": true,						// 日志输出颜色。在 console
            "locking_debug": false,					// 锁的实时调试，遇到僵尸进程有用
            "refcount_debug": false,				// 参考计数器的实时调试, 遇到内存泄漏有用
            "libnice_debug": false,					// ice 的调试
            "min_nack_queue": 200,					// 每个句柄存储用于重传的数据包的最小时间窗口
            "nack-optimizations": false,			// 发送关键帧前清空 nack 队列
            "no_media_timer": 1,					// 改变没有媒体流的计时器值
            "slowlink_threshold": 0					// 每秒丢包数，超过阈值就发送 slow_link 事件到客户端，0 不发送事件
        }
    }
    ```

- set_session_timeout: 改变全局的会话超时时间

  - request

    ```json
    {
    	"janus":	"set_session_timeout",
    	"transaction":	"l1Mrk00izyzc",
        "timeout": 62,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
    	"janus":	"set_session_timeout",
    	"transaction":	"l1Mrk00izyzc",
        "timeout": 62,
        "admin_secret": "janusoverlord"
    }
    ```

- set_log_level: 改变日志等级

  - request

    ```json
    {
    	"janus":	"set_log_level",
    	"transaction":	"l1Mrk00izyzc",
        "level": 6,							// 0-7之间的值
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "level": 6
    }
    ```

- set_log_timestamps:使能或禁止在日志中添加时间戳

  + request

    ```json
    {
    	"janus":	"set_log_timestamps",
    	"transaction":	"l1Mrk00izyzc",
        "timestamps": false,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "log_timestamps": false
    }
    ```

- set_log_colors: 使能或禁止日志带颜色输出

  - request

    ```json
    {
    	"janus":	"set_log_colors",
    	"transaction":	"l1Mrk00izyzc",
        "colors": true,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "log_colors": true
    }
    ```

- set_locking_debug: 使能或禁止锁（互斥锁等）的实时调试，在遇到死锁时很有用

  - request

    ```json
    {
    	"janus":	"set_locking_debug",
    	"transaction":	"l1Mrk00izyzc",
        "debug": false,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "locking_debug": false
    }
    ```

    

- set_refcount_debug:使能或禁止引用计数的实时调试，在遇到janus内部结构体内存泄漏时使用

  + request

    ```json
    {
    	"janus":	"set_refcount_debug",
    	"transaction":	"l1Mrk00izyzc",
        "debug": false,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "refcount_debug": false
    }
    ```

- set_libnice_debug:使能或禁止libnice库的调试

  - request

    ```json
    {
    	"janus":	"set_libnice_debug",
    	"transaction":	"l1Mrk00izyzc",
        "debug": false,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "libnice_debug": false
    }
    ```

- set_min_nack_queue:改变NACK队列的最小值

  - request

    ```json
    {
    	"janus":	"set_min_nack_queue",
    	"transaction":	"l1Mrk00izyzc",
        "min_nack_queue": 200,
        "admin_secret": "janusoverlord"
    }
    ```

  - response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "min_nack_queue": 200
    }
    ```

- set_no_media_timer: 改变没有媒体流的计时器值

  - request

    ```json
    {
    	"janus":	"set_no_media_timer",
    	"transaction":	"l1Mrk00izyzc",
        "no_media_timer": 1,
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "no_media_timer": 1
    }
    ```

    

- set_slowlink_threshold: 每秒丢包数，超过阈值就发送 slow_link 事件到客户端，0 不发送事件

  + request

    ```json
    {
    	"janus":	"set_slowlink_threshold",
    	"transaction":	"l1Mrk00izyzc",
        "slowlink_threshold": 0,
        "admin_secret": "janusoverlord"
    }
    ```

  + reponse

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "slowlink_threshold": 0
    }
    ```

### Token 相关的请求

这部分请求只有在使能了token存储的机制之后才有效，即上面描述的 auth_token，不能使用token_auth_secret，否则就无法使用以下 api。后续的 create_session 类似都需要带上 token，否则会被拒绝

- add_token: 增加一个有效tocken

  + request

    ```json
    {
        "janus": "add_token",
        "token": "1234",
        "plugins": [
            "janus.plugin.voicemail",
            "janus.plugin.audiobridge",
            "janus.plugin.echotest",
            "janus.plugin.recordplay",
            "janus.plugin.textroom",
            "janus.plugin.videoroom",
            "janus.plugin.videocall",
            "janus.plugin.nosip",
            "janus.plugin.streaming",
            "janus.plugin.sip"
        ],
        "transaction": "CJjN1NTwZvui",
        "admin_secret": "janusoverlord"
    }
    ```

  + reponse

    ```json
    {
       "janus": "success",
       "transaction": "CJjN1NTwZvui",
       "data": {
          "plugins": [
             "janus.plugin.voicemail",
             "janus.plugin.audiobridge",
             "janus.plugin.echotest",
             "janus.plugin.recordplay",
             "janus.plugin.textroom",
             "janus.plugin.videoroom",
             "janus.plugin.videocall",
             "janus.plugin.nosip",
             "janus.plugin.streaming",
             "janus.plugin.sip"
          ]
       }
    }
    ```

    

- allow_token: 给与一个与插件使用相关的token

  + request

    ```json
    {
        "janus": "allow_token",
        "token": "1234",
        "plugins": [
            "janus.plugin.voicemail",
            "janus.plugin.audiobridge"
        ],
        "transaction": "CJjN1NTwZvui",
        "admin_secret": "janusoverlord"
    }
    ```

  + reponse

    ```json
    {
        "janus": "success",
        "transaction": "CJjN1NTwZvui",
        "data": {
            "plugins": [
                "janus.plugin.voicemail",
                "janus.plugin.audiobridge"
            ]
        }
    }
    ```

    

- disallow_token:移除一个与插件使用有关的token

  + request

    ```json
    {
        "janus": "disallow_token",
        "token": "1234",
        "plugins": [
            "janus.plugin.voicemail",
            "janus.plugin.audiobridge"
        ],
        "transaction": "CJjN1NTwZvui",
        "admin_secret": "janusoverlord"
    }
    ```

  + reponse

    ```json
    {
        "janus": "success",
        "transaction": "CJjN1NTwZvui",
        "data": {
            "plugins": []
        }
    }
    ```

    

- list_tokens: 列出现存的所有token

  - request

    ```json
    {
        "janus": "list_tokens",
        "transaction": "CJjN1NTwZvui",
        "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "CJjN1NTwZvui",
        "data": {
            "tokens": [
                {
                    "token": "1234",
                    "allowed_plugins": [
                        "janus.plugin.voicemail",
                        "janus.plugin.audiobridge",
                        "janus.plugin.echotest",
                        "janus.plugin.recordplay",
                        "janus.plugin.textroom",
                        "janus.plugin.videoroom",
                        "janus.plugin.videocall",
                        "janus.plugin.nosip",
                        "janus.plugin.streaming",
                        "janus.plugin.sip"
                    ]
                }
            ]
        }
    }
    ```

- remove_token:删除一个token

  + request

    ```json
    {
      "janus": "remove_token",
      "token": "1234",
      "transaction": "eVyLDzjqfyg3",
      "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
       "janus": "success",
       "transaction": "eVyLDzjqfyg3"
    }
    ```

  

### 会话相关的请求

- accept_new_sessions: 配置janus是否应该接受一个新的会话（当你任何时候想要清空一个janus实例使用这个非常方便）

  + request:

    ```json
    {
    	"janus":	"accept_new_sessions",
    	"transaction":	"l1Mrk00izyzc",
        "accept": false,
        "admin_secret": "janusoverlord"
    }
    ```

  + reponse

    ```json
    {
        "janus": "success",
        "transaction": "l1Mrk00izyzc",
        "accept": false
    }
    ```

- list_sessions:列出现存的active状态的会话

  + request

    ```json
    {
      "janus": "list_sessions",
      "transaction": "z4a9ie1CZD97",
      "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "z4a9ie1CZD97",
        "sessions": [
            5861290772059172
        ]
    }
    ```

- destory_session: 销毁一个会话

  - request

    ```json
    {
    	"janus":	"destroy",
    	"transaction":	"l1Mrk00izyzr",
        "session_id": 3086529275573380
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "session_id": 3086529275573380,
        "transaction": "l1Mrk00izyzr"
    }
    ```

### 句柄及webrtc相关的请求

- list_handles: 列出现存的active状态的ICE句柄

  - url：`https://ip:port/admin/session_id`

  - request

    ```json
    {
      "janus": "list_handles",
      "transaction": "ZBJomuxVRUyQ",
      "admin_secret": "janusoverlord"
    }
    ```

  - response

    ```json
    {
       "janus": "success",
       "session_id": 2548810000107482,
       "transaction": "ZBJomuxVRUyQ",
       "handles": [
          743792752849709
       ]
    }
    ```

    

- handle_info: 列出特定ICE句柄的所有信息。如果将plugin_only变量值设置为true则仅返回插件相关的信息，webrtc信息及状态均不会被包含在返回信息中。

  - url：`https://ip:port/admin/session_id/handle_id`

  - request

    ```json
    {
      "janus": "handle_info",
      "transaction": "DwzOP5mrCmX6",
      "admin_secret": "janusoverlord"
    }
    ```

  + response（video-room）

    ```json
    {
        "janus": "success",
        "session_id": 2548810000107482,
        "transaction": "DwzOP5mrCmX6",
        "handle_id": 743792752849709,
        "info": {
            "session_id": 2548810000107482,
            "session_last_activity": 152840675914,
            "session_timeout": 60,
            "session_transport": "janus.transport.http",
            "handle_id": 743792752849709,
            "opaque_id": "videoroomtest-jDH6WBjAcyI4",
            "loop-running": true,
            "created": 152597708101,
            "current_time": 152852556490,
            "plugin": "janus.plugin.videoroom",
            "plugin_specific": {
                "type": "publisher",
                "room": 1234,
                "id": 4406738697513809,
                "private_id": 1323301421,
                "display": "123",
                "media": {
                    "audio": true,
                    "audio_codec": "opus",
                    "video": true,
                    "video_codec": "vp8",
                    "data": false
                },
                "bitrate": 128000,
                "audio-level-dBov": 97,
                "talking": false,
                "hangingup": 0,
                "destroyed": 0
            },
            "flags": {
                "got-offer": true,
                "got-answer": true,
                "negotiated": true,
                "processing-offer": false,
                "starting": true,
                "ice-restart": false,
                "ready": true,
                "stopped": false,
                "alert": false,
                "trickle": true,
                "all-trickles": true,
                "resend-trickles": false,
                "trickle-synced": false,
                "data-channels": false,
                "has-audio": true,
                "has-video": true,
                "new-datachan-sdp": false,
                "rfc4588-rtx": true,
                "cleaning": false,
                "e2ee": false
            },
            "agent-created": 152600228063,
            "ice-mode": "full",
            "ice-role": "controlled",
            "sdps": {
                "profile": "UDP/TLS/RTP/SAVPF",
                "local": "sdp",
                "remote": "sdp"
            },
            "queued-packets": 0,
            "streams": [
                {
                    "id": 1,
                    "ready": -1,
                    "ssrc": {
                        "audio": 997893997,
                        "video": 1527131261,
                        "video-rtx": 3707199999,
                        "audio-peer": 3279797068,
                        "video-peer": 1111535207,
                        "video-peer-rtx": 996540079
                    },
                    "direction": {
                        "audio-send": false,
                        "audio-recv": true,
                        "video-send": false,
                        "video-recv": true
                    },
                    "codecs": {
                        "audio-pt": 111,
                        "audio-codec": "opus",
                        "video-pt": 96,
                        "video-rtx-pt": 97,
                        "video-codec": "vp8"
                    },
                    "extensions": {
                        "urn:ietf:params:rtp-hdrext:sdes:mid": 4,
                        "urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id": 10,
                        "urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id": 11,
                        "http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01": 3,
                        "urn:ietf:params:rtp-hdrext:ssrc-audio-level": 1,
                        "urn:3gpp:video-orientation": 13
                    },
                    "bwe": {
                        "twcc": true,
                        "twcc-ext-id": 3
                    },
                    "nack-queue-ms": 200,
                    "rtcp_stats": {
                        "audio": {
                            "base": 48000,
                            "rtt": 0,
                            "lost": 0,
                            "lost-by-remote": 0,
                            "jitter-local": 1,
                            "jitter-remote": 0,
                            "in-link-quality": 100,
                            "in-media-link-quality": 100,
                            "out-link-quality": 0,
                            "out-media-link-quality": 0
                        },
                        "video": {
                            "base": 90000,
                            "rtt": 0,
                            "lost": 0,
                            "lost-by-remote": 0,
                            "jitter-local": 3,
                            "jitter-remote": 0,
                            "in-link-quality": 100,
                            "in-media-link-quality": 100,
                            "out-link-quality": 0,
                            "out-media-link-quality": 0
                        }
                    },
                    "components": [
                        {
                            "id": 1,
                            "state": "ready",
                            "connected": 152600440611,
                            "local-candidates": [
                                "1 1 udp 2015363327 192.168.1.51 54605 typ host"
                            ],
                            "remote-candidates": [
                                "1755041049 1 udp 2122260223 192.168.74.1 51297 typ host generation 0 ufrag bTNl network-id 1",
                                "2530088836 1 udp 2122129151 192.168.1.106 51299 typ host generation 0 ufrag bTNl network-id 3",
                                "2272787581 1 udp 2122194687 172.16.100.1 51298 typ host generation 0 ufrag bTNl network-id 2"
                            ],
                            "selected-pair": "192.168.1.51:54605 [host,udp] <-> 192.168.1.106:51299 [host,udp]",
                            "dtls": {
                                "fingerprint": "2E:31:41:43:A9:AC:1F:58:AC:57:D7:15:6C:7E:DB:99:1D:13:58:4A:89:31:43:CB:F8:9F:97:26:2E:66:B9:76",
                                "remote-fingerprint": "CE:70:89:E8:0E:A0:F0:FC:42:7E:CB:75:54:29:C8:4A:A4:2D:C8:36:70:37:57:59:A8:AB:9D:62:19:A9:B6:A7",
                                "remote-fingerprint-hash": "sha-256",
                                "dtls-role": "active",
                                "dtls-state": "connected",
                                "retransmissions": 0,
                                "valid": true,
                                "srtp-profile": "SRTP_AES128_CM_SHA1_80",
                                "ready": true,
                                "handshake-started": 152600440643,
                                "connected": 152600447955,
                                "sctp-association": false
                            },
                            "in_stats": {
                                "audio_packets": 12606,
                                "audio_bytes": 775132,
                                "audio_bytes_lastsec": 3036,
                                "do_audio_nacks": false,
                                "video_packets": 7031,
                                "video_bytes": 1870792,
                                "video_bytes_lastsec": 6938,
                                "do_video_nacks": true,
                                "video_nacks": 0,
                                "video_retransmissions": 0,
                                "data_packets": 3,
                                "data_bytes": 2247
                            },
                            "out_stats": {
                                "audio_packets": 0,
                                "audio_bytes": 0,
                                "audio_bytes_lastsec": 0,
                                "audio_nacks": 0,
                                "video_packets": 0,
                                "video_bytes": 0,
                                "video_bytes_lastsec": 0,
                                "video_nacks": 0,
                                "data_packets": 3,
                                "data_bytes": 2298
                            }
                        }
                    ]
                }
            ]
        }
    }
    ```

    

- start_pcap: 开始将发送的RTP/RTCP包抓包保存成pcap文件

  - url：`https://ip:port/admin/session_id/handle_id`

  - request:

    ```json
    {
      "janus": "start_pcap",
      "transaction": "YHBmWC0jO2ax",
      "admin_secret": "janusoverlord",
      "folder": "/root/",
      "filename": "b.pcap",
      "truncate": 0							// 抓取多少包截断，0 不截断
    }
    ```

  - response:

    ```json
    {
        "janus": "success",
        "transaction": "YHBmWC0jO2ax"
    }
    ```

- stop_pcap:停止抓包

  - url：`https://ip:port/admin/session_id/handle_id`

  - request

    ```json
    {
      "janus": "stop_pcap",
      "transaction": "40dHehG1RkxP",
      "admin_secret": "janusoverlord"
    }
    ```

  - response

    ```json
    {
        "janus": "success",
        "transaction": "40dHehG1RkxP"
    }
    ```

- start_text2pcap:和start_pcap请求一样，但是保存为text文件。上同，其中 start_pcap 改为 start_text2pcap

- stop_text2pcap:停止text2pcap抓包。上同，其中 stop_pcap 不变

- message_plugin:发送同步消息给插件。几乎所有插件都实现该请求用于优化该插件的资源管理。管理插件内容，建议查看对应插件提供的同步 api，如 [janus的videoroom插件](https://zhuanlan.zhihu.com/p/150461418)。（注：在 postman有所有的使用示例）

  + request

    ```json
    {
      "janus": "message_plugin",
      "plugin": "janus.plugin.videoroom",
      "transaction": "40dHehG1RkxP",
      "admin_secret": "janusoverlord",
      "request":{
          "request": "list"
      }
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "transaction": "40dHehG1RkxP",
        "response": {
            "videoroom": "success",
            "list": [
                {
                    "room": 1234,
                    "description": "Demo Room",
                    "pin_required": false,
                    "is_private": false,
                    "max_publishers": 6,
                    "bitrate": 128000,
                    "fir_freq": 10,
                    "require_pvtid": false,
                    "require_e2ee": false,
                    "notify_joining": false,
                    "audiocodec": "opus",
                    "videocodec": "vp8",
                    "opus_fec": true,
                    "record": false,
                    "lock_record": false,
                    "num_participants": 1,
                    "audiolevel_ext": true,
                    "audiolevel_event": true,
                    "audio_active_packets": 100,
                    "audio_level_average": 25,
                    "videoorient_ext": true,
                    "playoutdelay_ext": true,
                    "transport_wide_cc_ext": true
                }
            ]
        }
    }
    ```

    

- handup_webrtc: 挂断特定ICE句柄的PeerConnection，结果类似与主动挂断某个用户

  - url：`https://ip:port/admin/session_id/handle_id`

  - request

    ```json
    {
      "janus": "hangup_webrtc",
      "transaction": "40dHehG1RkxP",
      "admin_secret": "janusoverlord"
    }
    ```

  + response

    ```json
    {
        "janus": "success",
        "session_id": 4537884414371202,
        "transaction": "40dHehG1RkxP"
    }
    ```

    

- detach_handle:分离特定的句柄，和 janus API 的 detach 消息的功能一致。功能与 attach 相反

  - url：`https://ip:port/admin/session_id/handle_id`

  - request

    ```json
    {
      "janus": "detach_handle",
      "transaction": "40dHehG1RkxP",
      "admin_secret": "janusoverlord"
    }
    ```

  - response

    ```json
    {
        "janus": "success",
        "session_id": 4537884414371202,
        "transaction": "40dHehG1RkxP"
    }
    ```

    

## 引用

1. [janus的videoroom插件](https://zhuanlan.zhihu.com/p/150461418)

