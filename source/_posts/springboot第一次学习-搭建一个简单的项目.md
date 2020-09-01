---
title: springboot第一次学习-搭建一个简单的项目
date: 2020-09-01 21:53:50
cover: /images/springboot.jpg
tags: maven
category: springboot

---

阅读官方文档，官方文档中对于springboot的简单项目搭建的基本解释

## SpringBoot中spring-parent的作用

+ 根据文档的翻译可以简单概括为spring-boot-starter-parent不提供依赖关系。它主要提供maven的一些默认值，一个依赖管理器用于你添加依赖的时候不需要在maven中加version。[原文链接](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-dependencies)

+ <span style="color:red;">注意：其中不提供依赖的意思是，它不提供如start-web这一类的web启动依赖。即spring-boot-starter-parent不是启动程序，它只是提供了管理工具与预设置值</span>

  



