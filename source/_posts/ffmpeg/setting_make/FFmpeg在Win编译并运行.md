---
title: FFmpeg在Win编译并运行
date: 2022-04-04 15:27:05
tags: make
category: FFmpeg
---

本人选择编译的 FFmpeg 不是在 Linux 环境下面，也不是在 Win 环境下面的模拟 Linux 环境（例如：mgwin、cygwin）。

使用 vs 搭建 FFmpeg，其主要目的是为了方便调试与学习。本人较为喜欢 vs 的调试。

采用的 [FFmpeg](https://github.com/ShiftMediaProject/FFmpeg) 是专门为 Win 环境做了适配。

## 环境

| 环境名 | 版本     |
|----   |----     |
| WIN   | WIN10   |
| VS    | vs2017  |

## 基础的搭建

关于最基本的搭建内容有很多文章都有关于这方面的叙述，此处我将只会叙述一个大概主要的安装流程。

至于其中更详细内容可以参考下面的引用

+ 第一步：创建文件夹目录，目录结构如下所示
    ```
    - msvc (OutputDir)　　　　　　　　　　　　　 　　
    - source　　　　　　　　　　　　　　　　　 　　　
    　　- FFmpeg 　　　　　　　　　　　　　　　　　　
    　　- ..Any other libraries source code..
    ```
+ 第二步：使用 FFmpeg 项目目录下面的 project_get_dependencies.bat 脚本运行并获取依赖项目。
   > 注意：此处由于国内的原因，可以采用代理或者直接网上下载的方式获取，建议采用脚本，方便更新
+ 第三步：下载额外依赖的头文件到 msvc 的 include 的目录中
    + [下载](https://www.khronos.org/registry/OpenGL/api/GL/) opengl 的 glext.h 和 wglext.h  到  " msvc/include/gl/* "
    + [下载](https://www.khronos.org/registry/EGL/api/KHR/) opengl 的 khrplatform.h 到 " msvc/include/KHR/* "
    + [下载](https://github.com/FFmpeg/nv-codec-headers)  nv-codec-headers 项目的 "include" 文件夹下的内容到  " msvc/include/* "
    + [下载](https://github.com/GPUOpen-LibrariesAndSDKs/AMF) AMF 项目的  "amf/public/include" 文件夹下的内容到  " msvc/include/AMF/* "
+ 第四步：安装 [nasm](https://github.com/ShiftMediaProject/VSNASM/releases/latest) 与 [yasm](https://github.com/ShiftMediaProject/VSYASM/releases)
。将内容下载到本地后解压到 msvc 目录并分别使用管理员权限执行其中的 install_script.bat 脚本
+ 第五步：使用 visual studio 打开 **FFmpeg/SMP/ffmpeg_desp.sln** 项目文件

以上是编译时候的主要步骤，下面是在此过程中所遇到的问题以及解决方案

## 问题与解决

+ 关于依赖的缺失：我这边由于是新项目，网络上的很多描述没涉及到其中缺失的一些项目。
> 解决方案：在 ShiftMediaProject 这个 github 账号里面将缺失的项目都下载或者克隆到 source 目录下面，然后重启 vs 即可

+ 关于 h264 与 h265 项目无法编译：在保证 yasm 与 nasm 正确的情况下，仍然无法编译，编译器一直提示缺少 win sdk8.1
> 解决方案：通过 vs intall 下载安装好 sdk8.1 的内容

+ 关于新项目无法加载 uwp 结尾的项目问题：此问题是本地项目中没有安装 uwp 的库
> 解决方案：同样可以通过安装 uwp 的内容解决，实际情况以个人开发者的环境为准

+ 关于一些项目缺失某些头文件：注意查看情况，缺失头文件一般有两种情况。一种是这个项目的依赖项目编译失败，查看方式就是去 msvc/include 下面看是否有这个依赖项目的头文件
；第二种情况是这个项目依赖的 submodule 没有下载成功（例如：gnulib），查看方式是报错的时候会指向具体的某个编译的项目。
> 解决方案：如果是 submodule ，那么可以使用 `git submodule update` 来获取子项目，一直获取失败的话可以直接下载后拷贝进去。如果是
>项目缺失的直接按照第一条的解决方式即可解决 

最后，总结下。如果缺失某些头文件，请仔细思考是否是该项目的依赖项目编译失败。直接单独编译该项目或指明的依赖项目就可以准确定位了。
## 引用

[使用 VS2015 编译并调试 ffmpeg](https://www.cnblogs.com/BensonLaur/p/10989115.html)

