---
title: freeswitch中开发列出所有模块命令
date: 2021-05-25 20:48:10
tag: command
category: freeswitch
---

本文章主要是对本人在fs中开发一个命令的时候的一些随想。主要记录一些遇到的问题与一些简答的想法，不具有体系性，望与共勉！

## vs中编译代码

开发一个模块命令后，需要重新`build`。

+ 自言自语：点击调试好像没法重新加载代码，重新`build`很慢，是否有其它方法？只是一个模块，应该是编译这一个模块就可以塞？

干，vs2017中`生成`里面有重新生成修改的模块。-_-||

## 用到的函数

+ `switch_xml_open_cfg(const char *file_path, switch_xml_t *node, switch_event_t *params)`
  + 这个函数内部调用`switch_xml_locate`，看其中内容可以发现要求传入文件的名字即可。先就讲到这里，细节后面再说
  + file_path:需要指定文件的名字，不包括路径。例如: "modules.conf"
  + node: 声明一个switch_xml_t即可
  + params: 直接传输NULL
+ `switch_xml_child(switch_xml_t xml, const char *name)`
  + 这个函数主要用于获取xml文件的节点内容，xml懂得都懂
  + xml: 这里传入刚才在上诉函数中获取node
  + name:注意xml的层级关系，第一次的话只能获取最顶部的，然后再获取下面的child,
+ `switch_xml_attr_soft(switch_xml_t xml, const char *attr)`
  + 此函数用于获取在节点上的属性值。例如：<load module="mod_syslog"/>，其中的module中的值
  + xml: 获取到上面例如中的节点
  + name: 输入`"module"`然后就可以得到值
+ `stream->write_function(switch_stream_handle_t *handle, const char *fmt, ...)`
  + 此函数用于在console中输出内容
  + handle: 将在command中获取到的stream扔进去即可
  + fmt: 输出内容
  + ...：如果在fmt中使用占位符，可以在此处传入值
+ `sprintf、malloc、strlen`
  + 系统函数，就不讲了