# webSocket

websocket 是一种 H5 通过的新的网络通行方式，是一种新的协议实现了浏览器与服务器全双工通信，**一开始的握手需要借助Http请求**。这个网路通行方式相比于以前的是 http 网络通行的优点是可以保持持久连接，服务端可以主动发送 信息给客户端。在以往要实现获取服务端信息的实时数据的话可以通过 ajax 轮询、 http1.1 长链接或者 http2 协议实现，但是前面的两种都过于消耗资源了。

## http  与 websocket工作

![1620031905829](websocket/1620031905829.png)

可以看出 http 的工作流程是`建立连接->发送请求->请求响应->断开连接`，每次的请求都会走这个流程，而且客户端在接收到服务端的数据响应之后就会主动断开连接（利用这个特性可借助 http 1.1 实现长连接实现获取后端实时数据——发送请求后，服务端不立即响应，而是等有新数据是才将新数据响应回去，客户端接收都响应之后关闭旧连接后，从新发送请求建立长连接 ）。

websocket 的工作流程是 `建立连接->前后端通过消息进行通信->关闭连接`。

## websocket 的请求头和响应头

![1620032431678](websocket/1620032431678.png)

### **请求头中的信息：**

+ `Upgrade:websocket`：告知服务器通信协议已发生改变：我要发起的是websocket协议。以达到握手的目的。
+ `Sec-WebSocket-Version:13`:告知服务器使用websocket版本
+ `Sec-WebSocket-Extensions:permessage-deflate`:
+ `Sec-WebSocket-Key:xxxxxxxx`:字段记录着握手必不可少的键值，用于验证服务器是否支持websocket通信。
+ `Sec-WebSocket-Protocol:`:字段记录的是所需要使用的协议。

### **响应头中的信息：**

+ `connection:Upgrade`:客户端即将升级的协议是Websocket协议
+ `Sec-WebSocket-Accept:xxxxxxxx`:字段值是由握手请求中的Sec-WebSocket-Key字段值加密过后生成的。

websocket 需要类似 TCP 的客户端和服务器端通过握手连接，连接成功后才能相互通信，客户端可服务端都能主动发送数据给对方。



## 创建webSocket连接

websocket 对象提供了一组 API ，用于创建和管理 WebSocket连接，以及通过连接发送和接收数据。

### **创建 websocket**

```js
let ws = new WebSocket(url[,protocols]);
```

> **url**：表示要连接的URL，这个URL应该为响应 WebSocket 的地址
>
> **protocols**：可以是一个单个的协议名字字符串或者包含多个协议名字的字符串的数组

### 属性以及方法

`close([code][,reson])`关闭 WebSocket 连接或者停止正在进行的连接请求。

> + `code`：一个数字值，表示关闭连接的状态号，表示连接被关闭的原因
>
> + `reson`：一个可读字符串，表示连接被关闭的原因

`send(data)`通过WebSocket 连接向服务器发送数据。

> `data`：用于传输至服务器的数据，可以是 `USVString`、 `ArrayBuffer`、 `Blob`、 `ArrayBufferView`。

`onclose`：用于监听连接关闭事件，当 websocket 对象的 readyyState 状态变化为 CLOSE 时会触发该事件。会接收一个`close Event` 对象

`onerror`：当错误发生时用于监听 error 事件的事件监听器，会接收一个 error event 对象

`onmessage`：一个用于消息事件的事件监听器，这一事件当有消息达到的事件该事件会触发，会接收一个 message event 对象

`onopen`：一个用于监听打开事件的事件监听器，当 readyState 的值变为 OPEN 时触发，接收一个 open event 对象

`readyState`：连接状态。`0:连接还没有开启，1：连接已开启并准备好进行通信，2：连接正在关闭过程中， 3：连接已经关闭或者连接无法建立`。



## 后端 websocket 实现原理

[Nodejs教程20：WebSocket之二：用原生实现WebSocket应用](https://blog.csdn.net/chencl1986/article/details/88411056)

[WebSocket 学习--用nodejs搭建服务器](https://www.cnblogs.com/fps2tao/p/7875618.html)

[使用nodejs实现服务端websocket通讯](https://zhuanlan.zhihu.com/p/127889084)

[nodejs的websocket的服务器端是如何实现的？](https://www.zhihu.com/question/37647173/answer/1520244048)

[nodejs实现Websocket的数据接收发送](https://www.cnblogs.com/axes/p/4514199.html)



## 使用 websocket 实现服务

在 nodejs 中没有现成的 websocket 模块可以使用，但是可以使用其他人大牛写好的 websocket 库，避免重新造轮子。一些好用的 websocket 有 `socket.io`，`websocket`，`ws`等等，并且一般的 websocket 的库都提供了 前端版本的接口和后端版本的接口。这里使用 `websocket`库进行搭建 websocket 服务器

安装

```shell
npm install websocket
```

后端创建服务器

```js
//后端的websocket接口
var WebSocketServer = require('websocket').server;
//需要使用 http 模块进行连接通信
var http = require('http');

//全部客户端链接
var clients = [];

var server = http.createServer(function(request, response) {
    console.log((new Date()) + ' 收到请求来自：' + request.url);
    response.writeHead(404);
    response.end();
});
server.listen(3000, function() {
    console.log((new Date()) + '服务已开启于 3000 端口');
});

//搭建websocket 服务
wsServer = new WebSocketServer({
    httpServer: server,
});

//判断当前连接是否被允许
function originIsAllowed(origin) {
  return true;
}

wsServer.on('request', function(request) {
    if (!originIsAllowed(request.origin)) {
      request.reject();
      console.log((new Date()) + ' 这个连接被拒绝 ' + request.origin);
      return;
    }
    
    //得到当前连接
    var connection = request.accept(null, request.origin);
   	
    //将连接存入到 clients 中 
    clients.push(connection);
    
    //var connection = request.accept('echo-protocol', request.origin);//如果第一个参数为 'echo-protocol' 那么建立连接时也要加上这个 new WebSocket("ws://localhost:3000","echo-protocol");
    console.log((new Date()) + ' Connection accepted.');
    //监听前端发过来的 message 消息
    connection.on('message', function(message) {
        if (message.type === 'utf8') {
            console.log('Received Message: ' + message.utf8Data);
            //connection.sendUTF(message.utf8Data);
          	
            //将连接发送个所有的客户端
            clients.forEach(conn=>{
                conn.sendUTF(message.utf8Data);
            })
        }
        else if (message.type === 'binary') {
            console.log('Received Binary Message of ' + message.binaryData.length + ' bytes');
            connection.sendBytes(message.binaryData);
        }
    });
    // 监听连接关闭
    connection.on('close', function(reasonCode, description) {
        console.log((new Date()) + ' Peer ' + connection.remoteAddress + ' disconnected.');
        //将连接移除
        let index  = clients.indexOf(connection);
        clients.splice(index,1);
    });
});
```

前端连接

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        let ws = new WebSocket("ws://localhost:3000");
        ws.onopen = function(res){
            console.log("连接成功",res);
        }
        ws.onmessage = function(res){
            console.log("新消息",res);
        }
        ws.onclose = function(res){
            console.log("连接断开",res);
        }
    </script>
</body>
</html>
```



## 参考资料

[Http、Socket、WebSocket之间联系与区别](https://www.cnblogs.com/aspirant/p/11334957.html)

[WebSocket协议及优点](https://www.pianshen.com/article/3238281810/)

[WebSocket 协议有哪些缺点](https://www.zhihu.com/question/20155314)

[nodejs实现Websocket的数据接收发送](https://www.cnblogs.com/axes/p/4514199.html)

[用原生实现WebSocket应用](https://blog.csdn.net/chencl1986/article/details/88411056)

# webSockets实现简易聊天室

1. 安装`socket.io  express`

   ```shell
   npm i socket.io express --save
   ```

2. 构建工程目录

   ```shell
   npm init -y
   ```

3. 新建server.js

   ```js
   const express=require('express');
   const io=reqire('socket.io');
   const path=require('path');
   
   const app=express();
   
   const server=require('http').createServer(app);
   
   //存储连线
   users=[];
   connections=[];
   
   server.listen(process.env.PORT||5000,function(){
     console.log("server is running in : http://localhost:5000")
   });
   
   app.get('/',function(req,res){
     res.sendFile(path.join(__dirname,'/index.html'));
   })
   
   //监听ws连线
   io.sockets.on('connection',function(socket){
    	//将连线存储起来
     connetions.push(socket);
     console.log('连线',connetions);
     
    	//断线处理：将连线去除
     socket.on('disconnect',function(data){
       connections.splice(connections.indexOf(socket),1);
       console.log("user disconnect:%s online",connections.length)
     })
     
     //接收来自客户端的信息后，向客户端发送信息
       socket.on('send message',function(data){
         io.sockets.emit('new message',{msg:data})
       })
     
     // 用户登录
   	socket.on('new user',function(data,callback){
   		if(users.indexOf(data)!=-1){
   			callback(true);
   		}else{
   			callback(true);
   			socket.username=data;
   			users.push(data);
   			io.sockets.emit('get users',users);
   		}
   	})
   })
   ```

   注意：server 和 client 分别是 sockets 和 socket，注意一下。

4. 新建index.html文件

   ```html
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title></title>
   		<link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/4.4.1/css/bootstrap.min.css" rel="stylesheet">
   		<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.0/jquery.min.js"></script>
   		<script src="https://cdn.bootcdn.net/ajax/libs/socket.io/2.3.0/socket.io.js"></script>
   		<style type="text/css">
   			#chatroom{
   				display: none;
   			}
   		</style>
   	</head>
   	<body>
   		<div class="container">
   			<div id="userLogin" class="row">
   				<p id="loginError"></p>
   				<div class="col-md-12">
   					<form id="userForm" action="" method="">
   						<div class="form-group">
   							<input class="form-control" type="text" name="username" id="username" value="enter username" />
   							<input class="btn btn-primary" type="submit" name="loginBtn" id="loginBtn" value="Login" />
   						</div>
   					</form>
   				</div>
   			</div>
   			
   			<div id="chatroom" class="row">
   				<div class="col-md-4">
   					<div class="jumbotron">
   						<h1>Users</h1>
   						<ul class="list-group" id="users">
   							
   						</ul>
   					</div>
   				</div>
   				<div class="col-md-8">
   					<div class="chat" id="chat">
   					</div>
   					<form action="" id="mseeageForm">
   						<div class="form-group">
   							<textarea class="form-control" id="message" placeholder="请输入内容">
   							</textarea>
   							<input type="submit" name="btn" id="btn" class="btn btn-primary" value="ENTER" />
   						</div>
   					</form>
   				</div>
   			</div>
   		</div>
   		<script type="text/javascript">
   			$(function(){
   				const socket=io.connect();
   				const messageForm=$('#mseeageForm');
   				const message=$('#message');
   				const chat=$('#chat');
   				
   				const users=$('#users')
   				const loginError=$('#loginError');
   				const userLogin=$('#userLogin');
   				const userForm=$('#userForm');
   				const username=$('#username');
   				const chatroom=$('#chatroom');
   				
   				userForm.submit(function(e){
   					e.preventDefault();
   					// 登录
   					socket.emit('new user',username.val(),function(data){
   						if(data){
   							userLogin.hide();
   							chatroom.show();
   						}else{
   							loginError.text('login error')
   						}
   					});
   					username.val('');
   				})
   				// 登录成功后接收数据
   				socket.on('get users',function(data){
   					data.forEach(item=>{
   						users.append(`<li class="list-group-item">${item}</li>`)
   					})
   				})
   				
   				messageForm.submit(function(e){
   					e.preventDefault();
   					// 发送数据
   					socket.emit('send message',message.val());
   					message.val('');
   				})
   				// 接收数据
   				socket.on('new message',function({msg,username}){
   					chat.append(`<p><b>${username}：</b>${msg}</p>`)
   				})
   			})
   		</script>
   	</body>
   </html>
   ```

   