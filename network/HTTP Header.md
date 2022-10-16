#### 常见请求头

请求头主要用来传递客户端自身信息以及期望的响应形式

```less
// 标记报文的内容类型
Accept : text/plain, text/html
// 指定浏览器可以支持的 web 服务器返回内容压缩编码类型
Accept-Encoding : compress, gzip
// 浏览器可接受的语言
Accept-Language : en,zh
// 可以请求网页实体的一个或者多个子范围字段
Accept-Ranges : bytes
// HTTP 授权的授权证书
Authorization : Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
// 指定请求和响应遵循的缓存机制
Cache-Control : no-cache
// 表示是否需要持久连接，http1.1 默认持久连接
Connection: keep-alive
// HTTP 请求发送时，会把同域名下的所有 cookie 值一起发送给服务器
Cookie : $Version=1; Skin=new;
// 请求的内容长度
Content-Length : 335
// 请求的与实体对应的内容类型
Content-Type : application/json;charset=UTF-8
// 指定请求的服务器的域名和端口号
Host : www.zcmhi.com
// 只有请求内容与实体相匹配才有效
If-Match : “737060cd8c284d8af7ad3082f209582d”
// 它的值是上一次响应头的 Last-Modified 的值，如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回 304（协商缓存）
If-Modified-Since : Sat, 29 Oct 2010 19 : 43 : 31 GMT
// 它的值是上一次响应头的 ETag 值，与服务器回应的 Etag 比较判断是否改变，如果内容未改变返回 304（协商缓存）
If-None-Match : “737060cd8c284d8af7ad3082f209582d”
// 当前网页的地址，即来路
Referer: https://www.jianshu.com/
// 向服务器指定某种传输协议以便服务器进行转换
Upgrade: websocket
// 发出请求的设备信息
User-Agent : Mozilla/5.0 (Linux; X11)
```

#### 常见响应头

响应头主要用来传递服务端自身信息和返回的响应形式

```less
// 从原始服务器到代理缓存形成的估算时间
Age : 12
// 对某网络资源的有效的请求行为，不允许则返回 405
Allow : GET, HEAD
// 告诉所用的缓存机制
Cache-Control : no-cache
// web 服务器支持的返回内容压缩编码类型
Content-Encoding : gzip
// 响应体的语言
Content-Language : en,zh
// 响应体的长度
Content-Length : 348
// 返回内容的内容类型
Content-Type : application/json
// 请求变量的实体标签的当前值
ETag : “737060cd8c284d8af7ad3082f209582d”
// 响应过期的时间，再次请求时，浏览器会先去对比本地时间和 expires 标明的过期时间，如果没有过期直接从缓存中读取（强缓存）
Expires : Thu, 01 Dec 2010 16 : 00 : 00 GMT
// 请求资源的最后修改时间
Last-Modified : Tue, 15 Nov 2010 12 : 45 : 26 GMT
// 设置 Http Cookie
Set-Cookie : UserID=JohnDoe; Max-Age=3600; Version=1
```

#### Accept 系列字段

- 数据格式

多用途互联网邮件扩展（MIME）首先用在电子邮件系统中，HTTP 从 MIME type 取了一部分来标记报文 body 部分的数据类型

```less
// 请求
Accept: application/json
// 响应
Content-type: application/json
```

- 压缩方式

gzip 是最流行的一种压缩方式，此外还有 deflate、br

```less
// 请求
Accept-Encoding: gzip
// 响应
Content-Encoding: gzip
```

- 支持语言

```less
// 请求
Accept-Language: zh-CN, zh, en
// 响应
Content-Language: zh-CN, zh, en
```

- 字符集

```less
// 请求
Accept-Charset: charset=utf-8
// 响应
Content-Type: text/html; charset=utf-8
```

#### 传输定长和不定长的数据

- 定长：对于定长包体而言，发送端在传输的时候一般会带上 Content-Length, 来指明包体的长度

- 不定长：Transfer-Encoding: chunked 表示分块传输数据，会忽略 Content-Length，基于长连接持续推送动态内容

#### 传输大文件

HTTP 允许客户端只请求资源的一部分，具体请求哪一部分通过 Range 字段指定，服务器收到请求之后，首先验证范围是否合法，如果越界了那么返回 416 错误码，否则读取相应片段，返回 206 状态码

对于单段数据的请求，返回的响应如下

```bash
HTTP/1.1 206 Partial Content
Content-Length: 10
Accept-Ranges: bytes
# 0-9表示请求的返回，100表示资源的总大小
Content-Range: bytes 0-9/100

i am xxxxx
```

对于多段数据的请求，返回的响应如下

```bash
HTTP/1.1 206 Partial Content
# 表示是多段数据请求，响应体中的分隔符是 00000010101
Content-Type: multipart/byteranges; boundary=00000010101
Content-Length: 189
Connection: keep-alive
Accept-Ranges: bytes


--00000010101
Content-Type: text/plain
Content-Range: bytes 0-9/96

i am xxxxx
--00000010101
Content-Type: text/plain
Content-Range: bytes 20-29/96

eex jspy e
--00000010101--
```

#### 处理表单数据的提交

表单提交一般是 POST 请求，主要有两种表单提交的方式

- Content-Type：application/x-www-form-urlencoded

响应的数据会被编码成以&分隔的键值对，字符以 URL 编码方式编码

```
{a: 1, b: 2} -> a=1&b=2 -> a%3D1%26b%3D2
```

- Content-Type：multipart/form-data

响应体如下，数据会分成多个部分。在实际的场景中，对于图片等文件的上传，基本采用 multipart/form-data，因为没有必要做 URL 编码，编码耗时耗空间

```less
// 请求
Content-Type: multipart/form-data;boundary=----WebkitFormBoundaryRRJKeWfHPGrS4LKe
// 响应
Content-Disposition: form-data;name="data1";
Content-Type: text/plain
data1
----WebkitFormBoundaryRRJKeWfHPGrS4LKe
Content-Disposition: form-data;name="data2";
Content-Type: text/plain
data2
----WebkitFormBoundaryRRJKeWfHPGrS4LKe--
```

参考

1. [HTTP 灵魂之问](https://juejin.cn/post/6844904100035821575)
