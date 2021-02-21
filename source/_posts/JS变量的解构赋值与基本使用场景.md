---
title: js变量的解构赋值与基本使用场景
date: 2020-12-21 20:30:15
tags: 解构赋值
category: ES6
---

主要讲解JS变量的解构赋值的语法,与其在开发过程中的实际使用场景。

## js解构赋值

### 数组解构赋值

数组的解构遵循**只要等号两边的模式相同，左边的变量就会被赋予对应的值**。在使用的过程中可以使用默认值，默认值可以引用解构赋值的其他变量，但该变量必须已经声明

注意：数组的解构赋值，如果右边不是数组（不是可遍历的结构）会报错

+ 正常的使用情况

```javascript
//完整的使用
	//结果：a=1,b=2,c=3
	var [a, b, c] = [1, 2, 3];
	//结果：foo=1,bar=2,baz=3
	let [foo, [[bar], baz]] = [1, [[2], 3]];

//左边少，右边正常
	//结果：c=3
	var [, , c] = [1, 2, 3]
    //结果：a=1
    var [a] = [1, 2, 3]
    //剩下的情况，聪明的你肯定已经明白
//右边少，左边正常
	//结果：a=1,b=undefined,c=undefined
    var [a, b, c] = [1]
    //结果：a=undefined,b=undefined,c3
    var [a, b, c] = [, , 3]
    //剩下的情况，聪明的你肯定已经明白
```

+ 默认值

```javascript
//结果：foo=true
let [foo = truel = [];
let [foo = truel = [undefined];
let [foo = truel = [null];
     
//必须声明，否则无法使用
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```

+ 特殊的使用

```javascript
function f() {
  console.log('aaa');
}
//结果：x=1
let [x = f()] = [1];
```



