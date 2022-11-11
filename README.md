前端工程化，包括打包、编译等

#### 知识点

##### babel

babel 是一个 js 代码转换器，可以将 es6+的代码编译成 es5，使程序能够在低版本浏览器中运行

- plugin：babel 将转化功能拆解到多个 plugin 中，plugin 负责对 let、for of 等语法进行 polyfill。首先将源码转化为代码碎片 tokens，再转化为 ast，遍历 ast 将其修改为实际需要的 ast，最后再生成代码。其中遍历 ast 将其修改为实际需要的 ast 这部分就是可以自定义的部分，自定义 plugin 实际上是一个返回值带有 visitor 对象的函数

- presets：一个 presets 预设了一组 plugin，如@babel/preset-env、@babel/preset-react 等，@babel/preset-env 可以根据开发者配置的 browserlist 最小化的引入 plugin 和 core-js

- core-js：通过原型属性或全局注入方法来进行 polyfill

##### webpack

- 构建流程：Webpack 采用了 bundle 机制，将项目中各种类型的源文件转化供浏览器识别的 js、css、img 等文件，建立源文件之间的依赖关系合并为少数几个 bundle，bundle 机制的工作流程主要分为两步，首先构建模块依赖图，获取入口文件，使用对应 loader 将其转换为浏览器能识别的文件，然后解析为 ast 找到相关依赖，直到所有源文件遍历完成，然后分解模块依赖图，作 tree shaking，分解为 initial chunk、async chunks，并优化其中重复的 module，将多个 chunk 共享的 module 分离到 common chunk 中，最后将 chunk 生成文件。文件包括业务代码、第三方代码以及 runtime 和 manifest 数据。浏览器加载打包后的资源时，文件结果已经不复存在，runtime 和 manifest 数据主要用来解析和加载各个模块

- 代码分离：chunk 体积过大过小都不合适，有几种可以拆分 chunk 的方式，包括配置入口文件、动态导入、配置 splitChunks 的 cacheGroups 属性

- 缓存：生成的 chunk 名可以基于文件内容生成 hash，当文件修改时浏览器就会请求新的文件从而使缓存无效，及时将变更更新到浏览器中；配置 runtimeChunk 属性可以将 runtime 代码放到一个单独的 chunk 里，这意味着修改文件重新打包时只有对应的 chunk 和 runtime chunk 会发生改变，其他 chunk 不受影响，可以充分利用缓存减少获取资源；第三方库的代码不会频繁修改，可以配置 entry 或 cacheGroup 使其与业务代码分离，减少向服务端获取资源

- 模块热替换：run server 后启动一个本地服务，浏览器访问资源时作出响应，同时建立 websocket 链接。webpack 会监听源文件的变化，当保存文件时触发 webpack 重新编译，生成一个 hash 和 js 文件，并通过 websocket 将 hash 值给客户端，客户端收到后与上一次作比较，如果不一致则请求 manifest 数据去得知那些 chunk 发生了变更，最后通过插入 script 或 style 标签来实现热更新

- tree shaking：基于 esm，在代码没有运行时就知道哪些模块没有引用，从而移除掉，减小代码体积

- loader：处理文件，使用对应的 loader 转换文件可以其用 import 导入；自定义 loader 是一个接收对应类型文件代码作为参数、返回转换后的代码的函数；常用的有处理样式的 css-loader、style-loader、postcss-loader 等，处理文件的 raw-loader、file-loader 、url-loader 等，用来编译的 babel-loader、ts-loader，用来校验的 eslint-loader

- plugin：用于扩展 webpack 的功能，可以在 webpack 构建过程中广播出来的事件里做更多的事情；自定义 plugin 是一个含有接收 complier 实例作为参数的 apply 方法的类，可以订阅构建过程中抛出来的事件；常用的有构建时定义全局变量的 DefinePlugin、指定某个第三方库不被打包的 IgnorePlugin、压缩 js 的 terser-webpack-plugin、抽离 css 用 link 引入的 mini-css-extract-plugin、可视化分析打包文件体积的 webpack-bundle-analyzer、自动生成 html 的 html-webpack-plugin

##### rollup

rollup 依赖浏览器对 esm 规范的支持，无需注入其他代码，适合不需要兼容低版本浏览器的应用或 js 库，如果需要适应低版本浏览低需要加 polyfill

##### vite

vite 在开发环境采用 no-bundle 机制，无需构建和分解模块依赖图，利用浏览器对 esm 规范的支持去加载文件，在冷启动和热更新方面远远超过 webpack；在生产环境采用 rollup 打包

##### 移动端（自适应布局、其他适配问题）

自适应布局采用 rem 或 vw，本质上都是通过页面的等比缩放实现，以 rem 为例，可以设置 html font-size 为（window.innnerWidth / 设计稿宽度）\* 100，即 1rem=100px 方便计算；其他适配问题如 ios 底部前进后退区域挤压高度的问题、覆盖默认长按问题、下载问题、自动播放媒体问题等

##### server（docker、IIS、nginx）

- nginx & IIS：都可以用来作反向代理的 web 服务器，nginx 还可以做负载均衡；nginx 启动后，在 80 端口启动了一个 socket 服务，用 master 进程读取 nginx 配置文件并管理 worker 进程，每个 worker 进程维护一个线程管理连接，修改配置文件、reload 后会重新生成一个 worker 进程，实现热部署

- docker：软件安装时配置环境是一个比较麻烦的事，虚拟机是一种带环境安装的方案，但它是完整的操作系统，占用资源比较多，而 linux 容器只是对进程的隔离，启动快且占用资源小。docker 就是对 linux 容器的封装，它将应用程序打包到一个文件里，运行时就会生成一个虚拟容器；docker compose 可以进行多容器应用的管理和部署

##### 包管理工具

- npm & yarn：安装包时将版本区间解析为具体的版本号，将对应的 tar 包下载到本地，然后解压、拷贝到 node_modules 目录。npm2 版本之前依赖树嵌套结构，会导致包重复安装；npm3 版本后以及 yarn 采用扁平化结构避免这个问题，但扁平化算法耗时较长

- pnpm：将全局 store 硬连接到 node_modules/.pnpm 下，项目依赖树是嵌套结构，不过使用软连接，大大提升了安装速度，节省了磁盘空间

##### git

- 常用命令

  - pull & fetch：git pull 相当于 git fetch + git merge

  - reset & revert：reset 用于本地变更回滚，重置 add 或 commit 后的变更；revert 用于撤销已经提交到远程仓库的 commit，会生成一个撤销的 commit

  - merge & rebase：merge 会产生一条合并的 commit，只需要解决一次冲突，适合向公共分支合并；rebase 需要多次修改冲突，依次使用 git add 、git rebase --continue 的方式来处理冲突，完成 rebase 的过程，git 树不会分叉，适合公共分支向其他分支合并

- 工作流：从 master 拉开发分支 dev，多人协作开发时创建自己的 feature 分支，完成开发后合并到 dev 分支。提测后基于 dev 分支拉 release 分支，当测试过程中发现 bug，将变更提交到 release 分支，完成测试后合并到 dev 分支。发布后将 dev 合并到 master，如果之后发现了线上问题需要修改，基于 master 拉 hotfix 分支，修复完成后合并到 master

##### node

##### 单元测试

##### tcp

- tcp 和 udp 区别

  - tcp 建立连接前需要三次握手，而 udp 不需要握手就可以建立连接
  - tcp 是有状态的、可控的，保证包按序到达，如果丢包会重发，网络环境不好的情况下会控制发送速度，更为可靠，而 udp 是无状态的、不可控的
  - tcp 以字节流的形式传输，udp 直接传输数据报文，udp 头部更小，传播效率更高，适合直播、通话等场景，tcp 适合文件传输等对可靠性要求比较高的场景
  - tcp 一对一传输，udp 可以一对一、一对多、多对多传输

- 三次握手：客户端请求建立连接并发送 SYN 和 Seq=x，服务端收到后确认连接并响应 SYN、ACK=x+1、Seq=y，客户端收到后发送 ACK=y+1、Seq=z，服务端收到后建立 tcp 连接

- 四次挥手：客户端请求断开连接并发送 FIN 和 Seq=x，服务端收到后表示确认并发送 ACK=x+1、Seq=y，等到服务端将剩余数据发送完毕后关闭客户端连接，并响应 FIN、Seq=z，客户端收到后确认关闭并发送 ACK=z+1、Seq=u，完成关闭。客户端在正式断开前会等待两个报文最大生存时间，确保服务端能够收到客户端确认关闭的请求

- SYN Flood 攻击

  - 概念：握手前服务端会创建两个队列：半连接队列（客户端发送 SYN 时入队）、全连接队列（握手完成且还没有被具体的应用取走），SYN Flood 攻击是指伪造大量不存在的 ip 地址并不断向服务器发送 SYN，导致服务端半连接队列占满从而无法处理正常请求，而且客户端没有响应导致服务端误以为发送失败而不断重发
  - 措施：增加半连接队列容量；减少服务端重发次数；服务端接收到 SYN 时不立刻分配资源，而是先生成一个 SYN Cookie 并伴随其他信息发送给客户端，客户端确认连接时带上这个 Cookie，服务端验证合法之后再分配连接资源

- 快速打开：首次三次握手时服务端根据客户端发送的 SYN 生成 SYN Cookie，然后响应给客户端，客户端缓存下来；再次三次握手时客户端会将这个 Cookie 连同 SYN 一起发送给服务端，服务端验证通过后响应给客户端后就可以发送 http 请求了，提前了一个往返时延

- 流量控制：tcp 将发送的数据存到发送缓存区，接收的数据存到接收缓存区，通过接收缓存区大小控制发送缓存区大小实现流量控制

- 拥塞控制：当网络状况不好时，需要通过控制拥塞窗口的大小来限制发送端发送，实际上是控制了发送缓存区的大小

##### http

- 请求方法

- http1.1

  - 持久连接：默认 Connection 为 keep-alive，即持久连接，而 http1.0 每发起一个 http 请求就会创建一个 tcp 连接，响应之后就关闭
  - pipeline：服务端按序接收一个 tcp 连接下的多个 http 请求，如果前一个请求耗时过长会阻塞后面的请求，造成队头阻塞问题，可以采取提高 tcp 并发连接数量、域名分片等措施，但没有从根本解决问题

- http2

  - 二进制分帧：将报文转成二进制帧不分前后地进行传输，接收端按 streamID 拼接，避免了队头阻塞问题
  - 头部压缩

- 常见状态码

  - 101：http 升级为 websocket 的时候，如果服务端同意升级就会返回 101
  - 200：请求成功
  - 206：分块下载和断电续传时会返回 206
  - 301：永久重定向，请求的资源被永久移动到新的 url
  - 302：临时重定向
  - 304：命中协商缓存，返回 304 时服务器不会返回其他资源，客户端使用缓存
  - 400：请求参数错误
  - 401：请求需要用户的身份认证
  - 404：无法根据请求找到资源
  - 405：请求方法错误
  - 500：服务器内部错误
  - 502：网关或代理服务器请求错误

- 常见请求头和响应头

  - http 缓存相关字段，如强缓存相关的 expires、cache-control：max-age=xxx，协商缓存相关的 if-Modified-Since 和 Last-Modfied、if-None-Match 和 etag
  - Accept 相关字段，包括数据格式、压缩方式、所支持的语言等
  - 跨域资源共享（CORS）相关字段，如 Origin 和 Access-Control-Allow-Origin、Access-Control-Allow-Credentials、Access-Control-Allow-Methods 等

##### https

- 概念：http 明文传输是不安全的，https 通过 TLS/SSL 进行加密，http 和 tcp 之前多一个中间层用来加密

- TLS 1.2 握手过程

  1. 客户端发送随机数 client_random、TLS 版本及加密套件
  2. 服务端收到后确认 TLS 协议版本并响应随机数 server_random、所需的加密套件、数字证书、公钥以及 server_params
  3. 客户端首先确认证书是否有效，验证通过后利用 server_params、client_params 通过 ECDHE 算法生成预主密钥，将其用公钥加密发送给服务端
  4. 服务端用私钥解密拿到预主密钥，两端各自用 client_random、server_random、预主密钥生成会话密钥，之后的数据报文都用会话密钥进行对称加密

- TLS1.3：相比于 1.2 主要有以下改进

  - 安全性：RSA 加密彻底被 ECDHE 所取代
  - 握手流程优化：client_params 会在第一次握手的时候传给服务端，此时服务端就能生成会话密钥，这样无需等待客户端验证证书，提前了一个往返时延 ß

##### websocket

- 为什么需要：短轮询和长轮询的开销都比较大，websocket 一次握手之后就可以持久连接

- 连接过程：客户端发起请求，告知服务端要升级为 websocket 协议及其协议版本，服务器确认协议版本并返回 101 状态码表示协议切换成功

##### 跨域

- 为什么存在：浏览器出于安全考虑有同源策略，即协议、主机、端口有一个不同就是跨域，当检测到跨域并且没有 cors 响应头，会将响应体丢掉

- cors：跨域资源共享，简单请求时请求头自动添加 Origin 字段表明来源，如果 Origin 没有被包含在响应头的 Access-Control-Allow-Origin 中，响应就会被拦截；非简单请求额外多了一个请求方法为 OPTIONS 的预检请求，用来询问服务端是否允许连接

- jsonp：script 标签没有跨域限制，通过 src 填上目标地址从而发出 GET 请求

- 反向代理：反向代理服务器如 nginx。反向代理是将客户端的请求转发给目标服务器，而正向代理是帮助浏览器访问目标服务器

##### 浏览器存储（cookie、localStorage、SessionStorage、IndexedDB）

- cookie 本职工作并不是存储，而是让 http 有状态，cookie 可以携带用户信息在浏览器和服务器之前来回传递，大小 4kb，超过会截断，同域名的多个页面可以共享；默认是 session，会话结束就会过期

- localStorage 是持久化缓存，除非手动删除否则不会过期，大小 5-10M，同域名的多个页面可以共享

- SessionStorage 是临时性缓存，会话结束就会清除，大小 5-10M，同域名的多个页面不会共享

- indexDB 是浏览器上自带的一个非关系型数据库，大小 250M 左右

##### 浏览器缓存（Memory Cache，Service Worker Cache，HTTP Cache 强缓存[ expires,max-age ]、协商缓存[ Last-Modified&If-Modified-Since,Etag&If-None-Match ]，Push Cache）

- 内存缓存是最先被使用、响应最快的一种缓存，会随着会话结束而销毁

- 如果不使用任何 http 缓存策略，设置 cache-control：no-store；如果不使用浏览器缓存，直接向服务器询问资源是否有效，设置 cache-control：no-cache

- 如果使用代理服务器缓存，设置 cache-control：public，否则默认为 cache-control：private，只使用浏览器缓存

- 再次请求时，可以用浏览器本地时间和上一次响应头的 expires 时间戳对比，如果没有过期就命中强缓存，即应用浏览器本地缓存，这就需要本地时间和服务器时间一致；或设置 cache-control：max-age=xxx 来设置一个时间段，浏览器根据请求到的那个时间点加上时间段来判断是否命中强缓存

- 再次请求时，请求头携带 if-Modified-Since，值为上一次响应头的 Last-Modfied 值，即上一次修改时间，服务器收到后判断资源是否需要重新返回，如果不需要返回一个 304 状态码，浏览器资源从本地获取，否则返回一个完整的响应；或请求头携带 if-None-Match，值为上一次响应头的 etag 值，即根据文件内容的编码，服务器收到后判断资源是否需要重新返回；etag/if-none-match 相比于 last-modified/if-modified-since 更容易感知到文件的变化，但生成 etag 需要额外的开销

##### 浏览器进程和线程（进程线程区别，Browser 进程，第三方插件进程，GPU 进程，渲染进程 gui 渲染线程、js 线程、事件触发线程、定时触发器线程、异步 http 请求线程，WebWorker 线程，Browser 进程和渲染进程的通信，GUI 渲染线程与 JS 引擎线程互斥）

浏览器是多进程的，包括浏览器主进程、渲染进程、GPU 进程、第三方插件进程等，其中渲染进程包括 GUI 渲染线程、js 引擎线程、处理事件循环的事件触发线程、用来计时的定时触发器线程、http 请求线程，由于 js 可以操作 dom，GUI 渲染线程和 js 引擎线程是互斥的，js 执行时间过长会导致 GUi 渲染线程长时间挂起导致页面卡顿

##### 浏览器渲染机制

渲染进程的主线程将 html 解析为 dom 树；解析 css 并结合 dom 树生成 render 树；然后遍历所有可见元素并计算几何位置生成 layout 树；实现一些 3d 变换、页面滚动、定位等效果需要做 z 轴排序，需要生成不同的图层，最终生成 layer 树；渲染引擎将每个图层拆分成小的绘制指令，按顺序组成待绘制列表，将其交给合成线程将图层划分为图块，一般借助 GPU 进程进行快速栅格化，即将图块转换为位图；栅格化完成后，合成线程发出一个绘制命令给浏览器主进程，然后生成页面

修改元素几何属性会触发重排，会重复从生成 layout 树开始之后的过程，损耗较大；修改元素的绘制属性会触发重绘，会跳过生成 layout 树和 layer 树的过程；使用 css transform 属性会跳过浏览器主进程阶段，开始合成阶段的执行，性能最好

##### 事件循环（宏任务、微任务、过程）

- 浏览器事件循环：浏览器只有一个宏队列和微队列。先执行全局同步代码，遇到 setTimeout 等回调添加到宏任务队列中，遇到 Promise 等回调添加到微任务队列中。然后开始执行微任务队列队首的任务，直到将所有微任务队列中的任务执行完成。如果执行的过程中出现了新的微任务会添加到微队列的队尾并在这个周期执行完成。然后开始执行宏人物队列队首的任务，每个周期只执行一个。重复这个过程，直到微队列和宏队列清空

- node 事件循环：node 有 4 个宏队列（timers、io callback、check、close callback）和 2 个微队列（nextTick、other）。先执行全局同步代码，遇到异步任务将其回调加入到相应的队列中等待处理。然后开始执行微队列中的任务，先执行 nextTick 队列再执行存放其他微任务的队列，直到全部执行完。然后开始执行 4 个宏队列，依次执行完各个宏队列中所有的任务，每执行完一个宏队列中的任务会重复上一步执行这个过程中可能产生的微任务，依次类推，直到所有宏队列和微队列清空

##### 图片优化（jpg、png、svg、base64、webp）

- jpg：有损压缩，体积小，适合背景图、轮播图等
- png：无损压缩，体积大，支持透明，相比于 jpg 色彩表现力更强，适合 logo 等
- svg：无限放大也不会失真，可编程，相比于 jpg 和 png 体积更小，但渲染成本高，适合一些需要动态变化的 icon
- base64：对图片进行 base64 编码可以减少请求次数，但编码后的体积是原来的 4/3，适合小图标
- webp：兼具 jpg 和 png 的优点，还能展示动图，但兼容性较差

##### 安全问题（xss 分类[ 存储型、反射型、dom 型 ] 策略[ 输入过滤转义、http-only、csp ]，csrf 分类[ get、post、链接 ] 策略[ 同源检测、csrf token ]）

- 跨站脚本攻击 xss 是指页面被注入了恶意脚本并在浏览器里执行了，分为存储型 xss（恶意代码存储在了数据库中读取时就会执行，如网站评论等）、反射型 xss（恶意代码存储在 url 中）、dom 型 xss（dom 操作导致恶意代码注入到页面中）。预防措施：输入过滤，一定程度上预防恶意代码被保存到数据库中；转义 html，防止恶意代码直接加载到页面中；设 cookie httponly 为 true，即使恶意代码注入到了页面中也可以防止用户信息被脚本获取

- 跨站伪造请求 csrf 是指诱导用户进入第三方网站，利用被攻击网站已经验证的身份信息向被攻击网站发起请求，从而冒充用户做出操作，比如表单、图片、链接等方式。预防措施：同源检测，利用响应头中的 origin 或 referer 字段来确定来源域名；csrf token，用户发出请求时携带一个攻击者获取不到的 token，一般有服务端生成，然后服务端根据 token 来决定是否响应

##### 其他浏览器特性

- webrtc
- webgl
- canvas
- svg
- webAssembly
