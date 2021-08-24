---
title: CSS_Module
date: 2020-11-10 10:48:06
categories: CSS
tags:
- CSS Module
---

# CSS Module

- react编写过程中我们会发现，css文件都是全局作用的，这样就会产生样式混叠名称难以设置的问题
- CSS Module帮我们解决了这一问题

<!--more-->

## 1.局部作用域

- 产生局部作用域的唯一方法就是，使用一个独一无二的`className`，这也就是CSS Module的做法。

>这里我们先使用`create-react-app` 这个官方脚手架（2.0以后完全兼容了css module）

```
 [name].module.css										//命名规范
 import xxx from 'xxx.module.css'			//引用方法
 <div className={xxx.styleName}>			//用法
```

- ##### `Button.module.css`

```css
.error {
  background-color: red;
}
```

- ##### `Button.js`

```
import React, { Component } from 'react';
import styles from './Button.module.css'; // Import css modules
class Button extends Component {
  render() {
    return <button className={styles.error}>Error Button</button>;
  }
}
```

- ##### Result

```
//[filename]\_[classname]\_\_[hash]

<button class="Button_error_ax7yz">Error Button</button>
```

## 2.全局作用域

- CSS Modules 允许使用`:global(.className)`的语法，声明一个全局规则。凡是这样声明的`class`，<u>都不会被编译成哈希字符串</u>。

```
 xxx.css   												//命名规范
 import ‘xxx.css’									//引用方法
 <div className='styleName'>			//用法
```

- ##### `App.css`

```css
:global(.title) {
  color: green;
}
```
- ##### `App.js`

```
import React, { Component } from 'react';
import from './App.css'; // Import css modules
class App extends Component {
  render() {
    return (
      <h1 className="title">
        Hello World
      </h1>
    );
  }
}
```

- ##### Result

```
//不进行哈希编码

<h1 data-reactroot="" class="title">Hello World</h1>
```

#### 补充点

- CSS Modules 还提供一种显式的局部作用域语法`:local(.className)`，等同于`.className`

```css
//等效写法

:local(.title) {
  color: red;
}

:global(.title) {
  color: green;
}
```

## 3.定制哈希类名

```
create-react-app:默认的哈希算法是 [path][name]__[local]--[hash:base64:5]
```

- webpack中`css-loader`默认的哈希算法是`[hash:base64]`，这会将`.title`编译成`._3zyde4l1yATCOkgn-DBWEL`这样的字符串。
- `webpack.config.js` 里面可以定制哈希字符串的格式。

## 4.Class组合

- 一个选择器可以继承另一个选择器的规则，这称为"组合"

- #### composes

```css
.className {
  background-color: blue;
}

.title {
  composes: className;
  color: red;
}

//蓝底红字
```

```
//被编译成
._2DHwuiHWMnKTOYG45T0x34 {
  color: red;
}

._10B-buq6_BEOTOl9urIjf8 {
  background-color: blue;
}

<h1 class="_2DHwuiHWMnKTOYG45T0x34 _10B-buq6_BEOTOl9urIjf8">
```

## 5.输入其他模块

- 选择器也可以继承其他CSS文件中的规则

**another.css**

```css
.className {
  background-color: blue;
}
```

**App.css**可以继承`another.css`里面的规则

```css
.title {
  composes: className from './another.css';
  color: red;
}
```

未完待续。。。。




















