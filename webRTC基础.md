## webRTC 是什么

**WebRTC** (Web Real-Time Communications) 是一项**实时通讯技术**，它允许网络应用或者站点，在**不借助中间媒介**的情况下，建立浏**览器之间点对点**（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。也就是说它可以让浏览器之间直接进行数据交互而不需要借助服务器开进行数据通信。

## webRTC 的能力

WebRTC 是一个音视频处理+ 即时通讯的开源库，它是一个非常优秀的多媒体框架，而且还跨平台。在音视频领域有两个泰山北斗级别的开源库，一个是FFmpeg， 另一个就是WebRTC 。他们拥有各自不同的侧重点，优势。FFmepg的侧重点是多媒体的编辑，音视频的编解码等对视频文件的处理。 而对于WebRTC, 它的优势是整个网络中实现音视频的传输，对网络的抖动，丢包，网络的评估，在网络层面的各自算法优化保证了音视频传输的稳定，此外它还可以对网络传输经常发生的回音等问题优化处理，如回音消除，降噪。虽然WebRTC最初被设想为纯粹的P2P技术，但许多日常业务应用程序需要集中式媒体功能，通过P2S（peer-to-server）[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)提高可靠性、效率或扩展性。

总的来说，WebRTC能做下面这些事情：

- 音视频实时互动
- 游戏，即时通讯，文件传输等等
- 它是一个百宝箱，传输，音视频处理（回音消除，降噪）



## webRTC 前端使用核心模块

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

### TURN

一些路由器使用一种“对称型NAT”的NAT模型。这意味着路由器只接受和对端先前建立的连接（就是下一次请求建立新的连接映射）。

NAT的中继穿越方式[Traversal Using Relays around NAT (TURN)](http://en.wikipedia.org/wiki/TURN) 通过TURN服务器中继所有数据的方式来绕过“对称型NAT”。你需要在TURN服务器上创建一个连接，然后告诉所有对端设备发包到服务器上，TURN服务器再把包转发给你。很显然这种方式是开销很大的，所以只有在没得选择的情况下采用。

![An interaction between two users of a WebRTC application involving STUN and TURN servers.](https://mdn.mozillademos.org/files/6117/webrtc-turn.png)

### SDP

会话描述协议[Session Description Protocol (SDP)](http://en.wikipedia.org/wiki/Session_Description_Protocol) 是一个描述多媒体连接内容的协议，例如分辨率，格式，编码，加密算法等。所以在数据传输时两端都能够理解彼此的数据。本质上，这些描述内容的元数据并不是媒体流本身。

从技术上讲，SDP并不是一个真正的协议，而是一种数据格式，用于描述在设备之间共享媒体的连接。

记录SDP远远超出了本文档的范围。但是，这里有几件事值得注意。 



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