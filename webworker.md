# webWorker

在 html5 中有一个新的 API webWorker，这个api 使得原本只能是单线程的 javaScript 可以使用多线程的功能。

> Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。worker 线程必须等待主线程结束才开始。

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
let worker = new Worker(url,{name:"worker1"});
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



# 关于worker线程池

一般在开发的时候只需要使用的一个线程就足够解决我们的需求了，但是当我们需要使用多个线程的时候就需要使用线程池来帮助我们控制开启线程的数量了，如果只开一个线程，工作都在这一个子线程里做，不能保证它不阻塞。如果无止尽的开启而不进行控制，可能导致运行管理平台应用时，浏览器的内存消耗极高：一个web worker子线程的开销大概在5MB左右。

先规定线程通信的数据结构

```json
{
  //表示这个消息是否正确，也就是子线程在post这次message的时候，是否是因为报错而发过来，因为我们在子线程中会有这个设计机制，用来区分任务完成后的正常的消息和执行过程中因报错而发送的消息。如果为正常消息，我们约定为0，错误消息为1，暂定只有1。
  threadCode: 0,
  //表示消息真正的数据载体对象，如果threadCode为1，只返回taskId，以帮助主线程销毁找到调用上层promise的reject回调函数。Fecth取到的数据放在data内部。
  threadData: {taskId, data, code, msg}, 
  //表示消息错误的报错信息
  threadMsg:  'xxxxx',
  //表示数据频道，因为我们可能通过子线程做其他工作，在我们这个设计里至少有2个工作，一个是发起fetch请求，另外一个是响应主线程的检查(inspection)请求。所以需要一个额外的频道字段来确认不同工作。
  channel: 'fetch',
}
```

设计线程池

> 通过 Navagitor 对象的 HardWareConcurrecy 属性可以获取浏览器所属计算机的CPU核心数量，如果CPU有超线程技术，这个值就是实际核心数量的两倍。当然这个属性存在兼容性问题，如果取不到，则默认为4个。我们默认有多少个CPU线程数就开多少个子线程。线程池最大线程数量就这么确定了

```js
//基于webpack的项目最后使用这个work插件，方便通过import引用脚本
import work from 'webworkify-webpack';
class FetchThreadPool {
  constructor (option = {}){
    const {
      inspectIntervalTime = 10 * 1000,
      maximumWorkTime = 30 * 1000
    } = option;
    this.maximumThreadsNumber = window.navigator.hardwareConcurrency || 4;//最大可开线程数
    this.threads = [];//线程池容器
    this.inspectIntervalTime = inspectIntervalTime;//定时清理僵尸线程的时间
    this.maximumWorkTime = maximumWorkTime;//线程超时时间
    this.init();
  }
  //初始化开启所有线程待命
  init (){
    for (let i = 0; i < this.maximumThreadsNumber; i ++){
      this.createThread(i);
    }
    //设置定时器检查僵尸线程
    setInterval(() => this.inspectThreads(), this.inspectIntervalTime);
  }
  //创建线程
  createThread (i){
    // 开启子线程
    const thread = work(require.resolve('./fetch.worker.js'));
    // 绑定消息事件（相当于thread.onmessage=function(event){}）
    thread.addEventListener('message', event => {
      this.messageHandler(event, thread);
    });
    // 为线程添加一个自定义的id属性，用来标识线程
    thread['id'] = i;
    // 为线程添加一个自定义的busy属性，标识线程是否正在工作中
    thread['busy'] = false;
    // 为线程添加一个自定义的taskMap属性，用来记录当前线程执行的请求任务的promise的{resolve,reject}方法
    thread['taskMap'] = {};
    // 将线程保存到线程容器中
    this.threads[i] = thread;
  }
  //消息处理事件
  messageHandler (event, thread){
    //接收子线程返回的数据
    let {channel, threadCode, threadData, threadMsg} = event.data;
    // 线程运行成功
    if (threadCode === 0){
      switch (channel){
        case 'fetch':
          let {taskId, code, data, msg} = threadData;
          let reqPromise = thread.taskMap[taskId];
          if (reqPromise){
            // 使用promise 的 resolve/reject 方法处理并返回数据
            if (code === 0){
              reqPromise.resolve(data);
            } else {
              reqPromise.reject({code, msg});
            }
            // 处理完成后移除 promise
            thread.taskMap[taskId] = null;
          }
          // 设置线程的繁忙情况为 false
          thread.busy = false;
          break;
        case 'inspection':
          // 关闭掉超时的线程.
          let {isWorking, workTimeElapse} = threadData;
          if (isWorking && (workTimeElapse > this.maximumWorkTime)){
            console.warn(`Fetch worker thread ID: ${thread.id} is hanging up, details: ${JSON.stringify(threadData)}, it will be terminated.`);
            this.terminateZombieThread(thread);
          }
          break;
        default:
          break;
      }
    } else {
      // 线程出错
      if (threadData){
        let {taskId} = threadData;
        //设置线程繁忙状态为false
        thread.busy = false;
        let reqPromise = thread.taskMap[taskId];
        if (reqPromise){
          //返回错误信息
          reqPromise.reject({code: threadCode, msg: threadMsg});
        }
      }
    }
  }
  //关闭所有线程
  inspectThreads (){
    if (this.threads.length > 0){
      this.threads.forEach(thread => {
        //通知子线程关闭任务
        thread.postMessage({
          channel: 'inspection',
          data: {id: thread.id}
        });
      });
    }
  }
  // 杀掉线程，然后重新创建
  terminateZombieThread (thread){
    let id = thread.id;
    this.threads.splice(id, 1, null);
    thread.terminate();
    thread = null;
    this.createThread(id);
  }
  // 给线程分配任务
  dispatchThread ({url, options}, reqPromise){
    // 找到空闲的线程
    let thread = this.threads.find(thread => !thread.busy);
    if (!thread){
      //如果没有空闲的线程就在当前主线程行执行任务
      //fetchInMainThread({url, options});
    }else{
      // 生成一个新的任务id 用来映射 promise 处理方法
      let taskId = Date.now();
      thread.taskMap[taskId] = reqPromise;
      // 发送任务给子线程
      thread.postMessage({
        channel: 'fetch',
        data: {url, options, taskId}
      });
      thread.busy = true;
    }
  }
}
```

```js
export default self => {
  let isWorking = false;//是否工作中
  let startWorkingTime = 0;//上一次工作开始的时间
  let tasks = [];//任务队列
  self.addEventListener('message', async event => {
    const {channel, data} = event.data;
    switch (channel){
      case 'fetch':
        isWorking = true;
        startWorkingTime = Date.now();
        let {url, options, taskId} = data;
        tasks.push({url, options, taskId});
        try {
          //发送请求任务
          let response = await fetch(self.origin + url, options);
          if (response.ok){
            let {code, data, msg} = await response.json();
            self.postMessage({
              threadCode: 0,
              channel: 'fetch',
              threadData: {taskId, code, data, msg},
            });
          } else {
            const {status, statusText} = response;
            self.postMessage({
              threadCode: 0,
              channel: 'fetch',
              threadData: {taskId, code: status, msg: statusText || `http error, code: ${status}`},
            });
            console.info(`%c HTTP error, code: ${status}`, 'color: #CC0033');
          }
        } catch (e){
          self.postMessage({
            threadCode: 1,
            threadData: {taskId},
            threadMsg: `Fetch Web Worker Error: ${e}`
          });
        }
        isWorking = false;//任务执行完成工作状态改为否
        startWorkingTime = 0;//重置开始时间
        tasks = tasks.filter(task => task.taskId !== taskId);//从任务队列中将当前完成的任务移除
        break;
      case 'inspection':
        self.postMessage({
          threadCode: 0,
          channel: 'inspection',
          threadData: {
            isWorking,
            startWorkingTime,
            workTimeElapse: isWorking ? (Date.now() - startWorkingTime) : 0,
            tasks
          },
        });
        break;
      default:
        self.postMessage({
          threadCode: 1,
          threadMsg: `Fetch Web Worker Error: unknown message channel: ${channel}}.`
        });
        break;
    }
  });
};
```

参考于：[一个简单的HTML5 Web Worker 多线程与线程池应用](https://www.cnblogs.com/rock-roll/p/10176738.html)

