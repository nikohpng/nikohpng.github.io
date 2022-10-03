---
title: shell的基本认识
date: 2020-10-14 22:10:25
cover: /images/shell.jpg 
tags: bash
category: Shell

---
对shell的种类进行简单的介绍，主要还是集中在shell的变量、变量比较、条件控制与函数的介绍。其中会涉及到一些判断命令的使用，其中将会进行特殊的介绍！

## shell的种类

Linux中自带的shell,可以检查`/etc/shells`文件内容。每个发行版本可能不同

+ /bin/sh：已经被/bin/bash所替代
+ /bin/bash：就是Linux默认的shell
+ /bin/ksh：由AT&T Bell发展来的，兼容bash
+ /bin/tcsh：整合C Shell，提供更多功能
+ /bin/csh：已经被tcsh替代
+ /bin/zsh：基于ksh法阵来，功能更强大的shell

>tips:后续有关变量、控制语句、逻辑判断采用的全部都是bash

## bash的变量

bash的变量有`文本`（字符串）与`数字`类型，其中数字类型最大值为`2^32`。在变量中可以定义`数组`和`map`类型。变量的使用采用`${变量名}`或`$变量名`

### 文本类型定义

文本类型可使用双引号`""`或单引号`''`，其中区别是双引号可以保留变量原本的性质，单引号中的内容只是文本。如下代码的`var1`变量

```shell
    # 注意：变量=号两端不能有空格
    var="变量内容"
    echo ${var}

    # 双引号保留变量性质
    var1="${var}"
    echo ${var1}

    # unset 取消变量设置
    unset var1
    echo ${var1}
```

如果从系统中获取内容，并将其赋值给变量。则有以下两种方式

```shell
    # 从系统中获取系统的版本信息，注意命令与括号之间的空格
    version=$( uname -r )

    # ``反引号中的命令会被先执行，执行的结果作为外部的输入信息
    ls -l `locate crontab`
```

> tips: 文本的定义建议采用双引号，有利于变量的使用。
从系统中获取内容，建议使用$()，因为反引号可能不容易失败

### 其它类型变量定义

这里其它类型变量定义包括`数组`、`整数`和`map`类型的定义,涉及的关键字为`declare`或`typeset`

```shell

    # 定义数组类型
    declare -a var
    typeset -a var
    
    # 定义数组变量后，通过index将值赋值进去
    var[1]="测试内容1"
    var[2]="测试内容2"
    echo "${var[1]},${var[2]}"

    # 定义整数类型
    declare -i sum=100+300+50
    echo "${sum}"


    # 定义map类型
    declare -A map
    map["test"]="hello"
    echo "${map['test']}"

```
> tips: declare -r会将变量设置为`只读`,变量中的内容不可以更改


## bash变量的范围

bash中普通定义的变量的范围为整个shell文件，如果想要在不同shell中引用则要使用`export`将其变为`全局变量`

```shell

    # shell1.sh 调用shell2.sh。运行shell1的时候用sh shell1.sh
    set name="hello world"
    export name
    sh shell2.sh

    # shell2.sh 输出变量name的值
    echo "${name}"
```

上诉情况shell1.sh是父进程，shell2.sh是子进程。上诉的变量的作用范围就是父进程与子进程。

## 变量比较

### `test`命令进行变量比较

变量的比较采用命令`test`进行比较，当然在后续的使用过程可以不用加命令名。具体的使用方法在后续会详细讲解，此处只说明其每个参数的比较的含义

| 测试标志           | 代表含义                                              |
| ------------------ | ----------------------------------------------------- |
| 关于“文件类型”判断 |                                                       |
| -e                 | 该文件名是否存在                                      |
| -f                 | 该文件名是否存在且为文件（常用）                      |
| -d                 | 该文件名是否存在且为目录（常用）                      |
| -b                 | 该文件名是否存在且为一个block device设备              |
| -c                 | 该文件名是否存在且为一个character device设备          |
| -S                 | 该文件名是否存在且为一个Socket文件                    |
| -p                 | 该文件名是否存在且为一个FIFO文件                      |
| -L                 | 该文件名是否存在且为一个连接文件                      |
| 关于文件的权限检测 |                                                       |
| -r                 | 检测该文件名是否存在且具有“可读”权限                  |
| -w                 | 检测该文件名是否存在且具有“可写”权限                  |
| -x                 | 检测该文件名是否存在且具有“可执行”权限                |
| -u                 | 检测该文件名是否存在且具有“SUID”的属性                |
| -g                 | 检测该文件名是否存在且具有“SGID”的属性                |
| -k                 | 检测该文件名是否存在且具有“Sticky bit”的属性          |
| -s                 | 检测该文件名是否存在且具为“非空白文件”                |
| 两个文件之间的比较 |                                                       |
| -nt                | (newer than)判断file1是否比file2新                    |
| -ot                | (older than)判断file1是否比file2旧                    |
| -ef                | 判断file1与file2是否为同一文件                        |
| 两个整数之间的比较 |                                                       |
| -eq                | 两数值相等(equal)                                     |
| -ne                | 两数值不等(not equal)                                 |
| -gt                | n1大于n2(greater than)                                |
| -lt                | n1小于n2(less than)                                   |
| -ge                | n1大于等于n2(greater than or equal)                   |
| -le                | n1小于等于n2(less than or equal)                      |
| 判断字符串的数据   |                                                       |
| Test -z string     | 判断字符串是否为0，若string为空字符串，则为true       |
| Test -n string     | 判断字符串是否为非0，若字符串为空字符串，则为false    |
| test str1 = str2   | 判断str1是否等于str2，若相等，则回传true              |
| test str1 != str2  | 判断str1是否不等于str2，若相等，则回传false           |
| 多重条件判定       |                                                       |
| -a                 | 两个条件同时成立！例test -r file -a -x file           |
| -o                 | 任何一个条件成立！例test -f file -o -x file           |
| ！                 | 反向状态，如test ! -x file，当file不具有x时，回传true |

### 利用判断符号比较`[]`

上诉的使用过程中必须加上`test`。在shell script中，可以采用判断符号`[]`进行数据判断。如下面例子所述：

```shell
# 比较数字
test 1 -eq 1 && echo "成功"
或
[ 1 -eq 1 ] && echo "成功"

#比较字符串
test -z "" && echo "成功"
或
[ -z "" ] && echo "成功"
```

相信聪明的你已经明白这个判断在条件判断中的作用。`&&`是逻辑中的与，我们将在后续讲解！

> 注意：在使用`[]`符号的时候，其中的空格必须注意。如[空格"$home"空格==空格"$MAIL"空格]；在这里如果在`[]`符号中引用变量，必须加上`""`后再使用变量

### 逻辑判定符（`&&`、`||`）

逻辑判定符号有`&&`与`||`，其中的作用与普通的编程程序相同，分别表示`与`与`或`。但是其在shell中的使用有些特殊，如下所示：

```shell
# ||符号的使用,请复制到script中运行
read -p "请输入y或Y:" yn
[ "${yn}" == "Y" -o "${yn}" == "y" ] && echo "yes"
# 可替换为
[ "${yn}" == "Y" ] || [ "${yn}" == "y" ] && echo "yes"
```

## 条件控制

### 利用if..then

其中`if..then`就是普通程序中的`if`判断，其具体使用如下例

```shell
# 没有else
read -p "请输入y或Y:" yn
if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
	echo "y"
fi

# 有else
if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
	echo "y"
else
	echo "else"
fi

# 多条件判定
if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
	echo "y"
elif [ "$yn" == "N" ] || [ "$yn" == "n" ]; then
	echo "n"
else
	echo "else"
fi
```

### 利用`case..esac`

其中`case..esac`就是普通程序中的`switch`，其具体使用如下例

```shell
read -p "请输入y或Y:" yn
case ${yn} in
	"y" )
		echo "y"
		;;
    "n" )
    	echo "n"
    	;;
    * )
    	echo "else"
    	;;
esac
```

### 循环

#### while do done,until do done

其中`while do done`与普通程序中的`while`关键字一样的逻辑，只有满足条件执行。

其中`until do done`与普通程序中的`do..while`关键字不一样。`do..while` 是先执行语句， 然后满足条件就继续执行；`until do done`是当满足条件就终止循环

```shell
# 当满足条件就执行，直到不满足条件
while [ "$yn" == "Y" ] && [ "$yn" == "y" ]
	read -p "请输入y或Y:" yn
do
done
echo "ok,you are right!"

# 当满足条件就终止循环
until [ "$yn" == "Y" ] || [ "$yn" == "y" ]
do
	read -p "请输入y或Y:" yn
done
```

#### for..do..done(固定条件)

主要是对知道固定循环次数的进行循环，例如数组这一类，如下所示

```shell
declare -a var
var[1]="hello"
var[2]="world"
for v in ${var[*]}
do
	echo "$var"
done
# 或替换为下方
for v in ${var[@]}
do
	echo "$var"
done

#从1-100进行输出
for i in $(seq 1 100)
do
	echo $i
done
```

> tips:注意在遍历数组时，in后面必须使用`${var}`来表示变量，其中`[]`中的`*`与`@`表示数组汇总的所有元素。可以思考下为什么变量要加${}呢？和前面的判定符号有关哦~

循环的时候，确定步长进行输出

```shell
read -p "请输入需要计算总数和的数字：" nu
for (( i=1; i<$nu; i=i+1 ))
do
	s=$(( $s+$i ))
done
echo $s
```

## 函数(function)

`shell script`是由上而下、由左而右，因此`function`一定要设置在最**前面**！

`function`拥有内置变量，变量采用$1,$2..来获取

```shell
function printit(){
	echo "你输入的内容为：$1"
}

printit "hello world"
```



## shell追踪与调试

使用的命令是`sh -[nvx]`进行相应的调试，可以复制上面的shell测试下效果

| 参数名 | 效果描述                                         |
| ------ | ------------------------------------------------ |
| -n     | 不要执行script，仅查询语法问题                   |
| -v     | 在执行script前，先将script的内容输出到屏幕上     |
| -x     | 将使用到的script内容显示到屏幕上（这个很有用哦） |

