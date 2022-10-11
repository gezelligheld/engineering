前端工程化，包括打包、编译等

#### todo

- cli

  - 如何搭建一个脚手架

- test

  - 单元测试
  - 安全测试

- rollup

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

- 构建流程：根据配置和命令行参数初始化，如所需要的插件等，然后从入口 entry 出发，根据文件类型和对应的 loader 对文件进行编译，如果文件依赖了其他文件则递归的编译和解析。然后根据文件间的依赖关系合并成 chunk，最后将 chunk 转换为文件输出到文件系统中，包括业务代码、第三方代码以及 runtime 和 manifest 数据。浏览器加载打包后的资源时，文件结果已经不复存在，runtime 和 manifest 数据主要用来解析和加载各个模块

- 代码分离：chunk 体积过大过小都不合适，有几种可以拆分 chunk 的方式，包括配置入口文件、动态导入、配置 splitChunks 的 cacheGroups 属性

- 缓存：生成的 chunk 名可以基于文件内容生成 hash，当文件修改时浏览器就会请求新的文件从而使缓存无效，及时将变更更新到浏览器中；配置 runtimeChunk 属性可以将 runtime 代码放到一个单独的 chunk 里，这意味着修改文件重新打包时只有对应的 chunk 和 runtime chunk 会发生改变，其他 chunk 不受影响，可以充分利用缓存减少获取资源；第三方库的代码不会频繁修改，可以配置 entry 或 cacheGroup 使其与业务代码分离，减少向服务端获取资源

- 模块热替换：run server 后启动一个本地服务，浏览器访问资源时作出响应，同时建立 websocket 链接。webpack 会监听源文件的变化，当保存文件时触发 webpack 重新编译，生成一个 hash 和 js 文件，并通过 websocket 将 hash 值给客户端，客户端收到后与上一次作比较，如果不一致则请求 manifest 数据去得知那些 chunk 发生了变更，最后通过插入 script 或 style 标签来实现热更新

- tree shaking：基于 esm，在代码没有运行时就知道哪些模块没有引用，从而移除掉，减小代码体积

- loader：处理文件，使用对应的 loader 转换文件可以其用 import 导入；自定义 loader 是一个接收对应类型文件代码作为参数、返回转换后的代码的函数；常用的有处理样式的 css-loader、style-loader、postcss-loader 等，处理文件的 raw-loader、file-loader 、url-loader 等，用来编译的 babel-loader、ts-loader，用来校验的 eslint-loader

- plugin：用于扩展 webpack 的功能，可以在 webpack 构建过程中广播出来的事件里做更多的事情；自定义 plugin 是一个含有接收 complier 实例作为参数的 apply 方法的类，可以订阅构建过程中抛出来的事件；常用的有构建时定义全局变量的 DefinePlugin、指定某个第三方库不被打包的 IgnorePlugin、压缩 js 的 terser-webpack-plugin、抽离 css 用 link 引入的 mini-css-extract-plugin、可视化分析打包文件体积的 webpack-bundle-analyzer、自动生成 html 的 html-webpack-plugin

##### rollup

##### vite

##### 移动端（自适应布局、其他适配问题）

##### server（docker、IIS、nginx）

##### 其他（监控、埋点、包管理工具）

##### git

##### node

##### 单元测试
