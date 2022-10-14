前端工程化，包括打包、编译等

#### todo

- cli

  - 如何搭建一个脚手架

- test

  - 单元测试
  - 安全测试

- node

- CI

- CD

#### 知识点

##### babel（作用、plugin、preset、core-js、runtime）

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

- 正向代理 & 反向代理：正常情况下客户端直接向目标服务器发出请求，正向代理隐藏了真实的客户端，借助代理服务器向目标服务器收发请求，真实的客户端对目标服务器不可见；反向代理隐藏了真实的服务端，借助代理服务器向客户端收发请求，真实的服务端对客户端不可见

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
