---
title: FreeSWTICH中自定义命令的添加
date: 2022-01-08 17:24:57
tags: command
category: FreeSWTICH
---

本文将介绍如何在 FreeSWITCH 中开发自己的命令。
本文需要在[FreeSWTICH在vs中添加一个测试模块](./FreeSWTICH在vs中添加一个测试模块)作为基础知识。

## 环境

| 环境名 | 版本     |
|----   |----     |
| WIN   | WIN10   |
| FS    | 1.10.0  |
| VS    | vs2017  |

## FreeSWITCH 模块基础知识

关于模块的定义、生命周期的开启与结束都在以下所示的定义中：

```c
SWITCH_MODULE_SHUTDOWN_FUNCTION(mod_xx_shutdown);
SWITCH_MODULE_LOAD_FUNCTION(mod_xx_load);
SWITCH_MODULE_RUNTIME_FUNCTION(mod_xx_runtime);

SWITCH_MODULE_DEFINITION(mod_xx, mod_xx_load, mod_xx_shutdown, mod_xx_runtime);
```

此行我们着重关注模块的启动，因为我们将要在里面放置我们需要添加的命令。
那我们直接开始上代码把！

## 添加一个命令

在 `SWITCH_MODULE_LOAD_FUNCTION`中添加指向处理命令的 mod_name_function 并
告知控制台接收的命令字符串有 `help`这个命令

```c
SWITCH_MODULE_LOAD_FUNCTION(mod_xx_load)
{
	switch_api_interface_t *api_interface;

	/* connect my internal structure to the blank pointer passed to me */
	*module_interface = switch_loadable_module_create_module_interface(pool, modname);

	SWITCH_ADD_API(api_interface, "mod_name", "mod description", mod_name_function, "syntax");
	switch_console_set_complete("add mod name help");

	/* indicate that the module should continue to be loaded */
	return SWITCH_STATUS_SUCCESS;
}
```

## 处理命令

处理命令，如果只输入了`mod_name`而没有相应的`help`那么我们直接输出 hello world，代码如下所示：

```c
SWITCH_STANDARD_API(srs_function)
{
	char *mycmd = NULL;
	switch_status_t status = SWITCH_STATUS_SUCCESS;

	if (zstr(cmd)) 
	{ 
		stream->write_function(stream, "%s", "hello world");
		goto done;
	}

done:
	switch_safe_free(mycmd);
	return status;
}
```
