---
title: C中fork函数问题记录-fork输出结果不一致
date: 2021-02-21 20:46:40
cover: /images/c.jpg
tags: fork
category: c
---

在测试fork特性的时候，用命令行运行与ide中出现完全不同的效果。单独测试fork函数的时候发现，fork成功，但是它会从main函数开始运行。特此记录！

## 环境

+ 华为云centos7
+ 编程ide:vscode+remote
+ 编译环境：gcc version 4.8.5 20150623 (Red Hat 4.8.5-44)

## 代码与结果

```c
#include <unistd.h>  
#include <stdio.h>  
int main(void)  
{  
   int i=0;  
   printf("i son/pa ppid pid  fpid\n");  
   /id指当前进程的父进程pid  
   //pid指当前进程的pid,  
   //fpid指fork返回给当前进程的值  
   for(i=0;i<2;i++){  
       pid_t fpid=fork();  
       if(fpid==0)  
           printf("%d child  %4d %4d %4d\n",i,getppid(),getpid(),fpid);  
       else  
           printf("%d parent %4d %4d %4d\n",i,getppid(),getpid(),fpid);  
   }  
   return 0;  
}
```

vscode结果：

```c
i son/pa ppid pid  fpid
0 parent 26418 26424 26425
1 parent 26418 26424 26426
i son/pa ppid pid  fpid
0 child     1 26425    0
1 parent    1 26425 26427
i son/pa ppid pid  fpid
0 parent 26418 26424 26425
1 child     1 26426    0
i son/pa ppid pid  fpid
0 child     1 26425    0
1 child     1 26427    0
```

命令行输出结果：

```c
i son/pa ppid pid  fpid
0 parent 26622 26671 26672
1 parent 26622 26671 26673
0 child     1 26672    0
1 parent    1 26672 26674
1 child     1 26673    0
1 child     1 26674    0
```



## 总结

目前怀疑是否是vscode的remote造成此原因。结果尚未可知，有待后续研究。