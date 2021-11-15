# webWorker

在 html5 中有一个新的 API webWorker，这个api 使得原本只能是单线程的 javaScript 可以使用多线程的功能。

> Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

## webworker 使用的限制

### 同源限制

> 分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

### DOM 限制

> Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用`document`、`window`、`parent`这些对象。但是，Worker 线程可以`navigator`对象和`location`对象。

### 通信联系

> Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。
>
> 通过 `worker.postMessage()` 来相互发送信息，通过`worker.onmessage()`接收信息 

### 脚本限制

> Worker 线程不能执行`alert()`方法和`confirm()`方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

### 文件限制

> Worker 线程无法读取本地文件，即不能打开本机的文件系统（`file://`），它所加载的脚本，必须来自网络。



## webWorker 的全局对象及全局方法

在 webworker 中全局对象不是 window，也不是document，它有自己的全局对象，可以使用 self 或者 this 进行访问，在worker 的全局对象上有一些自己的全局方法。

- `self.name`： Worker 的名字。该属性只读，由构造函数指定`new Worker('worker.js', { name : 'myWorker' });`。
- `self.onmessage`：指定`message`事件的监听函数。
- `self.onmessageerror`：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- `self.close()`：关闭 Worker 线程。
- `self.postMessage()`：向产生这个 Worker 线程发送消息。
- `self.importScripts()`：加载 JS 脚本。



## webWorker实例上的API

- `Worker.onerror`：指定 error 事件的监听函数。
- `Worker.onmessage`：指定 message 事件的监听函数，发送过来的数据在`Event.data`属性中。
- `Worker.onmessageerror`：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- `Worker.postMessage()`：向 Worker 线程发送消息。
- `Worker.terminate()`：立即终止 Worker 线程。



## 基本使用

### 创建子线程

首先需要一个 worker.js 文件，这个文件就是子线程的执行文件。

**worker.js**

```js
// 添加接收消息事件
this.onmessage = function (e) {
  // 处理完之后将数据回给主线程
  this.postMessage('你发过来的是: ' + e.data);
}
```

在 worker.js 中 `this`也可以使用`self`代替，因为他们是相等的，而且对于`onmessage`这些全局方法可以不写 `this/self`；如果 `message`数据有多个函数需要绑定的话可以使用`addEventListener`进行绑定

```js
addEventListener('message', function (e) {
  postMessage('你发过来的是: ' + e.data);
}, false);
```



**创建子线程**

```js
let worker = new Worker("worker.js",{name:"worker1"});
```

`Worker()`构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，**且只能加载 JS 脚本，并且必须来自网络**，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

url 除了可以是 同源在线 js 脚本外还可以是`DOMString`，可以通过`URL.createObjectURL(object)`来创建。

```js
let blob = new Blob(["(function(){console.log(222)})()"]);
let url = window.URL.createObjectURL(blob);
let worker = new Worker("worker.js",{name:"worker1"});
```

> **`URL.createObjectURL(object)`创建的对象是不是自动销毁的，需要使用`URL.revokeObjectURL(url);`来销毁，否则可能导致内存溢出。**



### 添加数据传输事件监听

```js
worker.onmessage = function (event) {
  console.log('Received message ' + event.data);
  doSomething();
}

function doSomething() {
  // 执行任务
  worker.postMessage('Work done!');
}
```



### 错误处理

当worker 发生错误时，会触发主线程的 `onerror`事件，可以用来对worker 中的错误做处理

```js
worker.addEventListener('error', function (event) {
  // ...
});
```



### 关闭 webworker

使用完毕，为了节省系统资源，必须关闭 Worker。

```js
// 主线程
worker.terminate();

// Worker 线程
self.close();
```



### 加载脚本

在 worker 中可以使用`importScripts(url)`方法加载脚本。

```js
//加载一个 脚本
importScripts('script1.js');
//加载多个
importScripts('script1.js', 'script2.js');
```



### Worker 线程完成轮询

有时，浏览器需要轮询服务器状态，以便第一时间得知状态改变。这个工作可以放在 Worker 里面。

```js
function createWorker(f) {
  var blob = new Blob(['(' + f.toString() +')()']);
  var url = window.URL.createObjectURL(blob);
  var worker = new Worker(url);
  return worker;
}

var pollingWorker = createWorker(function (e) {
  var cache;

  function compare(new, old) { ... };

  setInterval(function () {
    fetch('/my-api-endpoint').then(function (res) {
      var data = res.json();

      if (!compare(data, cache)) {
        cache = data;
        self.postMessage(data);
      }
    })
  }, 1000)
});

pollingWorker.onmessage = function () {
  // render data
}

pollingWorker.postMessage('init');
```

上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。



## 关于数据传输的问题

主线程与 Worker 之间的通信内容，可以是文本，也可以是对象。需要注意的是，这种通信是拷贝关系，**即是传值而不是传址**，Worker 对通信内容的修改，不会影响到主线程。事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给 Worker，后者再将它还原。

主线程与 Worker 之间也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等类型，也可以在线程之间发送。

```js
// 主线程
var uInt8Array = new Uint8Array(new ArrayBuffer(10));
for (var i = 0; i < uInt8Array.length; ++i) {
  uInt8Array[i] = i * 2; // [0, 2, 4, 6, 8,...]
}
worker.postMessage(uInt8Array);

// Worker 线程
self.onmessage = function (e) {
  var uInt8Array = e.data;
  postMessage('Inside worker.js: uInt8Array.toString() = ' + uInt8Array.toString());
  postMessage('Inside worker.js: uInt8Array.byteLength = ' + uInt8Array.byteLength);
};
```

但是，拷贝方式发送二进制数据，会造成性能问题。比如，主线程向 Worker 发送一个 500MB 文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做[Transferable Objects](https://www.w3.org/html/wg/drafts/html/master/infrastructure.html#transferable-objects)。这使得主线程可以快速把数据交给 Worker，对于影像处理、声音处理、3D 运算等就非常方便了，不会产生性能负担。

如果要直接转移数据的控制权，就要使用下面的写法。

```js
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);

// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```



**来源于阮一峰老师的** [Web Worker 使用教程](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)



# SharedWorker

webWorker 其实有两种，一种是上面说的 webWorker ，它是一种**专用线程**，当页面关闭的时候结束，这就意味着专用线程只能被创建它的页面访问。还有一种就是 **sharedWorker 共享线程**，它可以被与之关联的多个页面访问。**只有当所有关联的页面都关闭的时候才会结束。**

> Worker 对应的是 专用线程；SharedWorker 对应的是共享线程
>
> 可以使用 Modernizr 库来检测浏览器对 js，css 的支持性

## sharedWorker 中的 属性及 API

- `SharedWorker.port`：返回一个[`MessagePort`](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort)用于与共享工作者通信并控制它的对象。
- `SharedWorker.onerror`：当事件监听发送错误时触发
- `SharedWorker.onconnect`：对于worker 的事件，有新连接的时候执行。

## MessagePort 中的 API

- `port.postMessage()`：从端口发送消息，并可选择将对象的所有权转移到其他浏览器上下文，如`port.postMessage(a,[a])`。
- `port.start()`：开始发送在端口上排队的消息，**如果是使用`post.addEventListener("message",()=>{})`进行事件绑定的话，默认是不会启动端口的，需要用`port.start()`启动，如果使用的是`port.onmessage`进行事件绑定的话就会自动开启端口。**
- `port.close()`：断开端口，使其不再处于活动状态。
- `port.onmessage`：开启消息监听，会自动执行 `port.stat()`
- `port.onmessageeror`：数据错误监听，传递过来的数据不能被序列化时执行

## 实例

在服务器上有一个 worker.js

```js
var data;
onconnect = function(e) {
	ports = e.ports;
  var port = ports[0];
  port.onmessage = function(e) {
    if(e.data=='get'){
        port.postMessage(data);
    }else{
        data=e.data;
    }
  }
}
```

在服务器上有一个页面 a.html

```html
<div id="log"></div>
<input type="text" name="" id="txt">
<button id="get">get</button>
<button id="set">set</button>
<script>
  var worker = new SharedWorker('worker.js');
  var get = document.getElementById('get');
  var set = document.getElementById('set');
  var txt = document.getElementById('txt');
  var log = document.getElementById('log');

  worker.port.addEventListener('message', function(e) {
    log.innerText = e.data;
	console.log(e);
  }, false);
  worker.port.addEventListener('messageerror', function(e) {
	console.log(e);
  }, false);
  worker.port.start(); // note: need this when using addEventListener

  set.addEventListener('click',function(e){
      worker.port.postMessage(txt.value);
  },false);

  get.addEventListener('click',function(e){
      worker.port.postMessage('get');
  },false);
</script>
```

在服务器上有一个页面 b.html

```html
<div id="log"></div>
<input type="text" name="" id="txt">
<button id="get">get</button>
<button id="set">set</button>
<script>
  var worker = new SharedWorker('worker.js');
  var get = document.getElementById('get');
  var set = document.getElementById('set');
  var txt = document.getElementById('txt');
  var log = document.getElementById('log');

  worker.port.addEventListener('message', function(e) {
    log.innerText = e.data;
	console.log(e);
  }, false);
  worker.port.addEventListener('messageerror', function(e) {
	console.log(e);
  }, false);
  worker.port.start(); // note: need this when using addEventListener

  set.addEventListener('click',function(e){
      worker.port.postMessage(txt.value);
  },false);

  get.addEventListener('click',function(e){
      worker.port.postMessage('get');
  },false);
</script>ttttt
```

**需要注意的是**：sharedWorker 是会被加载到浏览器执行的，所以是不能跨浏览器通信的，还有就是因为只有当所有关联的页面都关闭的时候才会结束。所以当修改了 worker.js 之后通过刷的方式是不能加载到最新的代码的，因为只要还有一个相关的网页打开那么就会使用旧的那份代码，所以要用新修改的就需要将相关的网页页签全部关闭到然后再重新打开。

 