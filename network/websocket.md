#### 为什么需要websocket

http协议下通信只能由客户端发起，服务端响应，如果服务端有连续的状态变化，有下面两种方法获取

- ajax轮询，定时发送http请求以获取最新的服务端状态变化

- Long Polling长轮询，保持长链接，等服务端有消息时返回，然后断开，周而复始

这两种方式开销比较大，而websocket一次握手，持久连接，服务端可以主动推送消息给客户端，有如下特点

- 建立在tcp协议之上，握手阶段采用http协议

- 默认端口号与http相同，为80和443

- 没有同源限制，客户端可以与任意服务端通信

- 协议标识符为ws，如果加密则为wss

- 传输的数据格式轻量，开销小，

#### 连接过程

1. 客户端发起HTTP握手，告诉服务端进行WebSocket协议通讯，并告知WebSocket协议版本

2. 服务端确认协议版本，升级为WebSocket协议。之后如果有数据需要推送，会主动推送给客户端

升级为websocket协议后的request header如下

```js
Accept-Encoding: gzip, deflate, br
Accept-Language: zh,zh-TW;q=0.9,en-US;q=0.8,en;q=0.7,zh-CN;q=0.6
Cache-Control: no-cache
// 要升级的协议
Connection: Upgrade
Host: 127.0.0.1:3000
Origin: http://localhost:3000
Pragma: no-cache
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
// 对应服务端响应头的Sec-WebSocket-Accept，一一对应，避免多余的连接
Sec-WebSocket-Key: bwb9SFiJONXhQ/A4pLaXIg==
// websocket的版本
Sec-WebSocket-Version: 13
// 要升级协议到websocket协议
Upgrade: websocket
```

升级为websocket协议后的response header如下

```js
Connection: Upgrade
Sec-WebSocket-Accept: 2jrbCWSCPlzPtxarlGTp4Y8XD20=
Upgrade: websocket
```

通过websocket实例可以拿到对应的连接状态，0（正在连接）-> 1（连接成功）-> 2（正在关闭）-> 3（已经关闭）

```js
new Websocket('ws://example.com:8080').readyState;
```