---
title: FreeSWITCH在Linux下编译模块编译
date: 2023-03-22 21:21:15
tags: mod
category: FreeSWTICH
---

本文主要讲解在 Linux 环境下编译 FreeSWITCH 模块时的配置以及所遇到的问题

## Linux 下编译 MOD 的配置

+ 在 configure.ac 中配置需要编译模块的 makefile文件，例如：src/mod/endpoints/mod_verto/Makefile
    ```shell
    PKG_CHECK_MODULES([HIREDIS], [hiredis >= 0.10.0],[
    AM_CONDITIONAL([HAVE_HIREDIS],[true])],[
    AC_MSG_RESULT([no]); AM_CONDITIONAL([HAVE_HIREDIS],[false])])
    
    PKG_CHECK_MODULES([REDIS_PLUS], [redis++ >= 1.3.10],[
    AM_CONDITIONAL([HAVE_REDIS_PLUS],[true])],[
    AC_MSG_RESULT([no]); AM_CONDITIONAL([HAVE_REDIS_PLUS],[false])])
    
    src/mod/applications/mod_redis_plus/Makefile
    ```
+ 在 modules.conf 中配置允许编译的模块，例如：endpoints/mod_verto
+ 运行 `autoreconf -fiv` 重新生成 m4 文件
+ 通过 rebootstrap.sh or bootstrap.sh 重新生成 configure 编译文件

## 关于 Linux 下模块的 Makefile.am 编写事项



## 失败问题总结

+ 无法生成 Makefile.in 文件
    + 注意模块位置不要写错了，否则无法生成
