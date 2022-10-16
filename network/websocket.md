#### 为什么需要 websocket

http 协议下通信只能由客户端发起，服务端响应，如果服务端有连续的状态变化，有下面两种方法获取

- 短轮询，定时发送 http 请求以获取最新的服务端状态变化

- 长轮询，服务端收到请求后不会立刻响应，等服务端有消息时返回，然后再由客户端重新发起这个流程

这两种方式开销比较大，而 websocket 一次握手，持久连接，服务端可以主动推送消息给客户端，有如下特点

- 建立在 tcp 协议之上，握手阶段采用 http 协议

- 默认端口号与 http 相同，为 80 和 443

- 没有同源限制，客户端可以与任意服务端通信

- 协议标识符为 ws，如果加密则为 wss

- 传输的数据格式轻量，开销小，

#### 连接过程

1. 首先客户端发起请求，告诉服务端进行 WebSocket 协议通讯，并告知 WebSocket 协议版本，请求头如下

```less
GET / HTTP/1.1
Host: localhost:8080
Origin: http://127.0.0.1:3000
// 表示要升级协议
Connection: Upgrade
// 表示要升级到websocket协议
Upgrade: websocket
// 表示websocket的版本
Sec-WebSocket-Version: 13
// 与后面服务端响应头的Sec-WebSocket-Accept是配套的，提供基本的防护
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==
```

2. 服务端确认协议版本，升级为 WebSocket 协议，返回状态码 101 表示协议切换，之后如果有数据需要推送，会主动推送给客户端，响应头如下

```less
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

其中 Sec-WebSocket-Accept 的值是根据 Sec-WebSocket-Key 然后通过以下方式计算的

```js
const crypto = require('crypto');
const magic = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
const secWebSocketKey = 'w4v7O6xFTi36lq3RNcgctw==';

const secWebSocketAccept = crypto
  .createHash('sha1')
  .update(secWebSocketKey + magic)
  .digest('base64');

console.log(secWebSocketAccept); // Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

通过 websocket 实例可以拿到对应的连接状态，0（正在连接）-> 1（连接成功）-> 2（正在关闭）-> 3（已经关闭）

```js
new Websocket('ws://example.com:8080').readyState;
```

#### 数据帧格式

WebSocket 的每条消息可能被切分成多个数据帧。当 WebSocket 的接收方收到一个数据帧时，会根据 FIN 的值来判断，是否已经收到消息的最后一个数据帧；根据 opcode 来区分操作的类型。比如 0x8 表示断开连接，0x0-0x2 表示数据交互。数据帧格式如下

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
```

数据分片示例如下

```less
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```

#### 心跳

长时间没有数据往来，WebSocket 会自动断开，避免造成网络资源的浪费，但某些场景需要保持连接，这时候需要发送方发 ping、接收方发 pong，对应的是 WebSocket 的两个控制帧，opcode 分别是 0x9、0xA。这就是所谓的心跳

参考

1. [WebSocket：5 分钟从入门到精通](https://juejin.cn/post/6844903544978407431)
