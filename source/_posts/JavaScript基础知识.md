---
title: JavaScript基础知识
date: 2020-10-11 23:25:23
categories: JavaScript
tags:
- js基础
---

> 1. ##### 代码特点
>
> 2. ##### 变量
>
> 3. ##### 数据类型
>
> 4. ##### 交互手段（alert，prompt，confirm）
>
> 5. ##### 类型转换
>
> 6. ##### 基础数学运算符
>
> 7. ##### 值的比较
>
> 8. ##### 逻辑运算符
>
> 9. ##### 条件分支（if 和 '?')
>
> 10. ##### 循环（while 和 for）
>
> 11. ##### "switch"语句
>
> 12. ##### 函数，函数表达式
>
> 13. ##### 箭头函数

<!--more-->

# 1.代码特点

## "script"标签

- `<script>`标签可以将js程序，插入html文档任何位置

### js程序三种引用方法

1. 行内样式（使用很少）

   ```js
   <input type="button" value="点我试试" onclick="alert('Hello World')"/>
   ```

2. `<script>`标签内嵌式

   ```js
   <script>
   	alert('hello world');
   </script>
   ```

3. `<script>`标签引入外部文件

   ```js
   <script src="my.js"></script>
   ```

   - 注意：一个`<script>`标签只能作为两种方式中的一种使用

### 现代的标记（makeup）

- 现在的`<script>`标签很少再使用特性（attribute），但可以在一些老的代码中找到他们

  `type` 特性 `<script type="text/javascript">`,现在用于Javascript模块

  `language` 特性 `<script language=...>`,现在已经默认js语言，无意义啦

## 现代模式"use strict"

- JavaScript 除了提供正常模式(sloppy)外，还提供了严格模式（strict mode）。ES5 的严格模式是采用具有限制性 JavaScript变体的一种方式，即在严格的条件下运行 JS 代码。

  > 严格模式对正常的 JavaScript 语义做了一些更改： 
  >
  > 1.严格模式通过**抛出错误**来消除了一些原有**静默错误**
  >
  > 2.消除代码运行的一些不安全之处，保证代码运行的安全。
  >
  > 3.提高编译器效率，**增加运行速度**。
  >
  > 4.**禁用了**在 ECMAScript 的未来版本中可能会定义的一些语法，为未来新版本的 Javascript 做好铺垫。比如一些保留字如：class,enum,export, extends, import, super 不能做变量名

- 使用方法：推荐加在脚本前

  ```js
  //更像是一个字符串，放在脚本文件的顶部，整个脚本文件就将以”现代模式“工作
  <script>
      "use strict";
  	//严格模式激活！！一旦激活无法回溯
  	alert('hello world');
  </script>
  ```

- 严格模式下基本特点

  ```js
  // 对于未声明的变量不允许使用
  'use strict';
  num = 10;
  console.log(num)	//not defined!
  
  //严格模式不允许删除变量
  var num1 = 1;
  delete num1;		//unqualified！
  
  //全局作用域中函数(或构造函数)中的this是 undefined
  function fn() {
      coonsole.log(this);
  }
  fn();
  
  //严格模式下，定时器 this 还是指向window
  setTimeout(function() {
      console.log(this);	
  }, 2000);
  
  //严格模式下，函数参数不允许有同名的形参
  function fn(a, a) {
      console.log(a + a);
  };
  fn(1, 2);			//4   相当于a=1, a=2 最终a=2
  
  //非函数代码块，不允许定义函数,比如if，for中（函数中套函数可以）
  if (true) {
      function () {}		//!!语法错误
  }
  ```

- 现代js支持`class` `modules`---高级语言结构，会自启动严格模式，无需再加

# 2.变量

- 我们可以使用`var` `let` `const`来声明变量存储数据
- 变量命名：`$` `_`字母，数字，  驼峰命名法

## let

- 现代变量声明方式

```js
let message = "Hello";
```

## var

- 老旧变量声明方式

```js
var message = "Hello";
```

## const

- 常量声明，类似let

1. 大写形式常量：硬编码值的别名

   ```js
   //比如:web16进制格式的颜色常量声明
   const COLOR_RED = "#F00";
   const COLOR_BLUE = "#00F";
   
   //...当我们需要一个颜色时
   let color = COLOR_BLUE;
   alert(color);	// #00F
   ```

2. 小写形式常量：一般不变的值

   ```js
   const pageDownloadTime = /*网页所需加载时间*/；
   //页面加载前是未知的，加载之后就是固定的值
   ```

## var和let区别

1. var没有块级作用域

   > - 即：不是函数作用域就是全局作用域

2. var允许重新声明

   > - let在同一作用域下，同时声明同一个变量两次会报错
   >
   > - var 允许重复声明（第二个声明初始化无效，作用域下确认了前面的声明

3. var声明的变量，有变量提升

   > - let声明的变量没有变量提升，变量声明必须在变量赋值前
   > - var声明可以在赋值的后面（声明有提升，赋值没有）
   >
   > ```js
   > //看一个有意思的案例
   > function sayHi () {
   >     phrase = "Hello";
   >     
   >     //这部分代码理论上是不可能执行的
   >     //但在进入这个函数作用域时，首先就处理了var语句！！！
   >     if (false) {
   >         var phrase;
   >     }
   >     
   >     alert(phrase);
   > }
   > sayHi();
   > ```

# 3.数据类型

- js中有8种基本数据类型（7种原始类型，1种引用类型）---动态类型

## Number类型

- 整数和浮点数

- 常规数字和特殊数值（Infinity, -Infinity, 和NaN)

## BigInt类型

- number无法表示大于`2e53-1`或小于`-（2e53-1）`的整数
- 当我们需要加密或者微妙精度的时间戳时，BigInt类型表示任意长度

```js
//尾部一个n表示是一个BigInt类型
const bigInt = 1234567890123456789012345678901234567890n;
```

## String类型

- 字符串必须括在引号里，一共有三种引号

  ```js
  let str = "Hello";
  let str2 = 'single quotes are ok too';
  let phrase = `can embed another ${str}`;
  ```

  > 功能扩展引号：允许通过将变量和表达式包装在`${}` 中

## Boolean 类型

- `true` `false`两个值

## "null"值

- 特殊的null不属于上述任何一种类型

- 它构成一个独立的类型。只包含`null` 值

  ```js
  let age = null;	//表示age是未知的
  
  //和其他语言不同，js的null不是一个对不存在的object的引用”或者 “null 指针”
  //Js中的 null 仅仅是一个代表“无”、“空”或“值未知”的特殊值。
  ```

## "undefined"值

- 也是一个独立的类型，表示 `未被赋值`

  ```js
  let age;
  alert(age);		//弹出“undefined”
  ```

> 注：通常，使用 `null` 将一个“空”或者“未知”的值写入变量中，而 `undefined` 则保留作为未进行初始化的事物的默认初始值

## object类型和symbol类型

- `object` 则用于储存数据集合和更复杂的实体

- `symbol` 类型用于创建对象的唯一标识符

## typeof运算符

- 支持两种语法形式 (有无括号都一样)
  1. 作为运算符：`typeof x`
  2. 函数形式：`typeof(x)`

```js
typeof undefined 		// "undefined"
typeof 0 				// "number"
typeof 10n 				// "bigint"
typeof true 			// "boolean"
typeof "foo" 			// "string"
typeof Symbol("id") 	// "symbol"
typeof Math 			// "object"
typeof null 			// "object"		语法上的一种错误
typeof alert		    // "function"	虽然没有function数据类型，但这样很方便！
```

# 4.三种交互方式

- 这里讲三种模态的，他们中止脚本的执行，不允许用户和其他页面交互，直到窗口被解除

## alert

- 弹出带信息的**模态框**，（“modal” 意味着用户不能与页面的其他部分）

  ```js
  alert("Hello");
  ```

## prompt

- 弹出一个带有文本消息的模态窗口，有input框和确定/取消按钮

  ```js
  result = prompt(title, [default]);
  //title: 显示文本提示信息
  //default: 指定input框的默认值		
  ```

- 访问者输入一些内容，确定就会返回一个字符串类型的值；取消获得null返回值

  ```js
  let age = prompt('How old are you?', 100);
  alert(`You are ${age} years old!`); 	// You are 100 years old!
  ```

- 没有输入内容点击确定，会返回空字符串

  > IE会提供默认值 undefined 
  >
  > ```js
  > //推荐
  > let test = prompt('test', '');	//加一个空字符串！！
  > ```

## conform

- conform函数，会显示一个带有question以及确认取消按钮的模态框

- 点击确认返回`true` ，点击取消返回`false` 

  ```js
  let isBoss = confirm("Are you the boss?");
  alert( isBoss ); 	// 如果“确定”按钮被按下，则显示 true
  ```

  

  



















































































































































