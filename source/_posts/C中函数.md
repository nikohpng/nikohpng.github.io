---
title: C中的函数
date: 2021-03-23 21:06:16
cover: /images/function.jpg
tags: 函数
category: c
---
本文主要着重函数的概念，以及在声明、调用上面进行讲解。其中会额外讲解变量的作用范围，其主要也是着眼于函数内。

## 函数的概念
+ 数学：数学中的函数是指一个数集按照一个指定的法则映射到另一个数集中，其中的法则被称为函数关系式或函数
+ 程序：是指传入输入数据按照一定的规则运算后输出一个有意义的数据，其中我们将一定的运算规则就称为函数。
> tips:其中关于函数的理解是本人个人总结，如有错误，还请指导。

## 函数的声明
在C语言中，任何函数的使用必须首先进行声明，否则进行调用。其中max对其进行了声明，声明形式为`return_type function_name( parameter list );`
```c
#include<stdio.h>
//声明的形式
int main(){
    int a=1,b=2;
    int max(int a,int b);
    int maxVar = max(a,b);
    printf("%d",maxVar);
    return 1;
}
int max(int a,int b){
    if(a>b){
        return a;
    }
    return b;
}

```
其中`paramter list`在声明与定义处叫形参，在调用处叫实参。其中形参与实参具有不同的地址空间，即如上代码修改max函数中的a变量不会影响main函数中的a变量

声明函数三种方式：
+ 直接在main之前或在main中调用函数之前声明。
+ 将所有需要声明的函数单独封装为一个`.h`头文件，使用`#include ""`引入。**注意**：此处必须使用双引号，其表示引入本地的头文件
+ 使用系统本地的声明内容。使用`#include<>`引入本地的头文件

## 函数的调用
函数的调用形式`function_name(paramter list);`进行调用，此处的的`paramter list`名为实参。
> 注意：其中实际参数与形式参数的对应关系是一一对应，参数类型必须一致，且从左往右依次解析

## 函数的返回值
函数的返回值在函数中采用`return`进行返回，如果是表达式的话注意采用`()`进行包裹
> 注意：函数返回的值必须与声明的时候`return type`保持一致，如果没有返回值需要使用`void`进行声明，函数中也不写`return`

## 函数的特殊赋值
在数组的过程中，与普通的形式参数有点不同。数组在实参使用的时候只需要将数组名放进去就可以，且形参与实参具有相同的地址空间
+ 原理：数组名是一个指向数组第一变量的地址的指针，因而传递到函数中，也是指向的同一个地址。
```c
#include<stdio.h>

int count(int arr[3]);

int main(){
    int arr[] = {1,2,3};
    int i = 0;
    
    int countAll = count(arr);
    printf("count: %d\n",countAll);

    for(i = 0 ; i < 3 ; i++){
        printf("i=%d\n",arr[i]);
    }
    return 0;
}

int count(int arr[3]){
    int i = 0,count = 0;
    for( i = 0 ; i < 3 ; i++){
        count += arr[i];
        arr[i] = 0;
    }
    return count;
}
```

## 局部变量
### 局部变量
+ 作用域：程序块与函数中。程序块如for if这一类，它的作用范围为{}内。函数中的形参是局部变量，作用范围直到return
+ 生命周期：程序块或方法块结束，局部变量结束
+ 内存地址： 局部变量都被定义在栈中
+ 注意：如果变量名冲突，以最近定义或赋值的变量名为准
### 全局变量
+ 作用域：从定义处开始，到程序结束
+ 生命周期： 随着程序的生和死亡
+ 内存位置：？

## auto、static、extern、register
### auto关键字
所有变量默认是此种方式属于局部变量。存储在动态存储区栈中。示例代码如下：
```c
#include<stdio.h>

int main(){
    auto int a = 1;
    printf("a=%d",a);
    return 1;
}
```
### static局部变量
将局部变量用`static`字段进行修饰，其具有两个重要特性：+ 不释放：在多次的函数运行中，static只初始化一次，整个程序的运行过程中不释放
+ 变量私有化： 定义的局部变量、全局变量或函数不能够被非本函数或非本文件将使用
示例代码如下：
```c
#include<stdio.h>

int count(int arr[3]);

int main(){
    int arr[] = {1,2,3};
    
    printf("count: %d\n",count(arr));
    printf("count: %d\n",count(arr));

    return 0;
}

int count(int arr[3]){
    int i = 0;
    static int count = 0;
    for( i = 0 ; i < 3 ; i++){
        count += arr[i];
    }
    return count;
}
```

### static全局变量
static全局变量只能在定义的文件中使用，正如上文所说的变量的私有化。（简单理解：加了static也没法加extern，就只能在本文件运行了）

### static函数名
static会让函数名默认只能在本文件中使用，无法通过使用extern让其分享其它文件。（简单理解：加了static也没法加extern，就只能在本文件运行了）

### register变量
C语言允许将局部变量存储寄存区中，全局变量不允许。
演示这个可以写一个斐波拉函数，这里就不做演示了。

### extern变量
此关键字用于扩展变量的作用域，被声明的变量，可以用于多个文件。示例代码如下
+ 单文件
```c
#include<stdio.h>
int main(){
    extern a;
    printf("a: %d\n",a);
    return 0;
}

int a = 10;
```
+ 多文件
```c
//TestExternalParent.c
#include<stdio.h>

int A = 1;

int main(int argc,char *argv[]){
    void print(int a);

    print(2);
    return 1;
}

//TestExternalChild.c
extern int A;

void print(int a){
    A += a;
    printf("%d",A);
}
```

### extern函数名
可以在不同文件中。定义时，不写则默认为extern
