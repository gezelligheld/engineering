RFC 规定 HTTP 的状态码为三位数，被分为五类

#### 1\*\*

1 开头的状态码表示服务器收到请求，需要请求者继续执行操作

- 100（Continue） 客户端应继续该请求

- 101（Switching Protocols） 服务器根据客户端的请求切换协议。在 HTTP 升级为 WebSocket 的时候，如果服务器同意变更，就会发送状态码 101

#### 2\*\*

2 开头的状态码表示操作被成功接收并处理

- 200（OK） 请求成功

- 201（Created） 成功请求并创建了新的资源

- 202（Accepted） 已经接受请求，但未处理完成

- 203（Non-Authoritative Information）成功请求，但信息未授权

- 204（No Content）含义与 200 相同，但没有响应体数据

- 206（Partial Content）部分内容，HTTP 分块下载和断点续传时使用，并带上相应的响应头字段 Content-Range

#### 3\*\*

3 开头的状态码表示重定向，需要进一步的操作以完成请求

- 301（Moved Permanently） 永久重定向，请求的资源已被永久的移动到新 url。比如你的网站从 HTTP 升级到了 HTTPS 了，以前的站点再也不用了，应当返回 301

- 302（Found） 临时重定向，客户端应继续使用原有 url。比如你的网站从 HTTP 升级到了 HTTPS 了，以前的站点暂时不用，应当返回 302

- 304（Not Modified） 协商缓存命中，服务器返回此状态码时不会返回任何资源，客户端可以使用已缓存的资源

#### 4\*\*

4 开头的状态码表示客户端错误

- 400（Bad Request） 客户端请求的语法错误，服务器无法理解

- 401（Unauthorized） 请求要求用户的身份认证

- 403（Forbidden） 服务器禁止访问

- 404（Not Found） 服务器无法根据客户端的请求找到资源

- 405（Method Not Allowed） 请求方法不被服务器端允许

- 406（Not Acceptable） 资源无法满足客户端的条件

- 407（Proxy Authentication Required） 请求要求代理的身份认证

- 408（Request Timeout） 服务器等待了太长时间

- 413（Request Entity Too Large） 请求体的数据过大

- 414（Request-URI Too Long） 请求行里的 URI 太大

#### 5\*\*

5 开头的状态码表示服务端错误

- 500（Internal Server Error） 服务器内部错误，无法完成请求

- 501（Not Implemented） 服务器不支持请求的功能，无法完成请求

- 502（Bad Gateway） 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应

- 503（Service Unavailable） 服务器暂时的无法处理客户端的请求

- 504（Gateway Time-out） 充当网关或代理的服务器，未及时从远端服务器获取请求

- 505（HTTP Version not supported） 服务器不支持请求的 HTTP 协议的版本，无法完成处理
