http 明文传输，在传输过程中任何人都有可能从中截获、修改或者伪造请求发送，是不安全的。HTTPS 是 HTTP 协议的一种扩展，使用传输层安全性(TLS)或安全套接字层(SSL)对通信协议进行加密，也就是 HTTP + SSL(TLS) = HTTPS

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/14/170d6eaf0abec4e7~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

HTTPS 协议其实非常简单，默认端口号 443，应答模式、报文结构、请求方法、URI、头字段、连接管理等等都完全沿用 HTTP

#### TLS 和 SSL

SSL（安全套接字层），在 OSI 七层模型中处于会话层(第 5 层)，当它发展到第三个大版本的时候才被标准化成为 TLS，也就是 TLS1.0 = SSL3.1

#### TLS 1.2 握手过程

#### TLS1.3

##### 对称加密

对称加密是指加密和解密时使用的密钥都是同样的密钥，需要保证密钥足够安全，但加密解密双方在传输密钥的过程中可能会泄漏，是不安全的

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/14/170d6eaf178d5797~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

##### 非对称加密

##### https 通信过程
