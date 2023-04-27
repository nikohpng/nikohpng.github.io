---
title: WebRTC在ubuntu中编译android
date: 2023-03-30 19:59:43
tags: WebRTC编译
category: WebRTC
---

本文将会讲解在 ubuntu 中拉取代码 flutter 中的 [WebRTC](https://github.com/webrtc-sdk/webrtc.git) 并打包成 android 可以使用的 aar。

> 注意：编译 Android 的时候，建议在国外的服务器上编译，否则可能出现无法获取各种依赖文件的问题

## 系统选择

注意：在编译 android 端的 WebRTC 的时候必须选择 ubuntu，其中最好是采用以下版本
```
Ubuntu 14.04 LTS (trusty with EoL April 2022）
Ubuntu 16.04 LTS (xenial with EoL April 2024）
Ubuntu 18.04 LTS (bionic with EoL April 2028）
Ubuntu 20.04 LTS (focal with Eol April 2030)
Ubuntu 20.10 (groovy) 
Debian 10 (buster) or later
```
如果不是，那么可能遇到一些环境问题，需要手动修改源码

## 设置代理

拉取 WebRTC 全程需要代理软件，所以请先设置好代理软件
```shell
export http_proxy=http://192.168.101.21:7890
export https_proxy=http://192.168.101.21:7890
```

## 安装 depot_tools

depot_tools 是一套 Google 用来编译 WebRTC 的构建工具，获取 depot_tools 前，请先开启 VPN。

下载 [depot_tools](https://storage.googleapis.com/chrome-infra/depot_tools.zip) 解压到某个目录，然后配置系统环境

```shell
echo "export PATH=$PWD/depot_tools:$PATH" > $HOME/.bashrc
source $HOME/.bashrc
```

## 获取源码

获取源代码不能使用官网的源代码，必须到 [webrtc-sdk](https://github.com/webrtc-sdk/webrtc) 中获取专门为 [flutter-webrtc](https://github.com/flutter-webrtc/flutter-webrtc) 修改过的源码。
获取源码步骤如下所示：

```
mkdir webrtc
cd webrtc
touch .gclient
echo "solutions = [
  {
    'name': 'src',
    'url': 'https://github.com/webrtc-sdk/webrtc.git',
    'deps_file': 'DEPS',
    'managed': False,
    'custom_deps': {},
  },
]
target_os = ['android']" > .gclient
gclient sync --nohooks --no-history
```

## 源码编译

+ 安装编译依赖：

	```shell
	gclient runhooks
	./src/build/install-build-deps.sh
	./src/build/install-build-deps-android.sh
	```
+ 编译 aar 文件：
	
	```shell
	vpython3 ./tools_webrtc/android/build_aar.py --build-dir webrtc_android --output ./webrtc_android/libwebrtc.aar --arch armeabi-v7a arm64-v8a x86_64 x86 --extra-gn-args 'is_java_debug=false rtc_include_tests=false rtc_use_h264=false is_component_build=false use_rtti=true rtc_build_examples=false treat_warnings_as_errors=false'
	```
## 异常问题

拉取代码、安装依赖或者编译的时候可能出现一些奇怪的问题，此处将对遇到的问题进行分析以及提出相应的解决方案

+ 执行`./src/build/install-build-deps.sh`, 系统版本不适配(即没有按照步骤中的系统版本进行安装)
	+ 原因：可能你使用的系统版本过高，过低那就没法了，至少要比上诉第一条中系统最低版本高
	+ 解决方案：通过 `lsb_release --codename --short` 查询系统 **short name**，加入到`./src/build/install-build-deps.sh`文件下的`supported_codenames` 变量中即可
+ 编译代码时可能出现缺少 LASTCHANGE 和 LASTCHANGE.committime 文件
	+ 解决方案：直接前往内网 svn 获取 [LASTCHANGE 和 LASTCHANGE.committime](svn://10.10.0.1/svnrepos/%E8%81%94%E9%80%9A/WebRTC/Dependency/M92/Win)即可
+ 编译时出现 `Mission sysroot` 错误
   + 原因：已经很明显，少 sysroot，并且提示里面要求 **run: build/linux/sysroot_scripts/install-sysroot.py --arch=amd64**
   + 解决方案：运行这个脚本即可，注意打开全局代理。如果还不行就手动下载，解压后放到`/usr/local/sysroot` 目录下，配置下 `/etc/profile` 指向 `sysroot` 这个文件夹
   ```shell
    mkdir -p sysroot
	tar -xvf debian_bullseye_amd64_sysroot.tar.xz -C sysroot
   ```
+ 编译时缺少 **llvm-build** 文件夹
	+ 原因：不知道什么原因就是无法同步下来
	+ 解决方案：在国外服务器上同步下来后，把 `llvm-build` 文件拷贝下来即可。也可以用点击 [llvm-build]()下载
