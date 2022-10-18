网页即时通信（WebRTC）是一个支持网页浏览器进行实时语音对话或视频对话的 API

#### webrtc 架构

![](../assets/webrtc.awebp)

- WebRTC C C++ API (PeerConnection)：负责 P2P 连接、音视频采集、音视频传输、非音视频数据传输等
- Session Management/ Abstract signaling (Session): 会话层，用来管理音视频，非音视频数据传输
- Voice Engine 音频引擎：包括音频编解码器、音频缓冲的 buffer、回音消除、降噪等
- Video Engine 视频引擎：包括视频编解码器、视频 jitter buffer、图像处理等
- Transport 传输模块：包括 SRTP 视频传输协议、流通道、P2P 等，使用 UDP 协议，因为音视频传输对及时性要求更高，允许部分丢帧

#### 通信流程

完成音视频通话需要了解四个模块：音视频采集、STUN/TURN 服务器、信令服务器、端与端之间 P2P 连接

##### 音视频采集

通过 getUserMedia 接口获取媒体数据流

```js
const localVideo = document.querySelector('video');
function gotLocalMediaStream(mediaStream) {
  localVideo.srcObject = mediaStream;
}
navigator.mediaDevices
  .getUserMedia({
    video: {
      width: 640,
      height: 480,
      frameRate: 15,
      facingMode: 'enviroment', // 设置为后置摄像头
      deviceId: deviceId ? { exact: deviceId } : undefined,
    },
    audio: false,
  })
  .then(gotLocalMediaStream)
  .catch((error) => console.log('navigator.getUserMedia error: ', error));
```

##### 通信前的准备工作

- 媒体协商

会话描述信息（SDP）描述了音频编解码器类型、传输协议等，参与视频通讯的双方必须先交换 SDP 信息（SessionDescription），才能知道用什么编码格式对方才能解析

- 网络协商

双方通信前还需要确定网络情况，理想的网络情况是每个浏览器的电脑都是私有公网 IP，可以直接进行点对点连接。但实际中双方会隔着 NAT 设备给获取地址造成了麻烦，WebRTC 通过 ICE 框架确定两端建立网络连接的最佳路径

网络地址转换（ NAT）设置在私网到公网的路由出口位置，组织内部的大量设备通过 NAT 就可以共享一个公网 IP 地址，解决了 IPv4 地址不足的问题

ICE 框架收集本地地址、通过 STUN 服务收集 NAT 外网地址、通过 TURN 收集中继地址来确定主机之间 P2P 最佳传输路径。NAT 会话穿越应用程序（STUN）允许位于 NAT 后的客户端找出自己的公网地址，由客户端发送 STUN 请求、STUN 服务响应告知由 NAT 分配给主机的 IP 地址和端口号。想让内网主机知道它的外网 IP，需要在公网上架设一台 STUN server。TURN 是 STUN 的扩展协议

##### 信令服务器

借助信令服务器交换彼此的媒体协商信息和网络协商信息

![](../assets/single-server.awebp)

##### 端对端的 P2P 连接

呼叫方创建 peerConnection 对象，创建本地会话描述 SDP offer 通过信令服务器传给被呼叫方，被呼叫方接收到后创建一个应答 SDP 会传给呼叫方进行比较。呼叫方通过 STUN server 获取外网网络信息，通过信令服务器发送给被呼叫方，被呼叫方作连通性检测。至此 P2P 连接建立，呼叫方将采集的的媒体数据流发送给被呼叫方

详细连接过程如下

![](../assets/webrtc-connect.awebp)

1. A 向 B 发起 WebRTC 呼叫，创建 peerConnection 对象，在参数中指定 Turn/Stun 的地址

```js
var pcConfig = {
  iceServers: [
    {
      urls: 'turn:stun.al.learningrtc.cn:3478',
      credential: 'mypasswd',
      username: 'garrylea',
    },
    {
      urls: ['stun:stun.example.com', 'stun:stun-1.example.com'],
    },
  ],
};
pc = new RTCPeerConnection(pcConfig);
```

2. A 调用 createOffer 方法创建本地会话描述(SDP offer)，SDP offer 包含有关已附加到 WebRTC 会话，浏览器支持的编解码器和选项的所有 MediaStreamTrack 信息，以及 ICE 代理。A 调用 setLocalDescription 方法将提案设置为本地会话描述，并传递给 ICE 层，之后通过信令服务器将会话描述发送给 B

```js
function sendMessage(roomid, data) {
  if (!socket) {
    console.log('socket is null');
  }
  socket.emit('message', roomid, data);
}

const offer = await pc.createOffer();
await pc.setLocalDescription(offer).catch(handleOfferError);
// 传输发起方本地SDP
sendMessage(roomid, offer);
```

3. A 端 pc.setLocalDescription(offer)创建后，一个 icecandidate 事件就被发送到 RTCPeerConnection，onicecandidate 事件会被触发。B 端接收到一个从远端页面通过信令发来的新的 ICE 候选地址信息，本机可以通过调用 RTCPeerConnection.addIceCandidate() 来添加一个 ICE 代理

```js
//A端
pc.onicecandidate = (event) => {
  if (!event.candidate) return;
  sendMessage(roomid, {
    type: 'candidate',
    label: event.candidate.sdpMLineIndex,
    id: event.candidate.sdpMid,
    candidate: event.candidate.candidate,
  });
};

//B端
socket.onmessage = (e) => {
  if (e.data.hasOwnProperty('type') && e.data.type === 'candidate') {
    var candidate = new RTCIceCandidate({
      sdpMLineIndex: data.label,
      candidate: data.candidate,
    });
    pc.addIceCandidate(candidate)
      .then(() => {
        console.log('Successed to add ice candidate');
      })
      .catch((err) => {
        console.error(err);
      });
  }
};
```

4. A 作为呼叫方获取本地媒体流，调用 addtrack 方法将音视频流流加入 RTCPeerConnection 对象中传输给另一端，加入时另一端触发 ontrack 事件

```js
// A
const pc = new RTCPeerConnection();
stream.getTracks().forEach((track) => {
  pc.addTrack(track, stream);
});
// B
const remoteVideo = document.querySelector('#remote-video');
pc.ontrack = (e) => {
  if (e && e.streams) {
    message.log('收到对方音频/视频流数据...');
    remoteVideo.srcObject = e.streams[0];
  }
};
```

5. B 调用 createAnswer 方法创建应答，调用 setLocalDeacription 方法应答设置为本地会话并传递给 ICE 层

```js
socket.onmessage = e => {
    message.log("接收到发送方SDP");
    await pc.setRemoteDescription(new RTCSessionDescription(e.data));
    message.log("创建接收方（应答）SDP");
    const answer = await pc.createAnswer();
    message.log(`传输接收方（应答）SDP`);
    sendMessage(roomid, answer);
    await pc.setLocalDescription(answer);
}
```

6. AB 都有了自己和对方的 SDP，媒体交换方面达成一致，收集的 ICE 完成连通性检测建立最连接方式，P2P 连接建立

此外 RTCPeerConnection API 可以建立点对点 P2P 互联，不需要中介服务器，延迟更低，只能传输非音视频数据

```js
const dc = pc.createDataChannel('chat');
dc.onmessage = () => {
  // todo
};
dc.onopen = function () {
  console.log('datachannel open');
};

dc.onclose = function () {
  console.log('datachannel close');
};

// 创建数据通道时触发
pc.ondatachannel = (e) => {
  if (!dc) {
    dc = e.channel;
    dc.onmessage = receivemsg;
    dc.onopen = dataChannelStateChange;
    dc.opclose = dataChannelStateChange;
  }
};
```

参考

1. [浅聊 WebRTC 视频通话](https://juejin.cn/post/7000205126719766565)
2. [WebRTC 中文文档](https://github.com/RTC-Developer/WebRTC-Documentation-in-Chinese)
