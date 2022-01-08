---
title: WebRTC 在 Win 中源代码编译
date: 2022-01-01 20:30:20
tags: WebRTC编译
category: WebRTC
---

本文主要讲解本人在 win 中编译 WebRTC 的过程中所遇到的问题与思考。

## gclient

“工欲善其事，必先利其器”。那么我们必须先对这个工具有一定的认知。

### 为什么需要 gclient

gclient 是谷歌开发的一款跨平台 git 仓库管理平台。

为什么要有这个呢？因为像这种大项目，被拆分成很多个子项目，子项目分别进行研发，这样的好处就是能够复用大量代码，方便多人开发（每个团队只负责一个项目），每个子项目都有版本规划。

那么集成代码的时候怎么办呢？难道要开发人员记得每个项目的版本号吗？比如：我们需要 webrtc M87，里面分成很多子项目，每个子项目都有对应的版本号，如果版本号错了可能存在接口兼容问题。

通过上诉问题，我们就知道 gclient 的作用了。本质就是方便集成所有的仓库的内容

### gclient是如何运行

gclient 将多个仓库组成一个 solution（解决方案）进行管理。这个解决方案用 .gclient 文件进行描述，其中描述了需要拉取那个解决方案，可以通过`gclient conf`创建文件。例如：我们要拉取 webrtc 这个解决方案，就在里面配置 webrtc 的链接。如下所示：
```
// 注意：此仅为示例内容，不具有实际价值
solutions = [
  { "name"        : 'src',
    "url"         : 'https://chromium.googlesource.com/external/webrtc.git@gitlab',
    "deps_file"   : 'DEPS',
    "managed"     : True,
    "custom_deps" : {
    },
    "custom_vars": {},
  },
]
```

那么项目之间的依赖关系用什么描述呢？这个内容就用 DEPS 这个特殊文件进行描述。

### gclient使用问题

在使用 gclient 进行拉取项目前，我们需要运行 `gclient` 命令，这个命令会自动为我们下载 git、python 的环境。由于下载地方在国外，所以需要开着代理进行下载。但是 gclient 下载仍然会出错，下面就是我的出错内容。

```
// 以自己的环境为主
set vs2017_install=D:\Program Files (x86)\Microsoft Visual Studio\2017\Community
set GYP_MSVS_OVERRIDE_PATH=D:\Program Files (x86)\Microsoft Visual Studio\2017\Community 
set GYP_GENERATORS=msvs-ninja,ninja
# 告诉depot_tools使用我们本机的VS进行编译
set DEPOT_TOOLS_WIN_TOOLCHAIN=0

gclient 

```

关于 gclient 的详细使用方法请查看引用中的[WebRTC client 源码环境工具配置](https://www.cnblogs.com/baitongtong/p/9561753.html)

> 注意：此处还在配置 gclient 。

#### 代理设置

本人在 win 中使用的是 [v2rayN](https://github.com/2dust/v2rayN/releases) 软件，本人使用时打开了**http代理和PAC全局模式修改**，再去 cmd 中配置代理地址，指向了http代理。

```shell
// 以你本地代理为准
set http_proxy=127.0.0.1:10809
set https_proxy=127.0.0.1:10809
```

> 注意：我这里很奇怪，如果不打开全局模式就算设置了代理也没法使用

#### cipd_client.exe 下载失败

开着代理下载 cipd 任然失败，我在此处卡了很久。根据网上所说直接将文件用浏览器下载，然后改名成 .cipd_client.exe 重新使用 gclient 就执行成功。

#### runhooks 失败

失败信息如下：

```
NOTICE:You have PROXY values set in your environment,......BOTO_CONFIG......NO_AUTH_BOTO_CONFIG environment var.
```

解决方法：新建一个文件，http_proxy.boto ，放在任意位置，我放到了 D:\depot_tools 中，文件内容如下：

```
[Boto]
proxy=127.0.0.1
proxy_port = 1080

```

## 源代码同步

我是直接使用以下命令就可以获取到源代码
```shell

mkdir webrtc-checkout
cd webrtc-checkout
fetch --nohooks webrtc
gclient sync

```

### 同步过程中遇到的问题

同步过程中本质还是要保证与远端外网的网络保持正常连接，我在保证网络连接正常的情况下也就一次性过了。

#### clang下载失败问题

我这边下载失败有两个问题。

其一：WebRTC 中的代码有问题，在 src/DEPS 中所有`download_from_google_storage`应该改为`src/third_party/depot_tools/download_from_google_storage.py`，因为我们没有将这个 py 文件设置为环境变量，那么我们就必须写全路径，这个怀疑是 webrtc 在不同开发人员中做了不同操作导致的

其二：就是真正的无法下载 clang 问题。这里我参考了[webrtc gclient sync运行后clang下载失败的解决办法！](https://blog.csdn.net/malihong1/article/details/73500806)的思路解决了问题。这里推荐使用 switchhost 这款软件，帮助我们更好的修改域名。

其三：这个不是问题。只是简单说下这个下载过程需要保证两个域名的正常访问。经过我的测试，这两个少一个都不行，但是上面的文章没有具体说出来。如下所示：
```
// 写于2021/7/10,以你的访问为准
142.250.206.208 commondatastorage.googleapis.com
183.236.60.194 android.googlesource.com
```

最后，这里说个小技巧。通过将 `https://commondatastorage.googleapis.com/chromium-browser-clang` 连接放到浏览器中访问，然后打开 F11 ，找到 network 并选择连接，查看 http 响应头中的 remote-ip 。这个技巧可以帮你更好的获取到可以访问的ip

> 注意：在这个过程中，必须一直保持代理打开着并保持连接

## 生成 vs 的解决方案

生成解决方案的时候，采用如下命令：

```
cd src
gn gen --ide=vs2017 out/Default
```

### 生成解决方案遇到的问题

#### 没有gn命令

很真实！我这个 cmd 环境中没有设置 gn 的环境变量，那么我就直接使用全路径。主要是不容易出错。路径如下所示：

```
E:\workspace\webrtc\source\src\buildtools\win\gn.exe
```

#### 遇到 WINDOWSSDKDIR 问题

这个问题的本质就是我没将 sdk 安装到 C 盘中，因而出现无法找到路径。[webrtc 编译报错 WINDOWSSDKDIR KeyError: ‘WINDOWSSDKDIR‘](https://blog.csdn.net/qq_40340448/article/details/115380257)这篇文章描述很详细了，就是将你安装在其它盘中的 sdk 的文件拷贝到 `C:\Program Files (x86)\Windows Kits\`中

这里我补充两点，因为这是我生成过程中遇到的问题。

第一：一般不好找到你的 sdk 文件在哪里，可以采用 everything 这个软件搜索 Windows Kits 。当然如果你能自己找到最好！

第二：需要额外再复制拷贝一个 Debuggers 和 Redist 内容到 C 盘下面。

#### SDK 版本太低

这个问题是 vs2017 默认使用 10.0.17763.0，但是 WebRTC 要求 10.0.19041 或者更高版本的 Windows 10 SDK。文档描述在 [win 编译 WebRTC 环境要求](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md)一文中。

可以下载 SDK 安装的软件，安装我们需要的版本 `https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk/`

> 注意：将你新下载的内容拷贝到 C 盘中哦！

## 引用

[Windows 平台WebRTC编译-VS2017](https://blog.jianchihu.net/webrtc-build-vs2017.html)

[关于 debug 安装](https://blog.csdn.net/aaronjny/article/details/79828939)

[windows 平台下载并编译 webrtc 代码（代理）](https://www.shangmayuan.com/a/7743c98044a240df8654f5d5.html)

[WebRTC client 源码环境工具配置](https://www.cnblogs.com/baitongtong/p/9561753.html)

[PAC 模式和全局代理模式的区别与对比](https://blog.csdn.net/ununie/article/details/90022020)

[WebRTC 编译报错 WINDOWSSDKDIR KeyError: ‘WINDOWSSDKDIR‘](https://blog.csdn.net/qq_40340448/article/details/115380257)

[webrtc gclient sync运行后clang下载失败的解决办法！](https://blog.csdn.net/malihong1/article/details/73500806)

[win 编译 WebRTC 环境要求](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md)
