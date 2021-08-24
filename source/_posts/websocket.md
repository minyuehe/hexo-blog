---
title: websocket
date: 2021-05-28 15:07:39
categories: HTML5
tags:
- socket
---

## 前言

没错就是计网实验，不想看c的socket编程，摸鱼到HTML5的websoket，一探究竟

通信协议已经有了HTTP协议，为什么还要有websocket新协议？

HTTP痛点：通信只能客户端发起！！

基于单点请求的特点：注定了如果服务器有连续的状态变化，客户端想获知很麻烦！

----人们想到`轮询`

但是，轮询效率很低，浪费资源（HTTP连接keep-alive）

于是引入今天的主角：websocket

<!--more-->

## websocket简介

最大的特点就是：

服务端可以主动向客户端推送消息，客户端也可以主动向服务器发送消息

真正的双向平等对话（属于[服务器推送技术](https://cloud.tencent.com/developer/article/1407649)的一种）

![img](websocket/bg2017051502.png)

**特点**

1. 建立在tcp协议上，服务端实现比较容易

2. 和HTTP有很好的兼容性，默认端口80和443，握手采用HTTP协议，可以通过各种HTTP代理服务器

3. 数据格式轻量，性能开销小，通信高效

4. 可发送文本，二进制数据

5. 没有同源限制，客户端可以和任意服务器通信

6. 协议标识符 `ws` (加密`wss`)

   服务器网址示例`ws://example.com:80/path`

![img](websocket/bg2017051503.jpg) 

## 客户端

### 示例

[这里](https://jsbin.com/muqamiqimu/edit?js,console)

```js
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};      
```

### API

#### 1.websocket构造函数

新建[websocket实例](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

```js
const ws = new WebSocket('ws://localhost:8080');
```

客户端就会和参数网址的服务器进行连接

#### 2.webSocket.readyState

`readyState` 属性返回实例对象的**当前状态**

- CONNECTION：值为0，表示正在连接
- OPEN：值为1，表示连接成功，可以通信了。
- CLOSING：值为2，表示连接正在关闭。
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

```js
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```

#### 3.webSocket.onopen

`onopen` 属性，用于指定**连接成功后**的回调函数

```javascript
ws.onopen = function () {
  ws.send('Hello Server!');
}
```

多个回调函数：采用事件监听语法

```javascript
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```

#### 4.webSocket.onclose

`onclose`属性，用于指定**连接关闭后**的回调函数。

```javascript
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};
```

多回调同理

#### 5.webSocket.onmessage

`onmessage`属性，用于指定**收到服务器数据后**的回调函数。

```javascript
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};
```

**注意：**

服务器数据可能是文本，也可能二进制（blob对象或Arraybuffer对象）

```javascript
ws.onmessage = function(event){
  if(typeof event.data === String) {
    console.log("Received data string");
  }

  if(event.data instanceof ArrayBuffer){
    const buffer = event.data;
    console.log("Received arraybuffer");
  }
}
```

除了动态判断收到的数据类型，也可以使用`binaryType`属性，显式指定收到的二进制数据类型。

```javascript
// 收到的是 blob 数据
ws.binaryType = "blob";
ws.onmessage = function(e) {
  console.log(e.data.size);
};

// 收到的是 ArrayBuffer 数据
ws.binaryType = "arraybuffer";
ws.onmessage = function(e) {
  console.log(e.data.byteLength);
};
```

#### 6.webSocket.send()

`send()`方法用于向服务器发送数据。

发送文本的例子。

```js
ws.send('your message');
```

发送 Blob 对象的例子。

```js
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```

发送 ArrayBuffer 对象的例子。

```js
// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```

#### 7.webSocket.bufferedAmount

`bufferedAmount`属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

```javascript
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

#### 8.webSocket.onerror

`onerror`属性，用于指定报错时的回调函数

```javascript
socket.onerror = function(event) {
  // handle error event
};
```

## 服务端示例

node实现

- [WebSocket-Node](https://github.com/theturtle32/WebSocket-Node)
- [µWebSockets](https://github.com/uWebSockets/uWebSockets)





## WebSocketd

推荐WebSocket服务器

[Websocketd](http://websocketd.com/)

**特点：**不限语言标准输入（stdin）就是 WebSocket 的输入，标准输出（stdout）就是 WebSocket 的输出













**参考链接**

http://www.ruanyifeng.com/blog/2017/05/websocket.html

















