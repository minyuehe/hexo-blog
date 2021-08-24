---
title: Promise回调
date: 2020-10-13 15:34:36
categories: JavaScript
tags:
- js异步
- Promise
---

# Promise回调

## 引入promise

- 为什么js要加入回调这一概念？  让我们来看下面的示例
- 异步编程

<!--more-->

下面一个异步行为：

```js
//使用loadScript(src)给脚本加载给定的src
function loadScript(src) {
    let script = document.createElement('script');
    script.src = src;
    document.head.append(script);
}
loadScript('/my/script.js');
//但是我们会发现：
// loadScript 下面的代码
// 不会等到脚本加载完成才执行
```

- 可以看出，我们需要一个回调信息，告诉我们浏览器何时完成了加载，之后我们才能使用新加载的函数和属性

```js
//于是我们将函数加入回调函数参数
//callback函数就实现了，当脚本文件加载完毕立即执行回调函数内容
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));
  document.head.append(script);
}

loadScript('/my/script.js', function(error, script) {
  if (error) {
    	// 处理 error
      handleError(error);
  } else {
    	// 脚本加载成功
  }
});
```

- 于此，我们又发现了新的问题，如果有多个嵌套的引入函数时，我们就要把代码写的像“厄运金字塔”一样，很容易失控！

{% asset_img 厄运金字塔.png 厄运金字塔 %}

- 进一步，我们想通过多个函数分开写法，减少嵌套,但这种方式代码支离破碎，读起来跳来跳去，多个函数命名也是问题，于是我们探索更好的方式，最好的方法之一就是 “promise”

```js
loadScript('1.js', step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('3.js', step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...加载完所有脚本后继续 (*)
  }
};
```

## Promise

- 这里我们做一个类比

  > "生产者代码"做一些事，并且需要一些时间，如：通过网络加载数据的代码，像一位歌手
  >
  > "消费者代码"想在"生产者代码"完成工作第一时间得到其工作成果，像粉丝

- promise就是将两者连接在一起的特殊JS对象

  > 前者：传递给 `new Promise`的函数**executor**，对象创建时自动执行
  >
  > 后者：将接收结果或 error。可以通过使用 `.then`、`.catch` 和 `.finally` 方法为消费函数进行注册。

### 构造器语法

  ```js
  let promise = new Promise(function(resovle, reject) {
      // executor (生产者代码，“歌手”)
  })
  ```

  - 传递给 `new Promise`的函数是**executor**，对象创建时自动执行---就是歌手

    > resolve 和 reject 是Javascript自身提供的的回调
    >
    > - `resolve(value)` — 如果任务成功完成并带有结果 `value`。
    > - `reject(error)` — 如果出现了 error，`error` 即为 error 对象。

  - 由 `new Promise` 构造器返回的 `promise` 对象具有以下内部属性：

    > - `state` — 最初是 `"pending"`，然后在 `resolve` 被调用时变为 `"fulfilled"`，或者在 `reject` 被调用时变为 `"rejected"`。
    > - `result` — 最初是 `undefined`，然后在 `resolve(value)` 被调用时变为 `value`，或者在 `reject(error)` 被调用时变为 `error`。

{% asset_img promise属性.png promise属性 %}

- 看两个实例

  ```js
  // (1) 成功完成
  let promise = new Promise(function(resolve, reject) {
    // 当 promise 被构造完成时，自动执行此函数
  
    // 1 秒后发出工作已经被完成的信号，并带有结果 "done"
    setTimeout(() => resolve("done"), 1000);
  });
  ```

{% asset_img sucess案例.png sucess案例 %}

  ```js
  // (2) error案例
  let promise = new Promise(function(resolve, reject) {
    // 1 秒后发出工作已经被完成的信号，并带有 error
    setTimeout(() => reject(new Error("Whoops!")), 1000);
  });
  ```

{% asset_img error案例.png error案例 %}

- 注意：executor 只能调用一个`resolve` 或一个 `reject`

  ```js
  // 宗旨:一个被 executor 完成的工作只能有一个结果或一个 error。
  let promise = new Promise(function(resolve, reject) {
    resolve("done");
  
    reject(new Error("…")); // 被忽略
    setTimeout(() => resolve("…")); // 被忽略
  });
  ```

### 消费者：then，catch，finally

- `state` **和** `result` **都是内部的**,不能直接访问他们，但我们可以对他们使用`.then`/`.catch`/`.finally` 方法

##### **then**

```js
promise.then (
	function(result) //handle a successful result
    function(error)``//handle an error
);
```

- 第一个参数是一个函数，在`promise resovled` 后运行并接受结果
- 第二个参数也是一个函数，在`promise rejected`后运行并接受error

```js
let promise = new Promise(function(resovle, reject) {
   setTimeout(() => resolve("done!"), 1000); 
});

//resolve运行，1s后执行第一个函数
promise.then(
	result => alert(result),	//done!
    error => alert(error),		//Error:Whoops!
);

//reject情况, 执行上面第二个函数
let promise = new Promise(function(resolve, reject) {
    setTimeout(() => reject(new Error('whoops!')), 1000);
});
```

- 如果只关心完成成功的结果！

```js
let promise = new Promise((resolve, reject) => {setTimeout(() => resolve("done~"), 1000)};
promise.then(alert); 	//1s后显示  done~
```

##### **catch**

- 如果我们只对error感兴趣

```js
// 使用null作为第一个参数
let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// .catch(f) 和 .then(null, f) 一样
promise.catch(alert);
```

- `.catch(f)` 调用是 `.then(null, f)` 的完全的模拟，它只是一个简写形式

##### **finally**

- 像常规的 `try {...} catch {...}` 中的 `finally` 子句一样，promise 中也有 `finally`。
- `.finally(f)` 调用与 `.then(f, f)` 类似，在某种意义上，`f` 总是在 promise 被 settled 时运行：即 promise 被 resolve 或 reject。
- `finally` 是执行清理（cleanup）的很好的处理程序（handler），例如无论结果如何，都停止使用不再需要的加载指示符（indicator）。

```js
new Promise((resolve, reject) => {
    /* 做一些需要时间的事儿，然后调用 resolve/reject */
})
// 在 promise 被 settled 时运行，无论成功与否
.finally(() => stop loading indicator)
.then(result => show result, err => show error)
```

1. `finally` 处理程序（handler）没有参数。在 `finally` 中，我们不知道 promise 是否成功。
2. `finally` 处理程序将结果和 error 传递给下一个处理程序。
3. `.finally(f)` 是比 `.then(f, f)` 更为方便的语法

##### **注意**

- 在 settled 的 promise 上，**`then` **会立即运行**

- promise 为 pending 状态，`.then/catch/finally` 处理程序（handler）将等待它。否则，如果 promise 已经是 settled 状态，它们就会立即执行，这也是Promise的优点

##### 示例：loadScript

- 前面我们提到，用于加载脚本的 `loadScript` 函数。

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));

  document.head.append(script);
}
```

- 下面用promise重写:

新函数 `loadScript` 将不需要回调。取而代之的是，它将创建并返回一个在加载完成时解析（resolve）的 promise 对象。外部代码可以使用 `.then` 向其添加处理程序（订阅函数）

```js
function loadScript(src) {
    return new Promise(function(resolve, reject) {
        let script = document.createElement('script');
        script.onload = () => resolve(script);
        script.onerror = () => reject(new Error(`Script load error for ${src}`));
        document.head.append(script);
    });
}

// 用法
let promise = loadscript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js");
promise.then(
	script => alert(`${script.src} is loaded`),
    error => alert(`Error ${error.message}`)
)
promise.then(script => alert('Another handler...'));
```

| Promise                                                      | Callbacks                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Promises 允许我们按照自然顺序进行编码。首先，我们运行 `loadScript` 和 `.then` 来处理结果。 | 在调用 `loadScript(script, callback)` 时，在我们处理的地方（disposal）必须有一个 `callback` 函数。换句话说，在调用 `loadScript` **之前**，我们必须知道如何处理结果。 |
| 我们可以根据需要，在 promise 上多次调用 `.then`              | 只能有一个回调。                                             |

- promise 为我们提供了更好的代码流和灵活性。


## Promise链

- 异步任务一个接着一个，怎样写出更好的代码呢？

**简单看一下Promise链：**

```js
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});
```

- 将 result 通过 `.then` 处理程序（handler）链进行传递。

{% asset_img Promise链.png Promise链 %}

- `promise.then` 的调用会返回了一个 promise，所以我们可以在其之上调用下一个 `.then`。
- 当处理程序（handler）返回一个值时，它将成为该 promise 的 result，所以将使用它调用下一个 `.then`。

**错误点**

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});
```

- 这里所做的只是一个 promise 的几个处理程序（handler）。它们不会相互传递 result；相反，它们之间彼此独立运行处理任务

{% asset_img 错误示范.png 错误示范 %}

### 返回promise

- `.then(handler)` 中所使用的处理程序（handler）可以创建并返回一个 promise

- 其他的处理程序（handler）将等待它 settled 后再获得其结果（result）


```js
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result); // 1

  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) { // (**)

  alert(result); // 2

  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {

  alert(result); // 4

});
```

- 与前面的示例相同：1 → 2 → 4，但是现在在每次 `alert` 调用之间会有 1 秒钟的延迟。
- 返回 promise 使我们能够构建异步行为链

### 示例：loadScript

- 与前面的loadScript函数结合

```js
loadScript("/article/promise-chaining/one.js")
  .then(script => loadScript("/article/promise-chaining/two.js"))
  .then(script => loadScript("/article/promise-chaining/three.js"))
  .then(script => {
    // 脚本加载完成，我们可以在这儿使用脚本中声明的函数
    one();
    two();
    three();
  });
```

- 每个 `loadScript` 调用都返回一个 promise，并且在它 resolve 时下一个 `.then` 开始运行。然后，它启动下一个脚本的加载。所以，脚本是一个接一个地加载的。

- 我们可以向每个 `loadScript` 直接添加 `.then`，但是那样是向右增长的，内层可以访问外层的变量有风险！！

>Thenables
>
>- 确切的说，处理程序返回的不完全是一个promise，而是一个被称为“thenable”对象，（一个具有`.then` 方法的任意对象，我们当做promise对象看待
>- 设计源于：因为实现了`.then`方法，第三方库可以实现自己的“promise”兼容对象，他们可以具有扩展的方法集，也和原生的promise兼容
>
>```js
>// 实例
>class Thenable {
>  constructor(num) {
>    this.num = num;
>  }
>  then(resolve, reject) {
>    alert(resolve); // function() { native code }
>    // 1 秒后使用 this.num*2 进行 resolve
>    setTimeout(() => resolve(this.num * 2), 1000);
>  }
>}
>
>new Promise(resolve => resolve(1))
>  .then(result => {
>    return new Thenable(result); // (*)
>  })
>  .then(alert); // 1000ms 后显示 2
>```
>
>- `(*)` 行中由 `.then` 处理程序（handler）返回的对象：如果它具有名为 `then` 的可调用方法，那么它将调用该方法并提供原生的函数 `resolve` 和 `reject` 作为参数（类似于 executor）
>- `resolve(2)` 在 1 秒后被调用，result 会被进一步沿着链向下传递。
>- 这个特性允许我们将自定义的对象与 promise 链集成在一起，而不必继承自 `Promise`。

### 更复杂的示例：fetch

- 前端编程中，promise通常用于网络请求

1. 使用 [fetch](https://zh.javascript.info/fetch) 方法从远程服务器加载用户信息

   ```js
   let promise = fetch(url);
   ```

   >1. 向`url` 发出网络请求，并返回一个promise。
   >
   >2. 当远程服务器返回 header（是在 **全部响应加载完成前**）时，该 promise 使用一个 `response` 对象来进行 resolve。
   >
   >3. 为了读取完整的响应，我们应该调用 `response.text()` 方法：当全部文字（full text）内容从远程服务器下载完成后，它会返回一个 promise，该 promise 以刚刚下载完成的这个文本作为 result 进行 resolve。

```js
fetch('/article/promise-chaining/user.json')
  // 当远程服务器响应时，下面的 .then 开始执行
  .then(function(response) {
    // 当 user.json 加载完成时，response.text() 会返回一个新的 promise
    // 该 promise 以加载的 user.json 为 result 进行 resolve
    return response.text();
  })
  .then(function(text) {
    // ...这是远程文件的内容
    alert(text); // {"name": "iliakan", "isAdmin": true}
  });
```

		>4. 从 `fetch` 返回的 `response` 对象还包括 `response.json()` 方法，该方法读取远程数据并将其解析为 JSON

```js
// 同上，但是使用 response.json() 将远程内容解析为 JSON
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => alert(user.name)); // iliakan, got user name
```

2. 多发一个到 GitHub 的请求，加载用户个人资料并显示头像：

```js
// 发送一个对 user.json 的请求
fetch('/article/promise-chaining/user.json')
  // 将其加载为 JSON
  .then(response => response.json())
  // 发送一个到 GitHub 的请求
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  // 将响应加载为 JSON
  .then(response => response.json())
  // 显示头像图片（githubUser.avatar_url）3 秒（也可以加上动画效果）
  .then(githubUser => {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => img.remove(), 3000); // (*)
  });
```

- 链没有扩展性，我们需要返回一个在头像显示结束时进行 resolve 的 promise。

```js
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
  .then(githubUser => new Promise(function(resolve, reject) { // (*)
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser); // (**)
    }, 3000);
  }))
  // 3 秒后触发
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
```

- 第 `(*)` 行的 `.then` 处理程序（handler）现在返回一个 `new Promise`，只有在 `setTimeout` 中的 `resolve(githubUser)` `(**)` 被调用后才会变为 settled。
- 异步行为应该始终返回一个 promise。

```js
// 整理后的代码
function loadJson(url) {
  return fetch(url)
    .then(response => response.json());
}

function loadGithubUser(name) {
  return fetch(`https://api.github.com/users/${name}`)
    .then(response => response.json());
}

function showAvatar(githubUser) {
  return new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  });
}

// 使用它们：
loadJson('/article/promise-chaining/user.json')
  .then(user => loadGithubUser(user.name))
  .then(showAvatar)
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
  // ...
```

**总结**

- 如果`.then` 处理程序返回一个promise，那么链的其他部分都会等待，知道他状态变为settled。
- 当他settled，其result（或error），将被进一步传递。

{% asset_img promise链图.png promise链图 %}

**思考**

```js
//这两个代码片段是否相等？换句话说，对于任何处理程序（handler），它们在任何情况下的行为都相同吗？

//(1)
promise.then(f1).catch(f2);
//(2)
promise.then(f1, f2);
```

- 不相同！！
- `.then` 将 result/error 传递给下一个 `.then/.catch`。所以在第一个例子中，在下面有一个 `catch`，而在第二个例子中并没有 `catch`，所以 error 未被处理。

## 使用 promise 进行错误处理

- Promise 链在错误（error）处理中十分强大。当一个 promise 被 reject 时，控制权将移交至最近的 rejection 处理程序（handler）

```js
// 例如：代码中url是错的
fetch('https://no-such-server.blabla') // reject
  .then(response => response.json())
  .catch(err => alert(err)) // TypeError: failed to fetch（这里的文字可能有所不同）
```

- 尝试可以看到，`.catch` 不必是立即的。它可能在一个或多个 `.then` 之后出现。

- 捕获所有 error 的最简单的方法是，将 `.catch` 附加到链的末尾：（当链中，哪一结promise被reject，就会被捕捉）


### 隐式 try…catch

- Promise 的执行者（executor）和 promise 的处理程序（handler）周围有一个“隐式的 `try..catch`”
- 如果发生异常，异常就会被捕获，并被视为 reject 进行处理。

1. 下面两段代码工作上相同

```js
new Promise((resolve, reject) => {
  throw new Error("Whoops!");
}).catch(alert); // Error: Whoops!
```

```js
new Promise((resolve, reject) => {
  reject(new Error("Whoops!"));
}).catch(alert); // Error: Whoops!
```

>在 executor 周围的“隐式 `try..catch`”自动捕获了 error，并将其变为 rejected promise

2. 同样在 handler 

```js
new Promise((resolve, reject) => {
  resolve("ok");
}).then((result) => {
  throw new Error("Whoops!"); // reject 这个 promise
}).catch(alert); // Error: Whoops!
```

>我们在 `.then` 处理程序（handler）中 `throw`，这意味着 promise 被 rejected，因此控制权移交至最近的 error 处理程序（handler）

3. 编程错误也是原因

```js
new Promise((resolve, reject) => {
  resolve("ok");
}).then((result) => {
  blabla(); // 没有这个函数
}).catch(alert); // ReferenceError: blabla is not defined
```

### 再次抛出

- 在常规的 `try..catch` 中，我们可以分析错误（error），如果我们无法处理它，可以将其再次抛出。对于 promise 来说，这也是可以的。
- 如果我们在 `.catch` 中 `throw`，那么控制权就会被移交到下一个最近的 error 处理程序（handler）。如果我们处理该 error 并正常完成，那么它将继续到最近的成功的 `.then` 处理程序（handler）。

1. 执行流：catch -> then

```js
new Promise((resolve, reject) => {

  throw new Error("Whoops!");

}).catch(function(error) {

  alert("The error is handled, continue normally");

}).then(() => alert("Next successful handler runs"));
```

>这里 `.catch` 块正常完成。所以下一个成功的 `.then` 处理程序（handler）就会被调用。

2. 执行流：catch -> catch

```js
new Promise((resolve, reject) => {

  throw new Error("Whoops!");

}).catch(function(error) { // (*)

  if (error instanceof URIError) {
    // 处理它
  } else {
    alert("Can't handle such error");

    throw error; // 再次抛出此 error 或另外一个 error，执行将跳转至下一个 catch
  }

}).then(function() {
  /* 不在这里运行 */
}).catch(error => { // (**)

  alert(`The unknown error has occurred: ${error}`);
  // 不会返回任何内容 => 执行正常进行

});
```

>执行从第一个 `.catch` `(*)` 沿着链跳转至下一个 `(**)`。

### 未处理的 rejection

- 当一个 error 没有被处理会发生什么？例如，我们忘了在链的尾端附加 `.catch`

```js
new Promise(function() {
  noSuchFunction(); // 这里出现 error（没有这个函数）
})
  .then(() => {
    // 一个或多个成功的 promise 处理程序（handler）
  }); // 尾端没有 .catch！
```

>当发生一个常规的错误（error）并且未被 `try..catch` 捕获时会发生什么？脚本死了，并在控制台（console）中留下了一个信息。对于在 promise 中未被处理的 rejection，也会发生类似的事儿。

>JavaScript 引擎会跟踪此类 rejection，在这种情况下会生成一个全局的 error。如果你运行上面这个代码，你可以在控制台（console）中看到。

- 在浏览器中，我们可以使用 `unhandledrejection` 事件来捕获这类 error：

```js
window.addEventListener('unhandledrejection', function(event) {
  // 这个事件对象有两个特殊的属性：
  alert(event.promise); // [object Promise] - 生成该全局 error 的 promise
  alert(event.reason); // Error: Whoops! - 未处理的 error 对象
});

new Promise(function() {
  throw new Error("Whoops!");
}); // 没有用来处理 error 的 catch
```

>如果出现了一个 error，并且在这儿没有 `.catch`，那么 `unhandledrejection` 处理程序（handler）就会被触发，并获取具有 error 相关信息的 `event` 对象，所以我们就能做一些后续处理了

- 通常此类 error 是无法恢复的，所以我们最好的解决方案是将问题告知用户，并且可以将事件报告给服务器。
- 在 Node.js 等非浏览器环境中，有其他用于跟踪未处理的 error 的方法。

### Fetch错误实例

- 当请求无法发出时，[fetch](https://developer.mozilla.org/zh/docs/Web/API/WindowOrWorkerGlobalScope/fetch) reject 会返回 promise
- 服务器返回一个错误 500 的非 JSON（non-JSON）页面该怎么办？
- 如果没有这个用户，GitHub 返回错误 404 的页面又该怎么办呢？

```js
fetch('no-such-user.json') // (*)
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`)) // (**)
  .then(response => response.json())
  .catch(alert); // SyntaxError: Unexpected token < in JSON at position 0
  // ...
```

>- 代码试图以 JSON 格式加载响应数据，但无论如何都会因为语法错误而失败。
>- 什么失败了，在哪里失败的。
>- 因此我们多添加一步：我们应该检查具有 HTTP 状态的 `response.status` 属性，如果不是 200 就抛出错误

```js
class HttpError extends Error { // (1)
  constructor(response) {
    super(`${response.status} for ${response.url}`);
    this.name = 'HttpError';
    this.response = response;
  }
}

function loadJson(url) { // (2)
  return fetch(url)
    .then(response => {
      if (response.status == 200) {
        return response.json();
      } else {
        throw new HttpError(response);
      }
    })
}

loadJson('no-such-user.json') // (3)
  .catch(alert); // HttpError: 404 for .../no-such-user.json
```

1. 我们为 HTTP 错误创建一个自定义类用于区分 HTTP 错误和其他类型错误。此外，新的类有一个 constructor，它接受 `response` 对象，并将其保存到 error 中。因此，错误处理（error-handling）代码就能够获得响应数据了。
2. 然后我们将请求（requesting）和错误处理代码包装进一个函数，它能够 fetch `url` **并** 将所有状态码不是 200 视为错误。这很方便，因为我们通常需要这样的逻辑。
3. 现在 `alert` 显示更多有用的描述信息。

- 拥有我们自己的错误处理类的好处是我们可以使用 `instanceof` 很容易地在错误处理代码中检查错误。

```js
// 例子：从 GitHub 加载给定名称的用户。如果没有这个用户，它将告知用户填写正确的名称
function demoGithubUser() {
  let name = prompt("Enter a name?", "iliakan");

  return loadJson(`https://api.github.com/users/${name}`)
    .then(user => {
      alert(`Full name: ${user.name}.`);
      return user;
    })
    .catch(err => {
      if (err instanceof HttpError && err.response.status == 404) {
        alert("No such user, please reenter.");
        return demoGithubUser();
      } else {
        throw err; // (*)
      }
    });
}

demoGithubUser();
```

>- 这里的 `.catch` 会捕获所有错误，但是它仅仅“知道如何处理” `HttpError 404`。在那种特殊情况下，它意味着没有这样的用户，而 `.catch` 仅仅在这种情况下重试。
>- 对于其他错误，仅仅是在 `(*)` 行再次抛出。

### 加载指示`.finally`

- 如果我们有加载指示（load-indication），`.finally` 是一个很好的处理程序（handler），在 fetch 完成时停止它：

```js
function demoGithubUser() {
  let name = prompt("Enter a name?", "iliakan");

  document.body.style.opacity = 0.3; // (1) 开始指示（indication）

  return loadJson(`https://api.github.com/users/${name}`)
    .finally(() => { // (2) 停止指示（indication）
      document.body.style.opacity = '';
      return new Promise(resolve => setTimeout(resolve)); // (*)
    })
    .then(user => {
      alert(`Full name: ${user.name}.`);
      return user;
    })
    .catch(err => {
      if (err instanceof HttpError && err.response.status == 404) {
        alert("No such user, please reenter.");
        return demoGithubUser();
      } else {
        throw err;
      }
    });
}

demoGithubUser();
```

>- 此处的 `(1)` 行，我们通过调暗文档来指示加载
>- 由于promise没有settled，先进入`.then` 返回`user`
>- 最后执行`.finally` 停止指示

- 有一个浏览器技巧，`(*)` 是从 `finally` 返回零延时（zero-timeout）的 promise。这是因为一些浏览器（比如 Chrome）需要“一点时间”外的 promise 处理程序来绘制文档的更改。因此它确保在进入链下一步之前，指示在视觉上是停止的。















