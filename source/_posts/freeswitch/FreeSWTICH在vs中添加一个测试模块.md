---
title: FreeSWTICH在vs中添加一个测试模块
date: 2021-06-24 20:39:30
tags: event
category: FreeSWTICH
---

本文主要记录如何在 vs2017 中添加一个模块，并编译运行。然后，测试 event 内容

## 环境

| 环境名 | 版本     |
|----   |----     |
| WIN   | WIN10   |
| FS    | 1.10.0  |
| VS    | vs2017  |

## 添加一个模块

### 手动添加各种参数

1. 在 vs2017 的资源管理中，选择一个文件夹，添加一个 c++ -> dll 项目，添加完成后删除多余的内容。

2. 添加引用，直接在项目处添加引用，选择项目中的引用 `FreeswitchCoreLib`

3. 增加库文件，选择 项目-> 属性 -> c/c++ -> 所有选项 -> 附加包含目录 增加项目解决方案的 lib 即可，目录内容为`$(SolutionDir)\src\include`

4. 修改预处理器。在 fs 项目中，通过 vs 添加的项目默认为导入，我们需要在 项目->属性->c/c++->预编译头->预处理器定义 中添加`MOD_EXPORTS`，将项目标识为导出。源代码如下所示：

   ```c
   // 我们在使用 SWITCH_MODULE_DEFINITION 时会被扩展为带有 SWITCH_MOD_DECLARE_DATA 前缀的结构体
   #if defined(SWITCH_MOD_DECLARE_STATIC)
   #define SWITCH_MOD_DECLARE(type)		type __stdcall
   #define SWITCH_MOD_DECLARE_NONSTD(type)	type __cdecl
   #define SWITCH_MOD_DECLARE_DATA
   #elif defined(MOD_EXPORTS)
   #define SWITCH_MOD_DECLARE(type)		__declspec(dllexport) type __stdcall
   #define SWITCH_MOD_DECLARE_NONSTD(type)	__declspec(dllexport) type __cdecl
   #define SWITCH_MOD_DECLARE_DATA			__declspec(dllexport)
   #else
   #define SWITCH_MOD_DECLARE(type)		__declspec(dllimport) type __stdcall
   #define SWITCH_MOD_DECLARE_NONSTD(type)	__declspec(dllimport) type __cdecl
   #define SWITCH_MOD_DECLARE_DATA			__declspec(dllimport)
   #endif
   ```

5. 修改 项目->属性->c/c++->语言->符合模式 为**否**

6. 在 “项目”（顶部导航栏） 中选择 “清理项目”，然后 “生成项目”即可

### 使用属性管理器

1. 在 vs2017 的资源管理中，选择一个文件夹，添加一个 c++ -> dll 项目，添加完成后删除多余的内容。

   > 注意：添加的代码为 C 语言代码，所以代码文件要以*.c结尾

2. 添加引用，直接在项目处添加引用，选择项目中的引用 `FreeswitchCoreLib`

3. **属性管理器**中找到新创建的项目，在**Debug | Win32**中选择“添加现有属性表”，添加`freeswitch/w32`中的`module_debug.props`

4. 修改 项目->属性->c/c++->语言->符合模式 为**否**
