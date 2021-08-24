---
title: CSS面试
date: 2021-04-14 16:07:38
categories: 面试
tags:
- CSS tips
---

#### 1.如何实现0.5px边框

- 利用缩放`transform: scale(0.5, 0.5)`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>边框0.5px实现方法</title>
    <style type="text/css">
      .container {
        width: 500px;
        margin: 0px auto;
      }
      .border-scale {
        position: relative;
        padding: 10px;
      }

      .border-scale:after {
        content: "";
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        height: 0;
        /* background-color: #f00; */
        /* height设为1px，用background-color效果是一样的*/
        border-top: 1px solid #f00;
        transform: scaleY(0.5);
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h3>方案一：使用缩放</h3>
      <div class="border-scale">原理： 在Y轴方向上，压缩一半。</div>
    </div>
  </body>
</html>
```

>思路二：`background-image: linear-gradient( , , )`

- 拓展：四条边框

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>边框0.5px实现方法</title>
    <style type="text/css">
      .container {
        width: 500px;
        margin: 0px auto;
      }
      .border-all {
        position: relative;
        padding: 10px;
      }

      .border-all:after {
        content: "";
        position: absolute;
        left: 0;
        top: 0;
        z-index: -1;
        width: 200%;
        height: 200%;
        border: 1px solid #f00;
        //原点设定！！！
        transform-origin: 0 0;
        transform: scale(0.5, 0.5);
        border-radius: 10px;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h3>拓展：四周全是0.5px的线条的话</h3>
      <div class="border-all">
        这是实现一个盒子四周0.5px的做法，z-index，可以根据不同需求来调整使用，如果可以的话，不使用也是可以的。
      </div>
    </div>
  </body>
</html>
```

