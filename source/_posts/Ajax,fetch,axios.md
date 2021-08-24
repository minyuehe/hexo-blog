---
title: Ajax,fetch,axios
date: 2020-10-29 17:52:53
categories: JavaScript
tags:
- 网络请求
---

### javascript的Ajax

- Ajax的全称是Asynchronous JavaScript and XML，意思就是用JavaScript执行异步网络请求，而不需要重载（刷新）整个页面。
- Ajax使用XMLHttpRequest对象取得新数据，然后再通过 DOM 将新数据插入到页面中。也就是无需刷新即可从服务器获取数据

##### **使用方法**

>对于IE7+和其他浏览器，直接使用XML对象，IE6以前使用ActiveXObject对象

```js
var xhr;
if(window.XMLHttpRquest) {
  xhr = new XMLHttpRequest();
} else {
  xhr = new ActiveXObject('Microsoft.XMLHTTP');
}
```

##### **启动请求**

```js
xhr.open(method, url, boolean);
xhr.send([body]);
```

>method:请求方法：post，get等
>
>url：请求链接，只能向同源url发送
>
>boolean：是否异步，默认true（异步）
>
>body:包含request.body比如post要使用请求体，get方法填null或不填
>
>注意：调用open并不会真正发送请求，只是启动一个请求以备发送

##### **监听请求**

>load---请求完成（HTTP状态为400或500），并且响应已完全下载
>
>error---当无法送出请求，如网络中断，无效URl
>
>progress---在下载响应期间定期触发，报告下载了多少

```js
xhr.onload = function() {
  alert(`Loaded: ${xhr.status} ${xhr.response}`);
};

xhr.onerror = function() { // 仅在根本无法发出请求时触发
  alert(`Network Error`);
};

xhr.onprogress = function(event) { // 定期触发
  // event.loaded —— 已经下载了多少字节
  // event.lengthComputable = true，当服务器发送了 Content-Length header 时
  // event.total —— 总字节数（如果 lengthComputable 为 true）
  alert(`Received ${event.loaded} of ${event.total}`);
};
```

>status: HTTP状态码，200,404,403等，非HTTP错误为0；
>
>statusText：状态信息，200状态码对应OK；404对应Not Found；403对应Forbidden
>
>response：旧版本为responseText，服务器响应体

我们还可以指定**超时**

```
xhr.timeout = 10000; //timeout单位是ms
```

URL搜索参数（确保正确编码）

```js
let url = new URL('https://google.com/search');
url.searchParams.set('q','test me!');

//编码
xhr.open('GET', url);//https://google.com/search?q=test+me%21
```

**readyState**

xml的状态由 `0` → `1` → `2` → `3` → … → `3` → `4` 通过网络接收到一个数据包就重复一次状态`3`

```js
UNSENT = 0; // 初始状态
OPENED = 1; // open 被调用
HEADERS_RECEIVED = 2; // 接收到 response header
LOADING = 3; // 响应正在被加载（接收到一个数据包）
DONE = 4; // 请求完成
```

```js
//以前的监听事件就是这样实现的；现在被load，error，progress代替
xhr.onreadystatechange = function() {
  if(xhr.readyState === 3) {
    //加载中
  }
  if(xhr.readyState === 4) {
    //请求完成
  }
}
```

<!--more-->

##### **完整示例**

```js
//1.创建一个XML对象
let xhr = new XMLHttpRequest();
//2.配置请求
xhr.open('GET','同源url');
//3.发送网络请求
xhr.send();

//4.收到响应后调用
xhr.onload = function() {
  if(xhr.status !== 200) {
    alert(`Error${xhr.status}:${xhr.statusText}`);
  } else {
    alert(`Done,got ${xhr.response.length} bytes`)
  }
}

xhr.onprogress = function(event) {
  if(event.lengthComtable) {
    alert(`Received ${event.loaded} of ${event.total} bytes`);
  } else {
    alert(`Received ${event.loaded} bytes`);
  }
}

xhr.onerror = function() {
  alert('Request failed');
}
```

#####  响应类型

使用`xhr.responseType`设置响应格式

- “” （默认）：响应格式为字符串
- “text”：字符串
- “arraybuffer”：响应格式为ArrayBuffer，二进制数据
- “blob”：响应格式为Blod，二进制数据
- “document“：响应格式为XML document
- “json”：响应为JSON（自动解析）

以JSON为例

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/example/json');

xhr.responseType = 'json';

xhr.send();

// 响应为 {"message": "Hello, world!"}
xhr.onload = function() {
  let responseObj = xhr.response;
  alert(responseObj.message); // Hello, world!
};
```

##### 终止请求

随时都可以中止请求，调用`xhr.abort()`;触发abort事件，xhr.status变为0

##### 同步请求

open的第三个参数为`false`,阻塞脚本执行，像alert

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/hello.txt', false);

try {
  xhr.send();		//阻塞
  if (xhr.status != 200) {
    alert(`Error ${xhr.status}: ${xhr.statusText}`);
  } else {
    alert(xhr.response);
  }
} catch(err) { // 代替 onerror
  alert("Request failed");
}
```

缺点：阻塞程序执行；没有进度指示；不能向其他域发送请求；不能设置超时

##### HTTP-Header

XML允许发送自定义header，并且可以从响应中读取header

1. setRequestHeader(name, value)

   ```js
   xhr.setRequestHeader('Content-Type', 'application/json');
   ```

2. getResponseHeader(name)/getAllResponseHeaders()

   ```
   xhr.getResponseHeader('Content-Type');
   
   //header单行形式返回
   Cache-Control: max-age=31536000
   Content-Length: 4260
   Content-Type: image/png
   Date: Sat, 08 Sep 2012 16:53:16 GMT
   ```

注意：不能获取set-cookie；header只加不减，不能移除；对于一些浏览器专门管理的header不能修改（Referer，Host）

**补充：Host，Origin，Referer**

>1.Host:请求将被发送的目的地，**包括域名端口号**
>
>2.Referer：告诉服务器请求的原始资源URL，用于所有请求；**协议+域名+查询参数（不包括锚点信息）**；file协议下是不带referer的
>
>3.Origin：说明请求从哪里发起，**协议+域名**；存在CORS请求或者POST请求

##### 获取header对象

header之间都是'/r/n'为换行符；并且name和value之间都是':'分割

```js
let headers = xhr
  .getAllResponseHeaders()
  .split('\r\n')
  .reduce((result, current) => {
    let [name, value] = current.split(': ');
    result[name] = value;
    return result;
  }, {});

// headers['Content-Type'] = 'image/png'
```

##### POST，FormData

POST请求可以很好地使用FormData对象作为请求体

```js
let formData = new FormData([form]); // 创建一个对象，可以选择从 <form> 中获取数据
formData.append(name, value); // 附加一个字段
```

1. `xhr.open('POST', ...)` —— 使用 `POST` 方法。
2. `xhr.send(formData)` 将表单发送到服务器。

示例：

```html
<form name="person">
  <input name="name" value="John">
  <input name="surname" value="Smith">
</form>

<script>
  // 从表单预填充 FormData
  let formData = new FormData(document.forms.person);

  // 附加一个字段
  formData.append("middle", "Lee");

  // 将其发送出去
  let xhr = new XMLHttpRequest();
  xhr.open("POST", "/article/xmlhttprequest/post/user");
  xhr.send(formData);

  xhr.onload = () => alert(xhr.response);
</script>
```

以 `multipart/form-data` 编码发送表单。

**补充**：HTML表单的entype三种类型

1. application/x-www-urlencoded

- 默认编码模式，数据会被以x-www-urlencoded 方式编码到 Body 中来传送，如果是GET请求，会附在url后面发送（GET只支持ASCII字符集，其一个劣势）

- 编码方式：数据会被编码成以`&`分隔的键值对；字符以URL编码方式编码。

```
// 转换过程: {a: 1, b: 2} -> a=1&b=2 -> 如下(最终形式)
"a%3D1%26b%3D2"
```

2. multipart/form-data

- 请求头中`Content-Type`字段会包含`boundary`（由浏览器默认指定）
- 数据会分为多个部分，每一个部分都通过分隔符分隔，并且都有HTTP头部描述子包体，最后`--`结束

```
//请求体
Content-Disposition: form-data;name="data1";
Content-Type: text/plain
data1
----WebkitFormBoundaryRRJKeWfHPGrS4LKe
Content-Disposition: form-data;name="data2";
Content-Type: text/plain
data2
----WebkitFormBoundaryRRJKeWfHPGrS4LKe--
```

**优点**：每个表单元素都是独立的资源表述；对于一些文件上传，基本使用这个，而不是前者，没必要url编码。

3. text/plain

- 按照键值对排列表单数据`key1=value1\r\nkey2=value2`，不进行转义。

**使用JSON字符串形式发送**

- 不要忘了设置 `Content-Type: application/json`

```js
let xhr = new XMLHttpRequest();

let json = JSON.stringify({
  name: "John",
  surname: "Smith"
});

xhr.open("POST", '/submit')
xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');

xhr.send(json);
```

##### 上传进度

对于下载进度我们有onprogress事件监听，上传进度`xhr.upload`

- `loadstart` —— 上传开始。
- `progress` —— 上传期间定期触发。
- `abort` —— 上传中止。
- `error` —— 非 HTTP 错误。
- `load` —— 上传成功完成。
- `timeout` —— 上传超时（如果设置了 `timeout` 属性）。
- `loadend` —— 上传完成，无论成功还是 error。

**示例**

```html
<input type="file" onchange="upload(this.files[0])">
<!--onchange事件:对于file类型在选择文件后触发-->
<script>
function upload(file) {
  let xhr = new XMLHttpRequest();

  // 跟踪上传进度
  xhr.upload.onprogress = function(event) {
    console.log(`Uploaded ${event.loaded} of ${event.total}`);
  };

  // 跟踪完成：无论成功与否
  xhr.onloadend = function() {
    if (xhr.status == 200) {
      console.log("success");
    } else {
      console.log("error " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
</script>
```

### jQuery的Ajax

```js
$.ajax({
  url:"",
  type:"GET",
  contentType: '',
  async:true,
  data:{},
  dataType:"",
  success: function(){
  }
});
```

>url 必填项，规定把请求发送到哪个 URL。
>
>type 以什么样的方式获取数据，是get或post
>
>contentType：发送POST请求的格式，默认值为’application/x-www-form-urlencoded;
>
>charset=UTF-8’，也可以指定为text/plain、application/json
>
>async 是否异步执行AJAX请求，默认为true，千万不要指定为false
>
>data 发送的数据，可以是字符串、数组或object。如果是GET请求，data将被转换成query附加到URL上，如果是POST请求，根据contentType把data序列化成合适的格式；
>
>dataType
>
>接收的数据格式，可以指定为’html’、‘xml’、‘json’、'text’等，缺省情况下根据响应的Content-Type猜测。
>
>success 可选。执行成功时返回的数据。

是基于xhr开发的，针对MVC模式的编程模式，不太适合当前的MVVM。jQuery本身比较大。

### Axios

也是对原生XHR的封装；Promise实现版本，很好的实现异步逻辑。可以用于浏览器和node的HTTP库。

- 从浏览器中创建 [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- 从 node.js 创建 [http](http://nodejs.org/api/http.html) 请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 拦截请求和响应
- 转换请求数据和响应数据
- 取消请求
- 自动转换 JSON 数据
- 客户端支持防御 [XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

##### 设置全局axios默认值

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'
```

> 注：axios的headers的content-type默认`application/json`
>
> 默认情况下，axios会将js对象序列化为JSON，get请求对请求参数不用处理；post请求并且content-type为application/x-www-form-urlencoded，需要使用URLSearchParms格式化请求参数，否则content-type依然是application/JSON
>
> ```js
> var params = new URLSearchParams();
> params.append('param1', 'value1');
> params.append('param2', 'value2');
> ```

##### get请求三种方法

```js
// 第一种写法
axios.get('/user?id=12345&name=xiaoming')
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});

// 第二种写法
axios.get('/user', {
    params: {
      id: '12345'，
      name: 'xiaoming'
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});

// 第三种写法
axios({
    url: '/user',
    method: 'get',
    params: {
      id: '12345'，
      name: 'xiaoming'
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});
```

##### post请求两种方法

```js
// 第一种写法
axios({
    url: '/user',
    method: 'post',
    headers: {
        'Content-Type': 'application/json'
    },
    data: {
      id: '12345'，
      name: 'xiaoming'
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});

// 第二种写法
var url = '/user';
var data = {
      id: '12345'，
      name: 'xiaoming'
    };
axios.post(url, data, {
       headers: {
        'Content-Type': 'application/json'
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});
```

##### 并发请求

> axios.all()  axios.spread()

```js
function getUserAccount() {
  return axios.get('/user/12345');
}
 
function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}
 
axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
}));
```

##### 创建实例

> axios.create([config])

```js
var instance = axios.create({
  baseURL: 'http://localhost:3000/',
});
```

> 配置项优先级：

> config参数 > 实例的defaults属性 > node_modules/axios/lib/defaults.js 找到的库的默认值

```js
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();
 
// 覆写库的超时默认值
// 现在，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;
 
// 为已知需要花费很长时间的请求覆写超时设置
instance.get('/longRequest', {
  timeout: 5000
});
```

##### 拦截器

> 请求发出之前后者响应被then或catch处理前拦截他们做预处理

```js
//axiosInstance为自定义实例
axiosInstance.interceptors.response.use(    //request类似
  (res) => res.data,
  (err) => {
    console.log(err, "网络错误");
  }
);
```

> 可以在稍后移除拦截器

```js
var myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

### fetch

> 一个 基于promise设计的low-level API
>
> 优点：实现关注点分离，输入，输出，事件跟踪分离
>
> 缺点：不支持同步；只对网络请求报错，对400和500都当做成功请求，需要封装处理；

##### **基本使用**

```js
//1.await调用
let response = await fetch(url, options); // 解析 response header
let result = await response.json(); // 将 body 读取为 json

//2.promise形式
fetch(url, options)
  .then(response => response.json())
  .then(result => /* process result */)
```

- **options**

>method：HTTP方法
>
>headers：一个对象（不是所有的header都可以编写）
>
>body：post等方法时才有请求体
>
>- 字符串（例如 JSON 编码的），
>- `FormData` 对象，以 `form/multipart` 形式发送数据
>- `Blob`/`BufferSource` 发送二进制数据，
>- URLSearchParams，以 `x-www-form-urlencoded` 编码形式发送数据，很少使用。

- **response**

>**响应属性**：status，ok（状态码为2XX即为，true），headers（类似于map的对象）
>
>**响应体：**只能获取一次！！
>
>- **`response.text()`** —— 读取 response，并以文本形式返回 response，
>- **`response.json()`** —— 将 response 解析为 JSON 对象形式，
>- **`response.formData()`** —— 以 `FormData` 对象（form/multipart 编码，参见下一章）的形式返回 response，
>- **`response.blob()`** —— 以 [Blob](https://zh.javascript.info/blob)（具有类型的二进制数据）形式返回 response，
>- **`response.arrayBuffer()`** —— 以 [ArrayBuffer](https://zh.javascript.info/arraybuffer-binary-arrays)（低级别的二进制数据）形式返回 response。

##### 实例：上传图片

> 使用Blob对象通过fetch提交二进制数据，`<canvas>`作为画布

```html
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Submit" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let blob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));
      let response = await fetch('/article/fetch/post/image', {
        method: 'POST',
        body: blob
      });

      // 服务器给出确认信息和图片大小作为响应
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

![cavans](Ajax,fetch,axios/image-20210331151953251.png)

> 注：没有手动设置content/type，因为Bolb对象通过toBlob生成带有，"image/png"，这个类型就是content-type值

```js
//promise改写
function submit() {
  canvasElem.toBlob(function(blob) {
    fetch('/article/fetch/post/image', {
      method: 'POST',
      body: blob
    })
      .then(response => response.json())
      .then(result => alert(JSON.stringify(result, null, 2)))
  }, 'image/png');
}
```

##### 实例：fetch github用户信息

>要获取一个用户，我们需要：`fetch('https://api.github.com/users/USERNAME')`.
>
>如果响应的状态码是 `200`，则调用 `.json()` 来读取 JS 对象。
>
>否则，如果 `fetch` 失败，或者响应的状态码不是 `200`，我们只需要向结果数组返回 `null` 即可

```js
async function getUsers(names) {
  let jobs = [];

  for(let name of names) {
    let job = fetch(`https://api.github.com/users/${name}`).then(
      successResponse => {
        if (successResponse.status != 200) {
          return null;
        } else {
          return successResponse.json();
        }
      },
      failResponse => {
        return null;
      }
    );
    jobs.push(job);
  }

  let results = await Promise.all(jobs);

  return results;
}
```

> 这里使用Promise.all确保都resolve结束再返回；直接.then而不是在Promise.all中实现，使得json解析不需要累计解决，提升效率。

##### POST和FormData

fetch 可以接受一个FormData作为body，并编码发送出去，带有 `Content-Type: multipart/form-data`

**实例：发送一个简单的表单**

```html
<form id="formElem">
  <input type="text" name="name" value="John">
  <input type="text" name="surname" value="Smith">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

![fetch-formdata](Ajax,fetch,axios/image-20210401054132188.png)

> 修改FormData中的字段
>
> - `formData.append(name, value)` —— 添加具有给定 `name` 和 `value` 的表单字段，
> - `formData.append(name, blob, fileName)` —— 添加一个字段，就像它是 `<input type="file">`，第三个参数 `fileName` 设置文件名（而不是表单字段名），因为它是用户文件系统中文件的名称，
> - `formData.delete(name)` —— 移除带有给定 `name` 的字段，
> - `formData.get(name)` —— 获取带有给定 `name` 的字段值，
> - `formData.has(name)` —— 如果存在带有给定 `name` 的字段，则返回 `true`，否则返回 `false`。
>
> 从技术上来讲，一个表单可以包含多个具有相同 `name` 的字段，因此，多次调用 `append` 将会添加多个具有相同名称的字段。
>
> 还有一个 `set` 方法，语法与 `append` 相同。不同之处在于 `.set` 移除所有具有给定 `name` 的字段，然后附加一个新字段。因此，它确保了只有一个具有这种 `name` 的字段，其他的和 `append` 一样：
>
> - `formData.set(name, value)`，
> - `formData.set(name, blob, fileName)`。
>
> 我们也可以使用 `for..of` 循环迭代 formData 字段：
>
> ```js
> let formData = new FormData();
> formData.append('key1', 'value1');
> formData.append('key2', 'value2');
> 
> // 列出 key/value 对
> for(let [name, value] of formData) {
>   alert(`${name} = ${value}`); // key1=value1，然后是 key2=value2
> }
> ```

**实例：发送带有文件的表单**

表单始终以 `Content-Type: multipart/form-data` 来发送数据，这个编码允许发送文件。因此 `<input type="file">` 字段也能被发送

```js
<form id="formElem">
  <input type="text" name="firstName" value="John">
  Picture: <input type="file" name="picture" accept="image/*">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user-avatar', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

![fetch-file](Ajax,fetch,axios/image-20210401055022057.png)

**实例：发送具有Blob的表单**

前面我们知道可以使用Blob发送一个动态生成的二进制数据（图片）；实际更加方便的发送图片的方式不是单独发，而是作为表单的一部分，并带有附加字段（比如：“name”和其他的metadata）一起发

服务器也更加适合接受多部分编码的表单（multipart-encoded form），而不是原始的二进制数据。

```html
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Submit" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let imageBlob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));

      let formData = new FormData();
      formData.append("firstName", "John");
      formData.append("image", imageBlob, "image.png");

      let response = await fetch('/article/formdata/post/image-form', {
        method: 'POST',
        body: formData
      });
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

注意：图片Blob是如何添加的？

```js
//像表单 <input type="file" name="image"> 一样
formData.append("image", imageBlob, "image.png");
```

##### 上传下载进度

>无法跟踪上传进度（请看XMLHttpRequest）
>
>下载进度：使用`response.body`属性,给与进度读取完全控制。

```js
// Step 1：启动 fetch，并获得一个 reader
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits?per_page=100');

const reader = response.body.getReader();

// Step 2：获得总长度（length）
const contentLength = +response.headers.get('Content-Length');

// Step 3：读取数据
let receivedLength = 0; // 当前接收到了这么多字节
let chunks = []; // 接收到的二进制块的数组（包括 body）
while(true) {
  const {done, value} = await reader.read();

  if (done) {
    break;
  }

  chunks.push(value);
  receivedLength += value.length;

  console.log(`Received ${receivedLength} of ${contentLength}`)
}

// Step 4：将块连接到单个 Uint8Array
let chunksAll = new Uint8Array(receivedLength); // (4.1)
let position = 0;
for(let chunk of chunks) {
  chunksAll.set(chunk, position); // (4.2)
  position += chunk.length;
}

// Step 5：解码成字符串
let result = new TextDecoder("utf-8").decode(chunksAll);

// 我们完成啦！
let commits = JSON.parse(result);
alert(commits[0].author.login);
```

>1. `response.body.getReader()`:获取一个流读取器
>
>2.  `Content-Length` :跨域请求可能不存在这个 header
>
>3.  `await reader.read()`：响应块放在chunks中
>
>4. 合并这些`Uint8Array` 字节块数组
>
>5. 解码为字符串
>
>   `result = new TextDecoder("utf-8").decode(chunksAll);`
>
>6. 或者想要的到二进制数据，直接将chunks这个字节数组变为二进制
>
>   `blob = new Blob(chunks);`

##### 中止（Abort）

>js通常没有终止promise的概念，但提供了一个内建对象`AbortController`，不仅可以终止fetch，还可以终止其他异步任务

**AbortController 对象**

> 创建一个控制器`let controller = new AbortController();`
>
> - 一个方法abort()
> - 一个属性signal，设置事件监听器
>
> 当 `abort()` 被调用时：
>
> - `controller.signal` 就会触发 `abort` 事件。
> - `controller.signal.aborted` 属性变为 `true`。

```js
let controller = new AbortController();
let signal = controller.signal;

// 可取消的操作这一部分
// 获取 "signal" 对象，
// 并将监听器设置为在 controller.abort() 被调用时触发
signal.addEventListener('abort', () => alert("abort!"));

// 另一部分，取消（在之后的任何时候）：
controller.abort(); // 中止！

// 事件触发，signal.aborted 变为 true
alert(signal.aborted); // true
```

**结合Fetch**

>作为fetch的一个option进行传递

```js
let controller = new AbortController();
fetch(url, {
  signal: controller.signal
});
```

**实例：1s后终止fetch**

>`fetch` 从 `signal` 获取了事件并中止了请求。当一个 fetch 被中止，它的 promise 就会以一个 error `AbortError` reject

```js
// 1 秒后中止
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);

try {
  let response = await fetch('/article/fetch-abort/demo/hang', {
    signal: controller.signal
  });
} catch(err) {
  if (err.name == 'AbortError') { // handle abort()
    alert("Aborted!");
  } else {
    throw err;
  }
}
```

**例子：终止其他异步promise**

> 核心就是添加abort监听事件

```js
let urls = [...];
let controller = new AbortController();

let ourJob = new Promise((resolve, reject) => { // 我们的任务
  ...
  controller.signal.addEventListener('abort', reject);
});

let fetchJobs = urls.map(url => fetch(url, { // fetches
  signal: controller.signal
}));

// 等待完成我们的任务和所有 fetch
let results = await Promise.all([...fetchJobs, ourJob]);

// 如果 controller.abort() 被从其他地方调用，
// 它将中止所有 fetch 和 ourJob
```























