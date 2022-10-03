---
title: C与FS中内存分配简析
date: 2021-06-03 20:41:07
tags: 内存分配
category: freeswitch
---

本文将会讲解C标准库中内存分配在不同平台的不同实现方式，以及fs对标准的一个工具化封装。主要是对fs的内存分配有一个简单的认识。

## C中的内存分配

C中的内存分配主要有以下三个方法。此处只是对其做简单的介绍，我发现一篇文章写得很好，可以阅读[calloc、malloc、realloc函数的区别及用法！](https://blog.csdn.net/weibo1230123/article/details/81503135)一文

+ `malloc`:其原型为`void *malloc(unsigned int num_bytes)`，其中此处的num_bytes不是Bytes,所以需要将c语言中的基础类型`sizeof`求值

+ `calloc`:其原型为`void *calloc(size_t n, size_t size)`，这个本质还是malloc进行分配，分配完后并进行了初始化。可以看引用中[malloc和calloc的差别](https://www.cnblogs.com/mfrbuaa/p/5383026.html)一文

+ `realloc`:其原型为`void *realloc(void *ptr, size_t new_Size)`，就是对空间进行扩充。扩充过程中如果比较大，那么会重新开辟空间并将原来空间内容拷贝过去，并将以前的空间释放。此处的大以系统为准，在以前的地址申请，如果能够申请那么大就直接返回以前的地址。

  >  注意：ptr不能NULL

## 引用

[malloc和calloc的差别](https://www.cnblogs.com/mfrbuaa/p/5383026.html)

[calloc、malloc、realloc函数的区别及用法！](https://blog.csdn.net/weibo1230123/article/details/81503135)

