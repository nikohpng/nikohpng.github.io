---
title: FFmpeg的配置安装错误记录
date: 2022-01-25 21:06:36
tags: make
category: FFmpeg
---

本文主要记录本人在编译安装 FFmpeg 的时候出现的各种问题，这种问题出现在各种环境中，因而我会标明不同的环境下面。

当然主要环境分为三大类 Win、Linux和交叉编译，Mac本人没有就不算在里面。

## `“libavutil/avconfig.h”: No such file or directory`

+ 背景：我在 Win 环境中编译 FreeSWITCH 的 mod_av 模块的时候，这个模块依赖于 FFmpeg，因而总是报这个错误。

+ 错误信息：
    ![avconfig_err](../../../../../../images/ffmpeg/setting_make/questions/avconfig_err.png)

+ 解决：
