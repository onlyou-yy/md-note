## webRTC 是什么

**WebRTC** (Web Real-Time Communications) 是一项**实时通讯技术**，它允许网络应用或者站点，在**不借助中间媒介**的情况下，建立**浏览器之间点对点**（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。也就是说它可以让浏览器之间直接进行数据交互而不需要借助服务器来进行数据通信。

以往客户端进行通信大多需要借助服务器来做数据传递的工作，这样也就必然会导致数据传递的速度比 peer-to-peer 的模式慢，因为p2p这种模式下也就不需要服务器进行数据传递，我们客户端之间就能直接通信，这就少走了一步路，使得通信的速度比`B/S`，`C/S`的模式快。

## webRTC 的能力

WebRTC 是一个音视频处理+ 即时通讯的开源库，它是一个非常优秀的多媒体框架，而且还跨平台。在音视频领域有两个泰山北斗级别的开源库，一个是FFmpeg， 另一个就是WebRTC 。他们拥有各自不同的侧重点，优势。FFmepg的侧重点是多媒体的编辑，音视频的编解码等对视频文件的处理。 而对于WebRTC, 它的优势是整个网络中实现音视频的传输，对网络的抖动，丢包，网络的评估，在网络层面的各自算法优化保证了音视频传输的稳定，此外它还可以对网络传输经常发生的回音等问题优化处理，如回音消除，降噪。虽然WebRTC最初被设想为纯粹的P2P技术，但许多日常业务应用程序需要集中式媒体功能，通过P2S（peer-to-server）[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)提高可靠性、效率或扩展性。

总的来说，WebRTC能做下面这些事情：

- 音视频实时互动
- 游戏，即时通讯，文件传输等等
- 它是一个百宝箱，传输，音视频处理（回音消除，降噪）



## webRTC 前端使用关键模块

面向第三方开发者的WebRTC标准API（Javascript），使开发者能够容易地开发出类似于网络视频聊天的web应用，这些API可分成`Network Stream API`、 `RTCPeerConnection`、`Peer-to-peer Data API`三类。

**Network Stream API**

> MediaStream：用来表示一个媒体数据流。
> MediaStreamTrack: 在浏览器中表示一个媒体源。

对于浏览器而言，可以通过`navigator.mediaDevices.getDisplayMedia({video:true,audio});`来获得媒体流数据，这些数据将会被用来喝另一端的浏览器做通信。

**RTCPeerConnection**

> RTCPeerConnection：一个RTCPeerConnection对象允许用户在两个浏览器之间直接通讯。
> RTCIceCandidate ：表示一个ICE协议的候选者。
> RTCIceServer：表示一个ICE Server。

**Peer-to-peer Data API**

> DataChannel：数据通道( DataChannel)接口表示一个在两个节点之间的双向的数据通道 。



## WebRTC 协议

webRTC协议由几个核心部件组成，分别为`ICE、STUN、NAT、TURN、SDP`。

### ICE

交互式连接设施[Interactive Connectivity Establishment (ICE)](http://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment) 是一个允许你的浏览器和对端浏览器建立连接的协议框架。在实际的网络当中，有很多原因能导致简单的从A端到B端直连不能如愿完成。这需要绕过阻止建立连接的防火墙，给你的设备分配一个唯一可见的地址（通常情况下我们的大部分设备没有一个固定的公网地址），如果路由器不允许主机直连，还得通过一台服务器转发数据。ICE通过使用以下几种技术完成上述工作。

### STUN

NAT的会话穿越功能[Session Traversal Utilities for NAT (STUN)](http://en.wikipedia.org/wiki/STUN) (缩略语的最后一个字母是NAT的首字母)是一个允许位于NAT后的客户端找出自己的公网地址，判断出路由器阻止直连的限制方法的协议。

客户端通过给公网的STUN服务器发送请求获得自己的公网地址信息，以及是否能够被（穿过路由器）访问。

![An interaction between two users of a WebRTC application involving a STUN server.](https://mdn.mozillademos.org/files/6115/webrtc-stun.png)

### NAT

网络地址转换协议[Network Address Translation (NAT)](http://en.wikipedia.org/wiki/NAT) 用来给你的（私网）设备映射一个公网的IP地址的协议。一般情况下，路由器的WAN口有一个公网IP，所有连接这个路由器LAN口的设备会分配一个私有网段的IP地址（例如192.168.1.3）。私网设备的IP被映射成路由器的公网IP和唯一的端口，通过这种方式不需要为每一个私网设备分配不同的公网IP，但是依然能被外网设备发现。

一些路由器严格地限定了部分私网设备的对外连接。这种情况下，即使STUN服务器识别了该私网设备的公网IP和端口的映射，依然无法和这个私网设备建立连接。这种情况下就需要转向TURN协议。

> 在访问外网资源的时候，一般会通过网关的路由表，这个表保存着多条数据，每条数据又包括`在本地局域网内的ip地址和端口号，映射到外网的ip和端口号，要访问的目标的ip和端口号`，其中将本地局域网地址映射到外网地址的工作就由NAT来完成。
>
> NAT 分为主要有四种，分别为
>
> + 对称NAT：对每个端都有不同的口令进行限制，仅允许符合条件的穿透
> + 完全锥形NAT：允许任意地址进行穿透
> + 端口限制锥形NAT：只允许对应ip+端口进行穿透
> + IP限制锥形NAT：只允许对应ip进行穿透



### TURN

一些路由器使用一种“对称型NAT”的NAT模型。这意味着路由器只接受和对端先前建立的连接（就是下一次请求建立新的连接映射）。

NAT的中继穿越方式[Traversal Using Relays around NAT (TURN)](http://en.wikipedia.org/wiki/TURN) 通过TURN服务器中继所有数据的方式来绕过“对称型NAT”。你需要在TURN服务器上创建一个连接，然后告诉所有对端设备发包到服务器上，TURN服务器再把包转发给你。很显然这种方式是开销很大的，所以只有在没得选择的情况下采用。

![An interaction between two users of a WebRTC application involving STUN and TURN servers.](https://mdn.mozillademos.org/files/6117/webrtc-turn.png)

### SDP

会话描述协议[Session Description Protocol (SDP)](http://en.wikipedia.org/wiki/Session_Description_Protocol) 是一个描述多媒体连接内容的协议，例如分辨率，格式，编码，加密算法等。所以在数据传输时两端都能够理解彼此的数据。本质上，这些描述内容的元数据并不是媒体流本身。

从技术上讲，SDP并不是一个真正的协议，而是一种数据格式，用于描述在设备之间共享媒体的连接。



### 关系

总的来说，他们之间的关系是：ICE 为服务提供最优的端点连接方案；SDP 主要用来描述端点之间的设备情况和使用的数据解码器，以及端点地址等主要还是设备层面上的描述；STUN 这是用来为端点之间传输 SDP 使端点之间能建立连接；NAT则是让端点的暴露在外网环境下使得端点之间能相互发现对方；TURN 则是在端点之间不能直接建立连接的情况下提供一个备选方案用来传递端点之间的数据。



## webRTC 前端主要API

### 方法

+ `var rtc = new RTCPeerConnection(porp)`，创建一个 `RTCPeerConnection`实例，这个构造函数继承的 EventTarget 对象。实例中有几个比较常用的属性，如`currentLocalDescription/localDescription`当前本地的SDP，`currentRemoteDescription`当前连接的端点的SDP，`connectionState`当前连接状态等。prop 可以不传，如果你有ice服务的话可以通过`prop.iceServers = [{urls:'xxxx'}]`设置多个ice服务器。
+ `rtc.createrOffer(sback,fback)`，创建一个offer（offer是一个RTCSessionDescription对象，有两个属性，type和sdp），接收两个参数成功回调和失败回调，需要主要的可以通过回调函数返回，也可以通过promise 的方法返回
+ `rtc.createAnswer(sback,fback)`，这个方法和`createOffer`是几乎一样的，不过这个方法是根据远端发来的 offer 来生成 answer （answer是一个RTCSessionDescription对象，有两个属性，type和sdp））的，所以在创建之前需要先通过`setRemoteDescription`方法根据offer建立为连接的远程端描述。
+ `rtc.setLocalDescription(sdp,sback,fback)`，改变与连接相关的本地描述(设置本地的sdp)，sdp可以是offer/answer中的sdp。同样是异步的函数，可以使用回调和promise.
+ `rtc.setRemoteDescription(sdp,sback,fback)`，改变与连接相关的远端描述(设置要连接方的sdp)，sdp一般是offer中的sdp，最好通过`new RTCSessionDescription(offer.sdp)`生成一个新的sdp。同样是异步的函数，可以使用回调和promise.
+ `rtc.addIceCandidate(ice)`，添加候选的ice方案
+ `rtc.getLocalStreams()`，返回连接的本地媒体流数组。这个数组可能是空数组。
+ `rtc.getRemoteStreams()`，返回连接的远端媒体流数组。这个数组可能是空数组。
+ `rtc.addTrack(track,streams)`，将一个新的媒体音轨添加到一组音轨中，这些音轨将被传输给另一个对等点，同时返回一个标识 sender。可以通过`let t = await navigator.mediaDevices.getUserMedia({video: true, audio: true})`得到对象后通过`t.getTracks()`获取到轨道。
+ `rtc.removeTrack(sender)`，通过 addTrack 返回 sender 来移除 track。
+ `rtc.close()`关闭一个RTCPeerConnection实例所调用的方法。
+ `var dc = rtc.createDataChannel(label,options)`，创建一个可以发送任意数据的数据通道(data channel)，label 是通道的名字，且需要进行协商。常用于后台传输内容, 例如: 图像, 文件传输, 聊天文字, 游戏数据更新包, 等等。返回一个 dataChannel 对象，这个对象可以通过`dc.send(data)`来发送数据。
  + `dc.onopen`处理建立连接
  + `dc.onmessage`接收消息事件。
  + `dc.onclose`关闭通道事件。
  + `dc.onerror`出错。
  + `dc.readyState`通道状态，`"connecting"` 该状态表示底层链路还未建立和激活；`"open"` 该状态表示底层链路已经连接成功并且运行；`"closing"` 该状态表示底层链路已经在关闭的过程中；`"closed"` 该状态表示底层链路已经完全被关闭
  + `dc.bufferedAmount`表示缓冲队列中等待发送的字节数。
  + `dc.binaryType`表示由链路发送的二进制数据的类型。该项的值应该为`"blob"`或者`"arraybuffer"`，默认值为`"blob"`。
  + `dc.send()`发送数据
  + `dc.close()`关闭通道



### 事件

+ `rtc.onicecandidate`当添加IceCandidate时触发
+ `rtc.oniceconnectionstatechange`当iceConnectionState改变时触发
+ `rtc.onnegotiationneeded`当需要进行协商时触发，比如创建数据通道`createDataChannel`时
+ `rtc.ondatachannel`当一个 RTCDataChannel 被添加到连接时，这个事件被触发。切回调的参数也是一个dataChannel
+ `rtc.ontrack`当添加track时触发
+ `rtc.onremovetrack`当移除track时触发



## 兼容性处理

目前 webRTC 的大多数接口都还是实验性阶段，所以需要做还兼通性处理，可以使用 [Adapter.js](https://github.com/webrtcHacks/adapter) 做兼容性 pollyfill。webRTC 的相关方法还是比较难用的，而且流程比较复杂，可以使用 peerjs 来简化操作。 



## 简单端点数据通信

实现两个浏览器标签页下的数据通信，只需要简单的几步就好。

在标签页一中写入如下代码

```js
let rtc1 = new RTCPeerConnection();
let dc = rtc1.createDataChannel('dc');
dc.onopen = e => console.log('数据通道建立成功')
dc.onmessage = e => console.log('接收到数据',e.data);
rtc1.onicecandidate = e => console.log('ice接点变化了,sdp:',rtc1.localDescription);
```

在标签页二中写入如下代码

```js
let rtc2 = new RTCPeerConnection();
rtc2.onicecandidate = e => console.log('ice接点变化了,sdp:',rtc2.localDescription);
rtc2.ondatachannel = e => {
  rtc2.dc = e.channel;
  rtc2.dc.onopen = e => console.log('数据通道建立成功');
  rtc2.dc.onmessage = e => console.log('接收到数据',e.data);
}
```

由于没有建立传输 offer/answer的服务器，所以我们需要手动将 offer 复制给到标签页二，将 answer 复杂到标签页一。

标签页一生成offer

```js
rtc1.createOffer().then(offer => rtc1.setLocalDescription(offer))
```

标签页二根据标签页一的offer生成answer

```js
rtc2.setRemoteDescription(offer).then(e => rtc2.createAnswer()).then(answer => rtc2.setLocalDescription(answer))
```

将标签二产生的answer 复制给标签页一，以此来建立连接

```js
rtc1.setRemoteDescription(answer)
```

之后我们就可以使用`send`方法发送数据了

```js
dc.send('hello world')
//-------------------------------------------
rtc2.dc.send('你好')
```





## nodejs配置https

```js
let express = require("express");
let http = require("http");
let https = require("https");
let fs = require("fs");
// Configuare https
const httpsOption = {
    key : fs.readFileSync("./https/xxxxxxxxxxxx.key"),
    cert: fs.readFileSync("./https/xxxxxxxxxxxx.pem")
}
// Create service
let app = express();
http.createServer(app).listen(80);
https.createServer(httpsOption, app).listen(443);
```



## 参考

[webRTC专题-晓果博客](https://blog.csdn.net/huangxiaoguo1/category_9705009.html)

[webRTC专题-极客雨露](https://blog.csdn.net/kyl282889543/category_9327113_2.html)

[WebRTC SDP协议](https://www.cnblogs.com/chyingp/p/sdp-in-webrtc.html)

[WebRTC ICE介绍](https://zhuanlan.zhihu.com/p/351105085)

[webrtc中ICE介绍](https://blog.csdn.net/u013692429/article/details/106529213)

[WebRTC 之ICE浅谈](https://zhuanlan.zhihu.com/p/60684464)

[你肯定不理解的技术，网络穿透，P2P，打洞的核心原理](https://www.bilibili.com/video/BV1Ey4y1z7JK)

[NAT的四种分类：全锥形NAT,地址受限锥形NAT,端口受限锥形NAT,对称NAT](https://blog.csdn.net/s2603898260/article/details/118755474)